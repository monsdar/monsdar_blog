---
title: "Contributing to Ansible"
date: 2025-06-20T12:00:00
draft: false
description: "Improving performance for complex deployments"
summary: "Improving performance for complex deployments"
---

A few years ago, when i took over DevOps duties in one of Rheinmetalls simulation projects, the first thing i did was to introduce automation and infrastructure as code using Ansible. What started as a simple "do this here, do that there" collection of playbooks has grown ever since and nowadays spans multiple collections with several hundred roles. 

When complexity started to grow we noticed that performance decreased. We introduced a number of measures, but after [fiddling with execution strategies](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_strategies.html), [facilitating async execution](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_async.html) and [tweaking the ssh parameters](https://www.redhat.com/en/blog/faster-ansible-playbook-execution) all other means had drawbacks: by splitting the complex deployment into several smaller ones we essentially circumvented Ansibles automatic dependency resolution. By adding skippable blocks and tags we put the burden of "what to deploy" onto the user instead of Ansible figuring out what needs to be done by itself. 

In the end it became unbearable: our most complex deployment is a software suite shipped with every new project. It contains a high level role that, after resolving dependencies, brings in 70 roles into the deployment. Overall a typical playbook run calls over 1000 tasks and to control and fine tune the deployment provides hundreds of variables (most of them stay default though).

## Finding and Fixing

I started investigating using [`ANSIBLE_VERBOSITY=6`](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#envvar-ANSIBLE_VERBOSITY) and [`ANSIBLE_DEBUG=true`](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#envvar-ANSIBLE_DEBUG). When in debug, Ansible prints out timestamps along with the log messages, so it was possible to see where the time is lost.

Here's some of the typical output that occurs during each task:

```
894 1748240216.80397: getting variables
894 1748240216.80399: in VariableManager get_vars()
894 1748240217.60269: Calling all_inventory to load vars for REDACTED
894 1748240217.60278: Calling groups_inventory to load vars for REDACTED
894 1748240217.60282: Calling all_plugins_inventory to load vars for REDACTED
894 1748240217.60292: Calling all_plugins_play to load vars for REDACTED
894 1748240217.60294: Calling groups_plugins_inventory to load vars for REDACTED
894 1748240217.60297: Calling groups_plugins_play to load vars for REDACTED
894 1748240217.60542: '/usr/local/lib/python3.10/dist-packages/ansible/plugins/connection/__init__' skipped due to reserved name
894 1748240217.80356: done with get_vars()
894 1748240217.80384: done getting variables
```

After seeing that `get_vars()` seems to take more than a second during our deployment I looked further into this and stumbled upon an [old comment](https://github.com/ansible/ansible/issues/73654#issuecomment-847002109) by [@Vladimir-csp](https://github.com/Vladimir-csp) that went mostly unnoticed (the issue was about memory consumption, but the comment hit the nail on the head regarding runtime performance).

I checked further and opened my [own issue](https://github.com/ansible/ansible/issues/85206) when there was none. Was I really the first to notice this? Are other people running much simpler deployments, or do they simply accept the performance issues?

When going through the code to see what's going on I noticed that `get_vars()` is called for each task - even if eventually it is dismissed due to tags not fitting. Adding a few more debug outputs showed the root-cause to be in the `Role.__init__()` code. In the end it was multiple issues:

* **Redundant dependencies in the tree:** I mentioned earlier that our dependency tree spans around 70 roles. Ansible however does not take into account that some dependencies are used by multiple roles. It simply adds every role to the dependency list, even if it's already in there. As our tree is pretty complex we ended up with a list of more than 3800 roles, the most fundamental roles were in there more than 100 times. To solve this I added a simple check that does not add the same role twice.
* **Reading default variables:** Whenever a task is executed the variables are read. This includes the default variables of the role that is being used (and its dependencies to make sure no default var is forgotten). As it turns out it takes some time (almost 0.5sec in my test environment) to go through the whole dependency tree and store the variables. But this is unnecessary: Default vars of a role do not change over the runtime of a playbook, so reading them once and caching them after was the way to go. With this change default vars are only read once in the beginning, then cached and later on taken from this cache when needed during each task.
* **Reading role variables:** The same turned out to be the case for normal role variables. These do not change as well, but take 0.5sec to read too. Again caching is the solution.

## Results

As of writing this [the Pull Request](https://github.com/ansible/ansible/pull/85249) is still open, as a fundamental change like this needs proper reviewing before releasing this out into the wild. So far I was able to resolve all remarks the maintainers had and even extended the tests a bit. When running the patched Ansible version in our deployments I was able to shave of tremendous amounts of time from the deployments:

* Full deployment...
    * with the latest 2.18 release of Ansible takes **90 minutes**
    * with my patched 2.19rc version takes only **10 minutes**
* A `--tags` deployment running a subset of all tasks...
    * with the latest 2.18 release of Ansible takes **75 minutes**
    * with my patched 2.19rc version takes only **2 minutes**

 This sounds too good to be true, but so far everything works as expected. Even more complex cases using `delegate_to` or overwriting and inheriting variables when using `include_role` worked like expected.

## Things to look out for

As this change was in the heart of the Ansible codebase any change I made had an impact onto other parts. The Ansible codebase is complex and there are a lot of edge-cases and workarounds to look out for. Luckily their test suite is pretty extensive and covers a lot of specialties. Two of their maintainers helped along the way and gave patient advice to any potential issue I ran into.

Here's a short list of things I encountered:

* **Variable precedence:** To ensure reusability Ansible has a guaranteed order of variable precedence. When you work with Ansible you know [their docs page](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable) about it, I have it open almost weekly. When introducing cached variables you need to make extra sure that vars from a later precedence level still overwrite ones from a previous (cached) one. 
* **Private and public variables:** Though not obvious Ansible has different encapsulation of variables. Think about loop-vars or variables that get introduced via `include_role` and `import_role`. The latter is even more complicated when using the `public: True` flag or when importing roles within an included role. Luckily there were integration tests covering these scenarios so I was able to solve upcoming problems quickly.
* **Run tests locally:** Speaking about tests: Within my development environment it wasn't easy to get them running, but once I had them available it was possible to iterate much quicker with my changes. It also ensured that my changes do not take more time from the reviewers away than I already had to take (thanks again for the patience!).

## Summary

Given the number of Ansible users I wonder a bit that something like this hasn't been adressed before - but that's often the case in Open Source. In German we have a saying: *"TEAM - Toll, ein Anderer machts"*, meaning *"TEAM - Nice, someone else is doing it"*. In this case it was me, but I'm more than happy to step forward. Given the results it was more than worth the time spent and I hope they'll help others as well.
