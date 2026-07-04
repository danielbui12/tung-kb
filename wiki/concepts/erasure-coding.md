---
title: Erasure Coding
type: concept
tags: [erasure-coding, redundancy, data-durability, reed-solomon]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, object-storage-domain-knowledge, storj-v3, raid, erasure-coding-algorithms-comparison]
---
# Erasure Coding

**Definition:** A redundancy scheme that splits an object into `k` data fragments and computes `m` parity fragments, storing all `n = k + m` fragments across independent failure domains so that any `k` of the `n` reconstruct the original — tolerating up to `m` losses.

## How it works
Encoding is one matrix multiply of the `k` data fragments against an `n x k` generator matrix (Vandermonde or the cheaper Cauchy variant) over a Galois field, almost always `GF(2^8)`, where every `k x k` submatrix is invertible. Placing the identity in the top `k` rows yields a *systematic* code, so the original data fragments appear verbatim and happy-path reads need no decode. Recovery stacks any `k` surviving rows, inverts that submatrix, and multiplies. "Erasure" means the missing fragments are *located* (dead disk, offline node), which makes the math far cheaper than correcting unknown errors. Reads require a quorum of `k` fragments.

## Why it matters / trade-offs
Erasure coding buys equal or better durability than replication at a fraction of the space: RS(10,4) tolerates four failures at 1.4x overhead versus 3x replication tolerating only two. The cost is repair: rebuilding one lost fragment classically reads `k` fragments (k:1 read amplification), straining [[repair-bandwidth]]. It also adds decode CPU on degraded reads, making it suit cold/archival/[[object-storage]] over hot low-latency workloads. [[reed-solomon]] sits on the [[mds-code|MDS Singleton bound]] for optimal efficiency; [[local-reconstruction-codes|LRC]] and [[regenerating-codes]] trade strict optimality for cheaper repair I/O along two different axes (read-count vs bytes-per-helper). A separate family, [[fountain-codes]] and [[ldpc]], drops the fixed-`n` assumption entirely for transmission-oriented (not storage-oriented) use.

## Related
[[reed-solomon]] [[mds-code]] [[local-reconstruction-codes]] [[regenerating-codes]] [[fountain-codes]] [[redundancy]] [[replication]] [[data-durability]] [[repair-bandwidth]] [[raid]] [[object-storage]]

## Sources
- [[erasure-coding-vs-raid]] — core mechanics, GF arithmetic, MDS bound, repair problem
- [[object-storage-domain-knowledge]] — DSN usage and overhead figures
- [[storj-v3]] — production RS parameters
- [[what-is-raid]] — RAID parity as a special case of erasure coding
- [[erasure-coding-algorithms-comparison]] — the full code-family comparison (MDS, LRC, regenerating, fountain, LDPC) and the storage-overhead/repair-cost/decode-complexity triangle
