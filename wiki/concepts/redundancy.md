---
title: Redundancy
type: concept
tags: [redundancy, data-durability, replication, erasure-coding]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [raid, erasure-coding-vs-raid, storj-v2, storj-v3]
---
# Redundancy

**Definition:** Controlled extra storage — parity or whole replicas — added to data so that a subset of surviving pieces can reconstruct the original after component failures.

## How it works
Redundancy spans an axis between two strategies. **Replication** keeps `r` whole copies (e.g. 3x): simple, with 1:1 repair and single-copy reads, but storage cost scales linearly with copies. **Erasure coding** stores `n = k + m` fragments where any `k` reconstruct the original: far cheaper space (RS(10,4) is 1.4x overhead and tolerates four losses, versus 3x replication tolerating two) but reads need a `k`-fragment quorum and repair reads `k` fragments per lost one. RAID parity is the in-box form of the same math. Distributed systems place pieces across independent failure domains (disk, host, rack, region) so correlated failures don't take out a quorum at once.

## Why it matters / trade-offs
Redundancy is the foundation of [[data-durability]]: more parity/copies means more tolerated failures, at the price of storage overhead and repair traffic. The choice between replication and erasure coding is the master design variable for storage systems — it cascades into overhead, read pattern, churn handling, and what proof systems must verify. Redundancy alone does not catch silent corruption (needs checksums/scrubbing) and is not backup (no protection from deletion, bugs, ransomware).

## Related
[[replication]] [[erasure-coding]] [[raid]] [[data-durability]] [[repair-bandwidth]] [[reed-solomon]]

## Sources
- [[what-is-raid]] — redundancy vs fault tolerance, mirroring/parity primitives
- [[erasure-coding-vs-raid]] — replication-vs-coding cost comparison
- [[storj-v2]] — early redundancy design
- [[storj-v3]] — production erasure-coded redundancy
