---
title: RAID
type: concept
tags: [raid, redundancy, parity, fault-tolerance]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [raid, erasure-coding-vs-raid]
---
# RAID

**Definition:** Redundant Array of Independent (Inexpensive) Disks — combining two or more physical drives into one logical volume to deliver redundancy, fault tolerance, and/or performance, with behavior set by the chosen RAID level.

## How it works
RAID composes three primitives: striping (splitting data across disks for parallel throughput), mirroring (replicating data to another disk), and parity (storing reconstruction information). Standard levels: RAID 0 (striping, no redundancy), RAID 1 (mirroring, half capacity), RAID 5 (block striping + distributed single parity, survives one disk loss), RAID 6 (dual distributed parity, survives two). Crucially, RAID parity *is* erasure coding confined to one box: RAID-5 is a `(k,1)` XOR code and RAID-6 a `(k,2)` Reed-Solomon code with fixed parity counts. Nested levels (10, 50, 60) stripe over redundant sub-arrays. Hot-swap and hot spares let a failed disk be rebuilt onto a spare on a live system.

## Why it matters / trade-offs
RAID provides high availability against disk failure inside a single array, but is **not a backup**: it does not protect against corruption, malware, controller failure, accidental deletion, or simultaneous multi-drive loss. The "write hole" (power loss between data and parity writes) is patched with battery-backed caches or journaling. Versus distributed [[erasure-coding]], RAID is intra-box with fixed `k+m` and rebuilds whole disks; distributed coding spreads tunable fragments across independent failure domains and survives whole-rack or whole-site outages.

## Related
[[erasure-coding]] [[reed-solomon]] [[redundancy]] [[replication]] [[data-durability]]

## Sources
- [[what-is-raid]] — levels, primitives, implementation, limits
- [[erasure-coding-vs-raid]] — RAID-5/6 as `(k,1)`/`(k,2)` codes, scope vs distributed coding
