---
title: Replication
type: concept
tags: [replication, redundancy, data-durability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [erasure-coding-vs-raid, object-storage-domain-knowledge]
---
# Replication

**Definition:** A redundancy strategy that stores whole copies of data (e.g. 3x replication) across independent locations, so any single surviving copy serves the data and tolerated failures equal copies minus one.

## How it works
Each replica is a complete, independently readable copy placed on a distinct failure domain. Reads need only one copy, so they are simple and fast with no decode or quorum. Repair is 1:1 — a lost replica is rebuilt by copying one surviving replica byte-for-byte — which keeps [[repair-bandwidth]] low per recovered unit (no k:1 read amplification). RAID-1 mirroring is the in-box form; distributed systems like Filecoin (sealed sectors) and Swarm (Kademlia neighborhood replication) use network-scale variants, sometimes coupling replication with proofs that copies are physically distinct.

## Why it matters / trade-offs
Replication's appeal is operational simplicity: trivial reads, fast repair, easy reasoning about placement. Its cost is space — 3x replication is 3x storage yet tolerates only two failures, whereas RS(10,4) tolerates four at just 1.4x overhead. So replication wins for hot, latency-sensitive, or small data where simplicity dominates, while [[erasure-coding]] wins for cold/archival/large data where the storage bill dominates. Many systems blend both: replicate hot tiers, erasure-code cold tiers.

## Related
[[erasure-coding]] [[redundancy]] [[data-durability]] [[repair-bandwidth]] [[raid]]

## Sources
- [[erasure-coding-vs-raid]] — 1:1 repair vs k:1, storage cost comparison
- [[object-storage-domain-knowledge]] — replication families across DSNs
