---
title: HDFS
type: entity
tags: [distributed-storage, erasure-coding, hadoop]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# HDFS

**What it is:** The Hadoop Distributed File System.

## Notes
HDFS traditionally relied on 3x replication for durability. It added native [[reed-solomon]] [[erasure-coding]] (and [[local-reconstruction-codes]]) so cold data can be stored at ~1.4x overhead instead of 3x while tolerating more failures — trading cheaper storage for a k:1 repair read amplification that suits archival/cold workloads rather than hot low-latency ones. Common HDFS-EC configs are RS(6,3) and RS(10,4), though exact parameters vary by deployment rather than being fixed defaults.

Like [[ceph]], HDFS is a single-administrative-domain datacenter store, not a trustless [[decentralized-storage-networks|decentralized network]]; its erasure coding is the same finite-field math used by [[storj]] and [[walrus]], but confined to one operator's cluster.

## Sources
- [[erasure-coding-vs-raid]] — HDFS native Reed-Solomon + LRC to replace 3x replication on cold data
- [[erasure-coding-algorithms-comparison]] — RS(6,3)/RS(10,4) as common but non-universal HDFS-EC configs
