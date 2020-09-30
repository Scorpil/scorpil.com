+++
title = "Before You Start Coding"
description = "Developer's pre-flight checklist"
author = "Scorpil"
date = 2020-10-01T07:30:00+02:00
tags = ["programming"]
images = ["/img/tree.png"]
+++

Full disclosure: things discussed in this post are, to be frank, quite obvious. My intent is not so much to teach a reader something new, as to remind him what might have been forgotten. After all, anyone who does a single kind of work daily for years inevitably develops habits and mental-shortcuts that are _usually_ useful but can misfire from time to time. After all, software development is too complex and too technical to be guided by subconscious intuition.

Pilots use pre-flight checks to combat this kind of problem, so why not developers? Here’s a check-list of things I find important to consider **before** writing the first line of code.

### Set your target straight

Imagine yourself solving these tasks:

- generating a single-use report for an established company
- writing a feature in proof-of-concept MVP for a startup
- extending at enterprise product that has 20 years of codebase support guarantee in its SLAs

Clearly, it wouldn't be smart to blindly apply the same set of software development best-practices for all three cases. You probably don’t need a perfectly polished code for a single-use report. A startup that operates in the “rush to market before we run out of money" mode is much more worried about the feature working than about its performance. Long-support code needs to be first and foremost maintainable. Also, if you’re working on a pet-project for fun, you might gasp skip writing tests (unless you enjoy writing tests, in which case I envy you a little bit). Some developers take so much pride in their craft that they lose sight of the forest behind the trees. Following all best practices and writing state-of-the-art code _feels_ right, but logically it's not the only option. Sometimes you should consciously generate technical debt. Reality constraints are not something that can be ignored.

As self-evident as this advice is, we’ve all heard stories where things went wrong because of misaligned goals. “Premature optimization” is a common special case. Refactoring an old codebase that rarely changes, just so it’s pretty, is another one. And to not leave you with the thought that goal misalignment is always linked to perfectionism: writing an unsupportable code to meet a _fictional_ deadline, either self-imposed or forced on to developers by impatient management, is the same kind of error.

You need to know where to go before you can start thinking about how to get there. Think of the most important criteria your solution needs to fulfill and align your decision-making with those.

### Consider alternatives

You’ve got the task in front of you, requirements are pretty clear, and you vaguely remember how you did something similar a few years before. Or maybe you don’t know how to solve it at first, but after some googling, you get a general idea. Start coding?

There are always multiple ways to solve a problem. The first viable solution is unlikely to be an optimal one. By focusing on the first approach discovered you rob yourself of choice. Do a thorough research first, and after you have options in front of you, carefully consider the pros and cons of each. There is no magical number of solutions you need. I often set the minimum bound to three for myself.

Selecting the right tool for a job is an obvious example of a decision that benefits from upfront research, and even then, in a real-world scenario, a tool that’s familiar often gets selected over the tool that fits. Don’t think this advice only applies to „macro“ decisions, it can be applied just as well for your everyday code design choices.

### Consult the docs. Yes, upfront

It’s impossible to keep up with the pace modern tech is moving. Doesn’t matter how many years of experience you have under your belt, when you pick up the project on a modern stack there will be tools and libraries, released recently, that have slipped under your radar. So most developers are in a constant in-flight-learning mode: picking up knowledge as they search answers on immediate questions. It’s an efficient way to learn, but it creates a risk of missing an easy solution because of the inability to form the right question.

Skimming the docs upfront, without digging deep into any particular topic, doesn’t take much time, but it helps create mental anchors, which your brain will fetch when you encounter the problem. You will have a better starting point for finding the right answer quickly.

### Design your APIs

The hardest part of writing code is to imagine in the minutest details how to handle each workflow, each exceptional situation, and each edge case. Many developers first solve a problem, then write a „wrapper“ to expose their work to the outside world through some kind of API (i don’t mean just REST API, function signatures and interfaces are APIs as well). Often this bottom-up design leads to an API that’s too „technical“, i.e. leaking it’s abstractions to the consumer. It’s hard to make your brain switch from one level of abstraction to another.

Before starting the work on implementation, put yourself in the shoes of the API user first (even if that user is you). What would be the user’s most common usage pattern? What’s the absolute minimal required set of parameters the API needs to have? Which terminology makes sense for the user? What input format would be most convenient? What defaults to set? It’s up to you how thorough you want this process to be.

### Split up the work

Each entity in a system exponentially increases the number of possible interactions. Code that’s easy to reason about forms a pyramid of abstraction layers, each one hiding its inner workings from the layer above. Each layer can be reasoned about independently, limiting the amount of „moving parts“ you need to keep track of in your brain. If you succeed in splitting the task into multiple loosely coupled entities upfront, you will simplify your code and work process.

If the task at hand is large enough, it might make sense to form a tree-like task structure, starting from the highest level of abstraction and moving down the layers until the tasks are small enough. The „small enough“ parameter depends on your particular codebase, mindset, and type of work you do, but after a bit of practice, you’ll find what granularity suits you the best.

Each task should be fairly self-sufficient and ideally shouldn't stay in "kinda-finished" state for long ("finished but not deployed", "finished but not reviewed", "finished but not tested" or "finished but waiting for authorization from XYZ department"). Finished is when you don't think about it anymore and it doesn't drain your mental resources. 

![Example of a task tree](/img/tree.png)

Do you have your own tips and tricks for organizing developer's work? Please share!
