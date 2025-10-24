---
title: "Distributed Key-Value Store"
draft: false
weight: 1
tags: ["Go", "Databases", "Distributed Systems", "Networking", "Concurrency"]
summary: "Building a Redis-inspired key-value store so I can stop failing system design interviews."
---



A Redis-inspired distributed key-value store built from scratch in Go. 
Features TCP networking, write-ahead logging, snapshot compaction, 
leader-follower replication, and transactions. Built to deeply understand 
distributed systems concepts like consensus, durability, and replication lag.

[Github](https://github.com/akarki2005/kvstore)