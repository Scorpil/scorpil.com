+++
title = "What the CAP Theorem Teaches Us About Engineering Organizations"
description = "How the CAP theorem explains trade-offs in software engineering organizations between quality, velocity, and communication."
tags = [
  "distributed systems",
  "software engineering",
  "engineering leadership",
  "system design",
  "technical debt"
]
date = 2025-06-20T09:00:00Z
author = "Scorpil"
draft = false
images = [ "/img/cap.png" ]
+++

Distributed systems are complex by nature. The more moving parts the system has — the more things can break. So over the decades that distributed systems became the norm, engineers have invented plenty of methodologies, principles, and mental shortcuts, to help us reason about distributed systems with less cognitive load.

One of the best-known examples of such a principle is the CAP theorem. The CAP theorem is relatively simple to understand — each distributed system is defined by the following three properties:

- **Consistency**: returning correct, up-to-date data
- **Availability**: returning data in a reasonable amount of time
- **Partition tolerance**: the ability to withstand network partitioning — simply put, the ability to communicate between the members of the system.

In a nutshell, the CAP theorem states that in any given system, only two of the three stated properties can be achieved at the same time. This is easy to prove empirically: imagine a database cluster that has been partitioned into two parts due to network connectivity failure. When a client requests data from a node in one network partition, that node has two options: it can either return the value in its local storage (potentially outdated, since the node cannot know if data has been updated in the other network partition), sacrificing consistency; or it can wait until the connection is restored, sacrificing availability. In practical terms, systems also attempt to get the best of both worlds by reconsolidating cached requests after network connectivity is restored, achieving eventual consistency — a middle ground of not fully consistent but always available system.

Though often discussed in the context of databases, the CAP theorem’s core trade-offs apply to any distributed system — including organizations themselves. We can directly map the CAP theorem's properties onto software engineering organizations:

- **Consistency → Quality**: working on compatible problems, without stepping on each other's toes. Producing high-quality, low-maintenance code. If your teams have trouble sorting out dependencies between each other or drown in tech debt - you have a Quality problem.
- **Availability → Velocity**: quick code turnaround, time-to-production
- **Partition tolerance → Communication**. Whether synchronous or asynchronous, information must flow between the teams and Individual Contributors to facilitate Software Development.

The conclusion of the CAP theorem in the context of the Software Engineering process can be rephrased as: to achieve both high Velocity and high Quality you must ensure efficient cross-team and intra-team communication. When communication is lacking you will either end up losing velocity, by re-writing code to adapt it to newly incoming requirements, or by simply wasting time waiting for input from stakeholders; alternatively, you may choose to lose Quality by cutting corners to meet the deadlines.

When we compare these two kinds of distributed systems — digital and human — one key difference stands out: network throughput. For most modern distributed systems, network throughput is effectively unlimited (with few exceptions for special cases that focus on high-throughput use cases, e.g. Apache Kafka). This is not the case for software engineering organizations.

Human communication is much less efficient than computer communication, and it tends to get less efficient the less structure it has. The least efficient form of professional communication in a software engineering organization is a meeting without an agenda. Probably the most efficient form of communication is a PR review (when done right, of course). Scrum ceremonies, architecture review meetings, emails, and Slack threads are all somewhere in between. Each form of communication has its time and place, but generally, the more focused and structured communication is, the less "communication throughput" it uses. Think about typical Open Source projects, where 90% of communication happens over PR reviews and Issue discussions, and how much leaner it is compared to a similar corporate project, where 90% of communication happens in a meeting room. To be clear, I do not advocate shifting all discussions to a single channel (you don't want to align on system requirements or on the high-level solution architecture in a PR review), but communication media should be consciously adjusted in a way to fit the available throughput sparingly.

Connectivity issues between components manifest themselves differently in the digital world of distributed systems and the social world of software development teams. Network partitioning means a complete lack of communication — no information goes through. In software engineering organizations, the problem is typically an excess of irrelevant, unstructured information taking up available throughput (and time) with a low "signal-to-noise" ratio. Results, however, are similar — nodes of the system do not receive the necessary information, so one or both of the remaining properties inevitably take a hit.

It's worth noting that just as smart engineers employ clever tricks to reach favorable Availability/Consistency tradeoffs (like Eventual Consistency), smart engineering leaders employ strategies to balance out Quality/Velocity in an imperfect communication environment (though I hesitate to call this "eventual quality"). Sometimes, in the face of uncertainty, it's better to take the risk of developing a solution based on a set of unproven assumptions, rather than wait for those assumptions to be validated. This might generate some tech debt, yet allows the team to maintain velocity in a limited communication setting, which is sometimes more important (e.g. when timing the market, developing MVP, etc.).

Thinking of an engineering organization as a distributed system can be a powerful tool for any scientifically minded engineering leader. The mathematical principles remain valid - even when the system nodes are as complex and unpredictable as humans.
