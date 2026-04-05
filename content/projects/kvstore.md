---
title: "lsm storage engine"
draft: false
weight: 1
tags: ["Go", "Storage Systems", "Databases", "Systems"]
summary: "Building a write-optimized storage engine from first principles to understand how real databases handle durability and performance."
---

A write-optimized LSM-tree storage engine built from scratch in Go.

Implements core database internals including a write-ahead log, 
in-memory memtables, immutable SSTables, and compaction. Designed for high 
write throughput and crash-safe durability, with tombstone-based deletes 
and an efficient on-disk data layout.

Built to understand how real systems like embedded key-value stores manage 
persistence, recovery, and performance under write-heavy workloads.

[Github](https://github.com/akarki2005/lsm-engine)