---
title: What Is RAID (Redundant Array Of Inexpensive Disks)?
type: source
tags: [raid, redundancy, fault-tolerance, parity, data-storage, mirroring, striping]
created: 2026-06-19
author: Unknown
source_url: "local: raw/What Is RAID (Redundant Array Of Inexpensive Disks)?.md"
source_type: article
ingested: 2026-06-19
---
# What Is RAID (Redundant Array Of Inexpensive Disks)?

**One-line:** RAID combines multiple physical disks into one logical unit to deliver redundancy, fault tolerance, and/or performance, with behavior determined by the chosen RAID level.

## Summary
RAID (Redundant Array of Independent/Inexpensive Disks) configures two or more drives to present as a single logical volume, trading raw capacity for high availability, performance, or both. The core goal is data redundancy: keeping a system operational through partial hardware failure so that a data center experiences no downtime when a disk dies. Two adjacent concepts frame the technology — redundancy (duplicate components serving the same function) and fault tolerance (a backup component takes over on failure to prevent service loss).

RAID can be implemented in software (built into most modern OSes, cheaper, no vendor lock-in, but CPU-bound and lost on power failure) or hardware (a dedicated RAID controller with battery-backed write cache, higher throughput, faster rebuilds, suited to mission-critical workloads). Data is distributed across drives via three primitives: mirroring (replicating data to another disk), striping (splitting data across disks for parallelism), and parity (storing reconstruction information so lost data can be rebuilt after a drive failure). Hot swapping (replacing a failed drive on a live system) and hot spares (idle drives that auto-activate on failure) reduce downtime, but availability ultimately depends on the RAID level deployed.

The article enumerates the standard single levels: RAID 0 (striping, no redundancy, capacity = sum of drives), RAID 1 (mirroring, usable capacity = half), RAID 2 (bit-level striping with Hamming-code error correction), RAID 3 (byte-level striping with a dedicated parity disk), RAID 4 (block-level striping with dedicated parity), RAID 5 (block striping with distributed parity, survives one drive loss), and RAID 6 (double distributed parity, survives two drive losses). Nested/hybrid levels combine two schemes: RAID 10 (1+0) and 0+1 (4-drive minimum), RAID 30 and RAID 50 (6-drive minimum), and RAID 60 (8-drive minimum). JBOD (spanning) presents disks as one volume with no RAID and no fault tolerance.

Deployment requires matched drives (capacity, write speed, transfer rate — arrays default to the smallest drive's capacity), a RAID controller, hot-swappable drive bays, and OS driver support. RAID is not a backup substitute: servers can still fail from hardware conflicts, malware/OS-upgrade corruption, incompatible drives, controller failure, or simultaneous multi-drive failures. On failure, administrators should avoid hasty action; parity-based striped arrays rebuild more easily than per-drive recovery, and RAID recovery software exists for harder cases.

## Key takeaways
- RAID's purpose is high availability via redundancy, not backup — it does not protect against corruption, malware, or controller failure.
- Three storage primitives underpin all levels: mirroring, striping, and parity.
- Level choice dictates the capacity/performance/fault-tolerance trade-off; RAID 0 has zero redundancy, RAID 1 halves capacity, RAID 5 survives one disk loss, RAID 6 survives two.
- Hardware RAID adds battery-backed write cache and faster rebuilds; software RAID is cheaper but CPU-dependent and fails on power loss.
- Nested levels (10, 30, 50, 60) stripe over redundant sub-arrays for combined performance and protection at the cost of more drives and overhead.
- Arrays default to the smallest member drive's capacity, so mismatched drives waste space.

## Concepts & entities
Obsidian wikilinks to wiki pages this source touches: [[raid]], [[redundancy]], [[data-durability]], [[parity]], [[mirroring]], [[striping]], [[fault-tolerance]], [[erasure-coding]], [[reed-solomon]], [[hot-swapping]], [[jbod]], [[object-storage]].

## Notable claims / data
- RAID 0 capacity = sum of all drives; RAID 1 usable capacity = half the sum (two 160GB drives → 160GB usable).
- An array uses the capacity of its smallest drive; a 250GB drive paired with an 80GB drive wastes 170GB outside JBOD.
- Minimum drive counts: RAID 0/1/2 = 2, RAID 3/4/5 = 3, RAID 6 = 4, RAID 10/0+1 = 4, RAID 30/50 = 6, RAID 60 = 8.
- RAID 5 survives one drive failure (distributed parity); RAID 6 survives two (double distributed parity).
- RAID 2 uses Hamming codes to verify and correct single-disk errors on each read.

## Questions raised
- How does parity-based RAID (5/6) relate formally to [[erasure-coding]] and [[reed-solomon]] codes used in [[object-storage]] and [[decentralized-storage]]?
- What are realistic rebuild-time and rebuild-failure risks for large modern drives, given the article only notes recovery scales with disk size?
- Why is RAID 2 essentially obsolete in practice despite its error-correction capability?
- How do RAID's local-disk redundancy guarantees compare with replication and erasure coding in distributed/object storage systems?
