+++
title = "Things Elixir's Phoenix Framework Does Right"
tags = [
    "elixir",
    "phoenix",
    "webdev",
]
date = 2020-09-25T07:30:34Z
author = "Scorpil"
+++

I dabbled in Phoenix for a while now, but never _really_ got my hands dirty with it right up until now. Apart from the whole framework being surprisingly well thought through, there are a few things that strike me as being done _exceptionally_ well in Phoenix, compared to the rest of modern web frameworks.

![Phoenix Framework Logo](/img/phoenix.png)

### 1. Striking a balance between flexibility and strictness 

Modern web frameworks can be roughly divided into two camps:
- Flexible "DIY" frameworks are a little more than a set of utilities for the most common web-related tasks. Most Go frameworks are like this, as well as ExpressJS. They enforce little to no rules for the structure of your applications and rely on the community to come up with the extensions and best practices. As a result, they are very flexible; those with a large community have extensions to perform any task imaginable. On the flip side, apps built on such a foundation can, given poor governance, slowly evolve into an unsupportable mess of incompatible plugins and mismatched coding styles.
- Strict "batteries included" frameworks bring with them a complete set of tools to perform common web development tasks, as well as a set of conventions to go with it. They guide the developer into optimal code structure and typically strive to provide a single favored way of doing things. Of course, these kinds of frameworks are also extendable, but built-in tools often get embedded so deep into the project that they are almost irreplaceable. In this category, the most popular examples are Django and Ruby on Rails.

Of course, most frameworks are not on an extreme end of the scale, but the distinction is there. Worth noting that neither group is strictly better than the other -- each has its usecases.

Phoenix Framework, in my mind, holds very close to the middle of this scale for these reasons:
- it builds upon Elixir's functional philosophy, so it has a very clear idea how things _should_ work (single request context passed around as the first argument to all components that participate in a response generation, avoiding side effects where possible, MVC-inspired architecture, etc.)
- it does not hide its internal details in a "black box", quite the opposite - it encourages you to understand internal conventions to write your own code in the same fashion. When you get comfortable using a framework, you can probably read its code without too much trouble.
- by default Phoenix comes with a huge pack of tools and utils (ORM, routing, test suit, HTML rendering, metrics dashboard (sic!)...), but in most cases, there's a trivial way to swap them out or turn them off.

### 2. Reactiveness

When an app requires bi-directional communication between client and server, you usually either
integrate a 3rd party library into the framework, which means writing a pile of glue code, or
use a specialized framework like Tornado, which (caution, personal opinion here) kind of an awkward choice for those parts of the web app that do not concern themselves with WebSockets.

Phoenix is great for classical HTTP, but persistent communication is where it _really_ shines. Primitives it gives you with channels, PubSub and Presence are just enough to avoid boilerplate without sacrificing flexibility. Recent live view release is a whole new way of building dynamic apps. I wouldn’t go as far as to call it revolutionary, but it is definitely an intriguing attempt of bridging the gap between frontend and backend.

### 3. Performance

Phoenix’s performance has surprisingly little to do with the framework itself. It inherited its impressive concurrence characteristics from Elixir, which got it from Erlang, which got it thanks to the primitives of the BEAM virtual machine and the architectural patterns of OTP. The main principle at work to achieve concurrency is to schedule lightweight threads of execution to run each independent piece of work concurrently. You might have seen this approach in other languages (goroutines, python's greenlets, etc.), that’s because it works great to organize concurrent code execution without performance hit and with a minimal headache for a developer. However, BEAM gives this concept support on a VM level, which means it can be optimized even on a hardware level.

While lightweight processes help you perform well on your IO-bound tasks, Elixir being a compiled language means that CPU bound tasks won't bottleneck easily as well, and perform on less computing resources than most alternatives. While I can't be 100% sure that it will be faster for your application than Go or Rust in terms of CPU usage, I'm reasonably sure that it will be more than fast enough in a context of a typical web app.

_**Update:** correction based on discussion in here and on other platforms: Elixir is slower than Go/Rust on purely CPU-bound tasks, mainly because BEAM interrupts running threads for task scheduling. Also, Elixir/Erlang compiles to bytecode, not directly to the machine code (although BeamAsm, JIT compiler for Erlang's VM, has [landed in master 4 days ago](https://github.com/erlang/otp/pull/2745), so this should change in the next OTP release)._

### 4. Failure tolerance and cluster-awareness

You might have heard Phoenix being called a "monolithic framework". This is true to some extent: Phoenix *does* encourage you to put your frontend, backend, and background tasks in the same app. However, it also provides facilities to ensure that failure in a single component of the app will not affect other independent components. To explain in short, the app is divided into processes that communicate with each other via kind-of event messages. Each component is supervised, the supervisor will catch unhandled failures and restart the process in an attempt to fix them. It's somewhat reminiscent of a microservice architecture, just on a lower level.

Unlike most frameworks, Phoenix understands that it will most likely run on more than one node. It provides a way to communicate over the network in exactly the same way you communicate between the processes within your app.

This does not mean that Phoenix is a bad choice for microservices, it just means that framework itself can handle some of the same concerns microservices are designed to handle. A smaller app might benefit from that by avoiding some of the complexities of building up your own microservices architecture. Bigger apps, and those that are already built as microservices, can still incorporate Phoenix effectively.

### Are there any drawbacks?

Elixir, and functional programming in general, is still a long way away from the mainstream. If you don't know your way around closures, immutable data structures, and functional thinking it will take you a while before getting to feel comfortable with Phoenix. The good news is that functional programming is on an upwards trend, and with applications growing more parallel and distributed, I don't think this trend will reverse any time soon. So the knowledge you acquire in the process will serve you well going forward, independent from what the future holds for Phoenix.

The included tooling is great, but you might miss some 3rd party SDK's when you need them. That's definitely something to consider when starting a project. To give you an example: [AWS](https://aws.amazon.com/getting-started/tools-sdks/) at the time of writing does not provide an official elixir client library.

Phoenix Framework a rock-solid, production-ready tool with a variety of usecases. It feels fresh and well thought through. I won’t be surprised if Phoenixes popularity continues to grow to reach the level of „Top tier“ frameworks in the next few years.