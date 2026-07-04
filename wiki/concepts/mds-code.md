---
title: MDS Code (Maximum Distance Separable)
type: concept
tags: [mds-code, singleton-bound, erasure-coding, reed-solomon]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# MDS Code (Maximum Distance Separable)

**Definition:** An erasure code that hits the Singleton bound: for `n` total fragments encoding `k` data symbols, **any** `k` of the `n` fragments suffice to reconstruct the original — the theoretical ceiling for storage efficiency at a given fault tolerance `m = n - k`.

## How it works
The Singleton bound states that a code's minimum distance `d` can be at most `n - k + 1`; a code achieving equality is MDS, and its minimum distance directly gives its fault tolerance (`m = d - 1` erasures survivable). Reed-Solomon achieves this because its generator matrix is built so that every `k x k` submatrix is invertible over the Galois field — any `k` rows, in any combination, recover the original `k` symbols. There is no redundancy left unused: `m` parity fragments buy exactly `m` tolerated losses, no more, no less.

## Why it matters / trade-offs
MDS-ness is the benchmark every other code is measured against. [[reed-solomon]] is MDS and therefore storage-optimal, which is why it remains the default for [[raid]], HDFS, [[storj]], and [[walrus]]. Codes that sacrifice MDS optimality do so deliberately to buy something else: [[local-reconstruction-codes]] give up strict MDS for cheaper single-failure repair reads; [[fountain-codes]] give up the hard MDS *guarantee* (RaptorQ is only near-MDS in practice) to drop the fixed-`n` assumption entirely; [[ldpc]] gives up MDS for near-capacity throughput at huge block sizes. In short, MDS optimality is the axis every non-MDS code trades away for cheaper repair, unknown-`n` tolerance, or decode speed.

## Related
[[reed-solomon]] [[erasure-coding]] [[local-reconstruction-codes]] [[regenerating-codes]] [[fountain-codes]] [[ldpc]]

## Sources
- [[erasure-coding-vs-raid]] — Singleton bound, RS as canonical MDS code
- [[erasure-coding-algorithms-comparison]] — MDS as the shared benchmark axis across the whole code family
