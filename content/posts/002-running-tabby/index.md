---
title: "Tabby: Run your own local AI code completion"
date: 2024-09-24T20:00:00
draft: false
summary: "Code completion is awesome, with Tabby you can run it locally and have everything under control."
---

With the advent of AI language models everybody can have their own assistant nowadays. Most of the time it feels like magic, some of the time it's hilarious. After OpenAI showed ChatGPT to the world a number of bigger and smaller companies are providing their services to the public.

This is also true for coding assistants - a special form of LLM that integrates directly into your IDE and suggest (mostly) helpful annotations to your code. Here's a short, non-complete list of services you could use:

* [GitHub Copilot](https://github.com/features/copilot) is probably the most famous service out there. It got trained using all of GitHub and therefore has seen a lot of codebase already. It integrates flawlessly with VSCode and provides many useful features.
* [GitLab Duo](https://about.gitlab.com/gitlab-duo/) is the answer from GitLab. It's a bit younger, but you can see that they're quickly catching up to Copilot. Duo is integrated into the whole of GitLab, so you can easily use it for summarizing a Merge Request or helping with sorting issues. Besides the currently available online version they're also planning an on-premise Duo that you can use.
* [Tabnine](https://www.tabnine.com/) and [Codeium](https://codeium.com/) are two of many upcoming alternatives to Copilot and Duo. They provide integration with additional IDEs, come with additional languages supported and most importantly, are running fully on-premise. That way it's possible to have a real private coding assistant that is not using your codebase as an input for others as well.

All of these services run on proprietary LLMs, besides Codeium, which has a free individual tier all of them cost money. This is where [Tabby](https://tabby.tabbyml.com/) is coming in. It provides...

* ...a complete Open Source, self-hosted server
* ...a number of IDE integrations
* ...access to a number of LLMs you can choose from according to your available hardware

The server is capable of handling multi-user environments with additional context being added per user. I'm currently running it on my Desktop computer with an GeForce RTX 3060. With the default models it runs very smooth, though the quality of the answers is not comparable to what the commercial services provide. This changes with one of the bigger models though. Starting tabby with the following CLI works pretty good for me: `tabby.exe serve --model DeepseekCoder-1.3B --chat-model Mistral-7B`. It takes a while for the initial download, but quality of answers is pretty good while latency is still tolerable. I can imagine using a more sophisticated GPU will yield even better results.

Tabby supports a [number of different models](https://tabby.tabbyml.com/docs/models/) and the team also [provides a benchmark](https://leaderboard.tabbyml.com/) so you can see compare what quality you can expect.

For the time being I'll continue using Tabby as a local assistant running on the same host I'm writing this on. I will give updates on how it fares in comparison to the other asisstants I know (and those I currently haven't tried, like GitLab Duo).
