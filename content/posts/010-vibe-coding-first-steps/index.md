---
title: "VibeCoding with Cursor AI"
date: 2025-05-03T12:00:00
draft: false
description: "Agentic AI is a Gamechanger"
summary: "Agentic AI is a Gamechanger"
---

For awhile now AI is pretty much everywhere. Besides getting personalized cooking receipts and putting your avatar in front of explosions it got added to VSCode as some sort of "improved Intellisense". With [GitHub Copilot](https://github.com/features/copilot) or Codeium it was possible to ask questions about your code directly in chat, have it write your unit-tests or create entire functions from a well-written comment.

That first generation of AI was ok and you could feel that it improved the results of your coding, but it wasn't more than that. Sometimes it felt like magic, but oftentimes you learned to ignore its suggestions because you already knew a better way to get the code you wanted.

Nowadays this changed. IDEs like [Cursor](https://www.cursor.com/) or [Windsurf](https://windsurf.com/) (from the Codeium team) have AI deeply integrated. It's not only possible to get a few completions here and there but to automatically implement full features that span the entire codebase.

I first stumbled upon Cursor when my brother shared [this blogpost from the Builder devs](https://www.builder.io/blog/cursor-tips) with me. At first glance it was too good to be true - but when I installed the Cursor IDE and started my 14-day trial I was impressed.

The codebase I had was a very early version of [NBAGrid](https://nbagrid.pythonanywhere.com/). It was mostly backend, I didn't even started with the Web UI yet. To get things going I asked for a main view that shows the grid to the user and got a first full version of the game running after just 2 hours of talking to the AI.

Since then I never went back and in my day job - where AI tools aren't allowed - coding feels like grind work. It's crazy how fast you get used to working with the agent as if it was a very knowledgable junior dev that you tell what to implement and then check their results. After just 2 weeks I was already pretty familiar with the new workflow and code output increased by a lot.

I found the following to work pretty well:

* **Keep up test coverage:** Even when very concise with what you want, the AI will sometimes change unrelated parts of the codebase. Reviewing every line it gives you thoroughly is needed. Also having a good test coverage and telling the AI to run the tests after it made changes helps a lot. It's actually pretty easy to get a good coverage, as AI can easily help you with that.
* **Keep a close-eye on the changes:** Don't get too much into the flow. Not following every change closely will easily get you in a situation where you do not understand what's going on under the hood anymore. AS tempting as it is, you'll spend that saved time later when you need to debug something that you think should've worked in the first place.
* **Use the agent for debugging:** Speaking of debugging: It's awesome how you can just tell the AI to add debug prints and then check for an issue you're foung within the code.
* **Keep your balance between high- and lowlevel:** You need to find the middleground between "changing larger parts of the codebase for a very big feature" and "micromanaging the agent because it always switches up your table-rows". The first results in most of your codebase not working anymore, the latter is very inefficient and you normally could've done the changes faster by yourself.

Where it really shines is with technologies you haven't worked with yet. I know myself around Python pretty good, but I haven't worked much with Django yet. I'm also very inexperienced when it comes to frontend web development. Connecting the pieces I know well by filling the knowledge-gaps with what the AI was showing me it was straight-forward to get something running. With these pointers I was able to read myself into the docs a bit more and get a better understanding of how everything works. I'm sure that the NBAGrid codebase now contains a large number of features and coding techniques I wouldn't have stumbled upon by just reading the docs and doing some tutorials before starting to code.

Lately I started trying out [different models](https://docs.cursor.com/settings/models). While the default Cursor model works pretty well, reasoning models like clause-3.7-sonnet provide good quality results and it's very interesting to see how it comes up with the solution it is asked for. Sometimes it's a bit frustrating though, especially when it stubbornly tries to make the changes it thinks it needs only for the result to be wrong. Instead of trying to change the approach it instead tries to apply the changes in different way. In one case it even tried editing the file by running `sed`-commands because it thought editing the file in-editor was the root cause of the issue it had.

Overall the step between the first VSCode extensions and the current state of agentic AI makes me wonder what the next major leap will look like. As a senior dev with a lot of experience the current version of AI is already very powerful. Hope in a few years from now it's not reversed roles, as the AI then is instructing the humans what to do next. 👀