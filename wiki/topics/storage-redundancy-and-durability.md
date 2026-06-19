---
title: Storage Redundancy and Durability
type: topic
tags: [erasure-coding, raid, redundancy, data-durability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [what-is-raid, erasure-coding-vs-raid, object-storage-domain-knowledge, storj-v3, storj-v2]
---
# Storage Redundancy and Durability

**Thesis / current understanding:** [[data-durability]] — the probability stored data survives failures over time — is bought with [[redundancy]]. There are two families: keep whole copies ([[replication]]) or encode data into fragments where a subset reconstructs the whole ([[erasure-coding]]). The same math runs from one box ([[raid]]) up to planet-scale decentralized networks; what changes is the *failure domain* (disk → host → rack → independent node) and the *cost axis* you optimize (storage overhead vs [[repair-bandwidth]] vs CPU).

## Landscape

- **[[replication]]** — 3× copies. Simple, 1:1 repair, but high storage cost. The baseline.
- **[[erasure-coding]]** — split into *k* data + *m* parity fragments (*n* = *k*+*m*); any *k* of *n* reconstruct, tolerating *m* losses. Overhead ~*n/k* (e.g. 1.4× for 10+4) vs 3× replication. Costs: needs a read quorum, CPU to encode/decode, and *k*:1 read amplification on repair.
- **[[reed-solomon]]** — the canonical MDS erasure code (sits on the Singleton bound: no wasted space). Galois-field polynomial math; workhorse of CDs, [[raid|RAID-6]], [[storj]], [[walrus]], [[hdfs]].
- **[[raid]]** — erasure coding confined to one box with a fixed recipe: RAID-5 = (*k*,1) XOR parity, RAID-6 = (*k*,2) Reed-Solomon. Provides *local* redundancy only; explicitly not a backup.
- **Repair-optimized codes** — LRC (local reconstruction codes, e.g. Azure ~(12,2,2)) add local parities to cheapen single-failure repair; regenerating codes (MSR/MBR, e.g. Clay codes in [[ceph]]) minimize [[repair-bandwidth]]. They trade strict MDS optimality for lower repair I/O.

## The core trade-off

```
              storage overhead   repair cost      read path
replication        high (3x)      cheap (1:1)      any 1 copy
Reed-Solomon       low (~1.4x)    expensive (k:1)  need k fragments
LRC                medium         cheap local      need k fragments
```

[[object-storage]] is the flagship erasure-coding use case (cheap durability for write-once-read-rarely data). Decentralized networks add churn and untrusted nodes, so they pair coding with active repair and [[proof-of-storage|proofs]] — see [[decentralized-storage-networks]].

## Open tensions / contradictions
- **Durability ≠ backup.** Erasure coding and RAID protect against *component failure* but not human error, ransomware, or (alone) silent corruption. [[scrubbing]]/checksumming is needed to convert silent corruption into known erasures.
- **Coding vs unique replication in DSNs.** [[storj]] treats erasure coding + repair as sufficient durability; [[filecoin]] insists replicas be *provably unique* ([[proof-of-replication]]) — a different threat model. See [[decentralized-storage-networks]].

## Open questions
- At what failure-domain correlation does erasure coding's durability advantage over replication break down?

## Sources
- [[what-is-raid]] — RAID levels, parity, mirroring, striping; local redundancy
- [[erasure-coding-vs-raid]] — EC math (MDS, Singleton, Galois fields), LRC/regenerating codes, RAID as a special case
- [[object-storage-domain-knowledge]] — durability numbers, EC in decentralized contexts
- [[storj-v3]], [[storj-v2]] — applied Reed-Solomon (k,m,o,n) with repair
