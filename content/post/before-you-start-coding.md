+++
title = "Before You Start Coding"
description = "Developer's pre-flight checklist"
author = "Scorpil"
date = 2020-09-30T01:14:32+02:00
tags = ["programming"]
draft = true
images = ["/img/tree.png"]
+++

Full disclosure: things discussed in this post are, to be frank, quite obvious. It’s purpose is not so much to tech a reader something new, as to remind him what might have been forgotten. After all, anyone who performs a single kind of work daily for years, inevitably develops habits and mental-shortcuts that are _usually_, but because they're subconscious they can misfire from time to time.

Pilots use pre-flight checks to combat this kind of problem, whey not developers? Here's the check-list of things I find important to consider before writing the first line of code.

### Set your target straight

Imagine yourself solving these tasks:

- generating a single-use report for an established company
- writing a feature in proof-of-concept MVP for a sturtup
- extending at enterprise product that has 20 years of codebase support guarantee in its SLAs

Clearly, you can't apply the same software development practices for all three cases. You probably don't need a perfect code for a single-use report. A startup that operates in the "rush to market before we run out of money mode" is much more worried about the featrue working, than about the its performance. Long-support code needs to be first of all maintainable. And if you're working on a pet-project for fun, you might *gasp* skip writing tests (unless you enjoy writing tests, in which case I envy you a little bit). Some developers take so much pride in their craft that they loose the sight of forest behind the trees. We try to follow all best practices and write state-of-the-art code because it feels right, even when logically its not the only option. Reality constraints is not something that can be ignored.

As self-evident as this advice is, we’ve all heard heard or read stories where things went wrong because of misaligned goals. “Premature optimization” is a common special case. Refactoring an old codebase that rarely changes, just so it’s pretty, is another one. And to not leave you with the thought that goal misalignment is always linked to perfectionism: writing an unsupportable code to meet a fictional deadline is the same kind of error (although it's often made by managers and not developers).

TL;DR: you need to know *where* to do before you can start thinking *how* to get there. Think of most important criteria your solution needs to fulfill, and align your decision-making with them.

### Consider alternatives

You’ve got the task in front of you, requirements are pretty clear, and you vaguely remember how you did something similar few years before. Or maybe you don’t know how to solve it at first, but after some googling you get a general idea. Start coding? Hmm...

There are always multiple ways to solve a problem. The first *viable* solution is unlikely to be an *optimal* one. By focusing on the first approach discovered, you rob yourself of choice. Do a thorough research first, and after you have a few options in front of you, carefully consider the pros and cons of each. There is no magical number of solutions you need I often set the minimal bound to three for myself.

Selecting a right tool for a job is an obvious example of a decision that benefits from upfront research, and even then, in a real-world scenario a tool that’s familiar often gets selected over the tool that fits. Don’t think this advice only applies to a „macro“ decisions, it can be applied just as well for your everyday code design choices.

TL;DR: evaluate a few options before committing to a solution 

### 3. Consult the docs. Yes, upfront

It’s impossible to keep up with a pace the modern tech is moving. Doesn’t matter how many years of experience you have under your belt, when you pick up the project on a modern stack there will be tools and libraries, released recently, that have slipped under your radar. So most developers are in constant learn-in-flight mode, picking up knowledge as they search answers on immediate questions. It’s an efficient way to learn, but it creates a risk of missing an easy solution because of inability to form a right *question*.

Skimming the docs upfront, without digging deep into any particular topic, doesn’t take much time, helps to create a mental anchors, which your brain will fetch when you encounter the problem. You will have a better starting point for finding the right answer quickly.

### 4. Define your APIs

The hardest part of writing an app is to imagine in minutest details how to handle each workflow, each exceptional situation and each edge case. Often developers first solve a problem, then write a „wrapper“ function to expose their work to the outside world through some kind of API (i don’t mean just REST API, function signatures and object interfaces are APIs as well). Sometimes this bottom-up design leads to an API that’s too „technical“, i.e. leaking it’s abstractions to the consumer.

### 5. Split it up

Each entity in a system exponentially increases the number of possible interactions. Code that’s easy to reason about forms a pyramid of abstraction layers, each one hiding its the inner workings from the layer above. Each layer can be reasoned about independently, limiting the amount of „moving parts“ you need to keep track of at once. If you succeed in splitting the task into multiple loosely coupled entities upfront, you will gain not just simpler code, but also an orderly work process. 

If the task at hand is large enough, it might make sense to form a tree-like structure starting from the highest level of abstraction, moving down the layers until the tasks are small enough. The „small enough“ parameter depends on your particular codebase, mindset and type of work you do, but after a little practice you’ll find what granularity suits you the best.

![Example task tree](/img/tree.png)