---
title: "My Understanding of the Raft Consensus Algorithm"
draft: false
weight: 2
date: 2025-10-26
tags: ["Consensus Algorithms", "Reliability", "Fault Tolerance", "CAP Theorem", "Distributed Systems"]
summary: "Some learnings from reading distributed systems theory."
---

### Introduction

Recently, I decided to dive into learning distributed systems with the implementation of a key-value store. No longer were buzzwords like reliability, scalability and maintainability gonna scare me! Or so I thought.

The first week of working on the project was fairly smooth sailing. I learned the syntax of Go, using the concurrency and networking primitives it provides to build an architecture supporting concurrent client connections with mutual exclusion. As I progressed further, I added persistence through a write-ahead log and synchronous disk writes, and later optimized log restoration through snapshotting and periodic compaction. 

At that point, I had a working store implementation with a single-node server supporting multiple connected clients. But then came a question that often comes up in the system design process: how do you make the system *reliable* and *fault-tolerant*? In other words, how do you make it so node failure is not catastrophic, or even noticeable to clients? 

Redundancy!

In designing distributed systems, distribution strategies can take on one of two key forms: *replication* or *partitioning/sharding*. The latter is usually implemented to facilitate system *scalability*; it involves splitting your data horizontally, across multiple servers. Great for storing large amounts of data; not so great for adding system redundancy. What we want for that goal is the former approach, *replication*, where we maintain copies of data across nodes so that a crash does not lead to total system failure.

One notable design pattern for implementing replication is the Raft Consensus Algorithm, described in [this paper](https://raft.github.io/raft.pdf). It solves a fundamental problem with replication: how can you keep multiple copies of the same data consistent with one another when networks are unreliable and servers crash randomly?

### How Raft Works

under construction 🚧 (will be finished once [this](https://en.wikipedia.org/wiki/Line_5_Eglinton) is finished)

### Other Approaches

Although it is great at what it does, Raft isn't the only approach to replication. It is one of many approaches which prioritizes *consistency*, where all nodes must agree on the exact same data, even if that means becoming unavailable for a short time while discrepancies are rectified. 

But what if your use case values *availability*, and is willing to accept potentially stale data in exchange for no downtime? That's where you'd want a system like the one Amazon uses for DynamoDB, described in [this paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf).

The tradeoff between consistency and availability is a recurring problem in the design of distributed systems, formalized in something called the CAP Theorem. You can read more about the CAP Theorem [here](https://www.ibm.com/think/topics/cap-theorem).