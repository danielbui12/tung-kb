---
title: HDFS
type: entity
tags: [distributed-storage, erasure-coding, hadoop]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [erasure-coding-vs-raid]
---
# HDFS

**What it is:** The Hadoop Distributed File System.

## Notes
HDFS traditionally relied on 3x replication for durability. It added native [[reed-solomon]] [[erasure-coding]] (and Local Reconstruction Codes, LRC) so cold data can be stored at ~1.4x overhead instead of 3x while tolerating more failures — trading cheaper storage for a k:1 repair read amplification that suits archival/cold workloads rather than hot low-latency ones.

Like [[ceph]], HDFS is a single-administrative-domain datacenter store, not a trustless [[decentralized-storage-networks|decentralized network]]; its erasure coding is the same finite-field math used by [[storj]] and [[walrus]], but confined to one operator's cluster.

## Sources
- [[erasure-coding-vs-raid]] — HDFS native Reed-Solomon + LRC to replace 3x replication on cold data
