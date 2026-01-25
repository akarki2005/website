---
title: "cloud nine"
draft: false
weight: 98
date: 2026-01-25
tags: ["AWS", "Azure", "Distributed Systems", "Data Centers", "Databases", "Cloud Computing"]
summary: "Why cloud systems are designed the way they are."
---

### introduction

"I know AWS" is a phrase that I've used far too confidently, far too many times. It seemed as if saying so was a golden ticket of sorts. I could list it as a skill on my resume or speak (i.e. handwave) about the cloud at an interview, and I'd immediately sound competent. 

The problem is that there's a huge difference between performative expertise and actual expertise - I was only good at the former.

It's not that I didn't know enough cloud terms and/or definitions, it was that I didn't yet understand the *why* behind cloud computing. I do now, so here it is with the CS jargon stripped away as much as possible (because gatekeeping sucks).

### why choose the cloud?

Let's say you're a startup that needs to decide how to scale your operations as your first users come in. For you, buying physical servers is a very steep capital expense, and you'll probably misjudge how much capacity you need anyways. Either it won't be enough, and you're screwed because you can't respond quickly enough when user demand spikes, or it'll be too much, and you're screwed because those shiny servers you spent a fortune on are sitting idle while traffic is dead and your balance sheet is red. All these screws, and no screwdriver :(

Now let's flip to the other extreme. Say you’re a multi-billion-dollar company operating at global scale. At this point, buying hardware is fine. You can afford it. You probably already own several data centers storing a worrying number of Excel spreadsheets.
But the core issue doesn't really go away. Demand shifts by region, the time of day, and whatever ad campaign marketing decided to launch that week. Capacity that’s perfect for one market is useless in another, and over-provisioning everywhere “just in case” is costly.

In both cases, the inability to respond to *elasticity* in demand means you’ve committed to infrastructure based on vibes. Obviously, not a good idea. So what is?

### what cloud services do

Cloud services like AWS and Azure serve as *abstractions* of physical data centers. Instead of owning hardware and planning around fixed capacity, you access (via API) shared pools of compute, storage, and networking that you can allocate on demand. 

Cloud platforms are built around *elasticity* and *scalability*. You can add capacity quickly when demand increases, scale individual components independently, and give that capacity back when the spike passes (and save $$$ in the process!). These decisions are fast and, crucially, reversible. You’re no longer locked into guesses you made ages ago.

Importantly, cloud services also assume that machines are unreliable, because treating any single one as “important” is generally a mistake. Instead of building systems which depend on specific servers staying alive, you spread responsibility across multiple. When a server goes down, you can just reroute traffic to another while a replacement is created in the background.

What this means, in essence, is that most cloud-hosted application architectures do not depend on individual machines or servers. Rather, collective system availability is king.

### why machines don't matter

Individual machines "not mattering" is only possible if they satisfy a very key property: *statelessness*. In other words, a machine is only non-critical if it doesn't know anything that nobody else does and that you wouldn't want to lose.

In practice, this means that most cloud-based *application servers* are *stateless*. They can handle user requests, maybe do a computation or two or talk to a database, but after that, they just forget everything. If one goes down mid request, it's not the end of the world. You can initialize a new instance and retry the request.

With this in mind, a lot of what the cloud does begins to make more sense. Autoscaling of application containers in response to heavy traffic loads works fairly easily, because new server instances don't need to remember anything on startup. Failures are survivable and cheap because no single container holds information that can't be found elsewhere.

But what about my data? Where does my state go if the application doesn't hold any of it?

### the state of the cloud

Your state, of course, gets pushed somewhere else. To databases, caches and queues. These stateful structures *do* matter; they hold critical data that you do not want to lose. 

If all important state gets pushed onto a small number of systems, those systems need stronger guarantees. Typically, these systems are *replicated* across multiple machines to ensure data *redundancy*. That way, a single failure doesn’t take all your data with it.

Unfortunately, this safety comes at a cost. Replication makes systems slower (it increases *latency*), coordination between replicas introduces complexity, and failures force you to consider tradeoffs between *consistency* and *availability*. But that’s the point: you accept that cost in a few carefully designed systems so the rest of the architecture is stateless, replaceable, and easy to scale. Because that's what the cloud is all about.