---
title: "write now, pay later"
draft: false
weight: 97
date: 2026-03-10
tags: ["B-Trees", "LSM-Trees", "Database Internals", "Storage Engines"]
summary: "A deep dive into database internals."
---

### introduction

For a long time the default answer to “how do databases store data?” was B-trees.
That answer still holds for systems like PostgreSQL, MySQL, and SQLite. But if you look at many modern storage engines such as RocksDB, Cassandra, or LevelDB, you will see something different.

They use LSM trees.

This is not because B-trees stopped working. Rather, the workload changed.
Modern systems ingest enormous amounts of data. Logs, metrics, events, telemetry, analytics streams. Data arrives continuously and at large scale. For these systems, writes, not reads, are often the bottleneck.

A storage engine ultimately implements a simple mapping:
`
key → value
`.
The hard part is doing this on disk.
Disks are unfortunately pretty picky about how they are used: 


- `sequential reads   😊`

- `random reads       🙂` 

- `sequential writes  😊`

- `random writes      🤬🖕`   


Even on SSDs, the difference between sequential and random writes is large enough to matter.

Because of this, storage engine design ends up being less about data structures, and more about I/O patterns.

### the fat tree

The easiest way to understand B-trees is to start with something familiar.

A binary search tree splits the key space in two at every node. One key sits in the node, and the two children represent everything smaller and everything larger.
A B-tree is basically that idea, but pushed much further.

Instead of storing one key and two children, a node stores many keys and many children. Each key partitions a range of values, and the children represent those ranges.

In other words, the branching factor is much larger than two.

This means B-trees are very fat.

But the reason for this has nothing to do with their [eating habits](https://onetreeplanted.org/blogs/stories/how-much-co2-does-tree-absorb?srsltid=AfmBOorpdySp6JAXHsRnEipbdFjpcwwC3aiUpGEIlCyutrpjfKac0JuR) or sedentary lifestyle. It comes from how disks work.

Databases read and write data in page-sized chunks, usually somewhere around 4–16KB. If you are going to pay the cost of reading a page from disk, you want to pack as much useful information into that page as possible.

So B-tree nodes are designed to fit neatly inside a page and hold dozens or even hundreds of keys.

The result is a tree that stays extremely shallow. Even large datasets usually require only a few page reads to locate a key.

From a read perspective, this works great.

From a write perspective, this sucks.

B-trees modify pages in place. Updating a key means finding the page that contains it and rewriting that page on disk. Sometimes the page even has to split into two to make room - which means random disk writes. 

For traditional transactional workloads that tradeoff is fine. For systems ingesting massive streams of data, it starts to look expensive.

### write now

LSM trees approach the problem from the other direction.

Instead of updating data on disk immediately, writes are buffered in memory first.
Incoming writes are inserted into an in-memory structure called the memtable. The memtable keeps keys sorted so that the data can later be written to disk efficiently. Implementations usually use structures like skip lists or balanced search trees to maintain that ordering.

Eventually the memtable fills up. When it does, its contents are flushed to disk as a sorted immutable file called an SSTable.

From that point on the file is never modified.

New updates simply create newer versions of keys in later SSTables.
Over time, background processes compact these files together, merging them and discarding older versions of keys.

The important shift is that disk writes become sequential appends instead of random updates.

Disks are much happier with handling that. 

### pay later

Of course, nothing is free.

Because updates create new versions of keys rather than modifying existing ones, data ends up scattered across multiple SSTables. Reads may need to check several files before finding the newest value.

Systems reduce this cost using bloom filters, caching, and indexing structures.
Compaction also introduces write amplification, since data is periodically rewritten when SSTables are merged.

There is also space amplification, because multiple versions of a key may temporarily exist until compaction removes the older ones.

In other words, LSM trees shift the cost around. Writes become cheap and sequential, while reads and background maintenance become more complicated.

Write now, pay later.

### decisions, decisions

The choice between B-trees and LSM trees is not really about which data structure is better.

It is about the workload.

B-trees assume a world where reads dominate and updates are relatively small. They keep the tree shallow so that lookups touch as few disk pages as possible.

LSM trees assume a world where data arrives continuously and systems are willing to reorganize it later.

Modern distributed systems often look much more like data ingestion pipelines than traditional databases. Logs, metrics, events, and telemetry arrive constantly and get reorganized in the background.

Under that workload, optimizing the write path tends to be desirable. But under more traditional workloads, the fat tree still sings.