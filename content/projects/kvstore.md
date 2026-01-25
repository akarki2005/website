---
title: "distributed key-value store"
draft: false
weight: 1
tags: ["Go", "Databases", "Distributed Systems", "Networking", "Concurrency"]
summary: "Building a key-value store from the ground up so I can stop failing system design interviews."
---

A distributed key-value store built from scratch in Go. 
Features TCP networking, write-ahead logging, snapshot compaction, 
leader-follower replication, and transactions. Built to deeply understand 
distributed systems concepts like consensus, durability, and replication.

[Github](https://github.com/akarki2005/kvstore)