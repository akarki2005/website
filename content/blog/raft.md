---
title: "boats and consensus algorithms"
draft: false
weight: 99
date: 2025-10-26
tags: ["Consensus Algorithms", "Reliability", "Fault Tolerance", "CAP Theorem", "Distributed Systems"]
summary: "Some learnings from reading distributed systems theory."
---

### introduction

If you spend enough time around distributed systems, someone will eventually tell you to read [*Designing Data-Intensive Applications*](https://unidel.edu.ng/focelibrary/books/Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems%20by%20Martin%20Kleppmann%20(z-lib.org).pdf). It’s usually recommended as a book about databases and large-scale systems. After a few chapters you realize it’s really about something else: all the ways computers fail once you connect a bunch of them together.

If a system runs on a single machine, failure is simple. The machine goes down and the system stops working. In a distributed system, the expectation is very different. Nodes can *and will* fail, networks can *and will* partition, and yet the system is expected to keep serving requests.

So how do you make sure failure is not catastrophic, or even noticeable to clients?

Redundancy!

In designing distributed systems, distribution strategies can take on one of two key forms: replication or partitioning/sharding. The latter is usually implemented to facilitate system scalability; it involves splitting your data horizontally, across multiple servers. Great for storing large amounts of data; not so great for adding system redundancy. What we want for that goal is the former approach, replication, where we maintain copies of data across nodes so that a crash does not lead to total system failure.

One notable design pattern for implementing replication is the Raft Consensus Algorithm, described in [this paper](https://raft.github.io/raft.pdf). It solves a fundamental problem with replication: how can you keep multiple copies of the same data consistent with one another when networks are unreliable and servers crash randomly?

### how raft works

Raft solves the replication problem by designating one node as the leader.

Instead of letting every server accept writes independently, all client write requests are routed through that leader. When a client submits an operation, the leader appends the change to its log and sends that entry to the other nodes in the cluster, known as followers.

Once a majority of the nodes acknowledge the entry, the operation is considered committed. At that point, each node applies the change to its local state machine.
This majority rule is what makes the system reliable. As long as most of the nodes are alive and able to communicate, the cluster can continue operating while maintaining a consistent history of operations.

Of course, leaders do not last forever. Machines crash, networks fail, and sometimes nodes simply become unreachable.

When that happens, the remaining servers begin a leader election. Each node votes for the candidate with the most up-to-date log. Eventually one node receives a majority of votes and becomes the new leader, and the cluster continues processing requests.

From the outside, the system behaves like a single logical server. Internally, it is just a group of machines agreeing on the same ordered log.

### the replicated log

The key idea behind Raft is that the log is the source of truth.

Instead of replicating raw state directly, Raft replicates a sequence of operations. Each node applies those operations in the exact same order, which guarantees that their state eventually converges.

If a follower falls behind, the leader simply sends it the missing log entries until it catches up. If logs ever diverge, the protocol ensures that inconsistent entries are overwritten so that the cluster eventually returns to a single agreed-upon history.

In practice, this means that every replica executes the same sequence of commands, producing the same result.

### other approaches

Although it is great at what it does, Raft isn't the only approach to replication. It is one of many approaches which prioritizes *consistency*, where all nodes must agree on the exact same data, even if that means becoming unavailable for a short time while discrepancies are rectified. 

But what if your use case values *availability*, and is willing to accept potentially stale data in exchange for no downtime? That's where you'd want a system like the one Amazon uses for DynamoDB, described in [this paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

The tradeoff between consistency and availability is a recurring problem in the design of distributed systems, formalized in something called the CAP Theorem. You can read more about the CAP Theorem [here](https://www.ibm.com/think/topics/cap-theorem).