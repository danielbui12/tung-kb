---
title: Reed-Solomon
type: concept
tags: [reed-solomon, erasure-coding, mds-code, galois-field]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [erasure-coding-vs-raid, storj-v3, object-storage-domain-knowledge]
---
# Reed-Solomon

**Definition:** The canonical Maximum Distance Separable (MDS) erasure code, treating data as symbols over a finite field and deriving parity such that any `k` of `n` symbols reconstruct the original — sitting exactly on the Singleton bound for optimal storage efficiency.

## How it works
Reed-Solomon operates over a Galois field, typically `GF(2^8)` (caps at 255 fragments) or `GF(2^16)` for wider stripes, where addition is XOR and multiply/divide use lookup tables or SIMD. Encoding multiplies the `k` data symbols by a generator matrix built from a Vandermonde or Cauchy structure; equivalently it can be viewed as polynomial evaluation at distinct points. Because every `k x k` submatrix is invertible, decoding inverts the surviving rows. Classical decoding of unknown errors uses Berlekamp-Welch (or Berlekamp-Massey + Forney), though storage systems exploit the cheaper *erasure* case where missing positions are known.

## Why it matters / trade-offs
RS is the workhorse of redundancy across scales: RAID-6 dual parity, CDs/DVDs, deep-space telemetry (Voyager), HDFS-EC, Storj (RS 29-of-80), and Walrus (over `GF(2^16)`). Being MDS, it wastes no space for a given fault tolerance. Its weakness is repair cost — rebuilding one fragment reads `k` others (k:1 amplification) — which motivates LRC and regenerating codes that sacrifice strict MDS for lower [[repair-bandwidth]]. Well-tuned SIMD implementations (Jerasure, Intel ISA-L, Backblaze) encode at several GB/s per core.

## Related
[[erasure-coding]] [[raid]] [[repair-bandwidth]] [[redundancy]] [[data-durability]] [[storj]] [[walrus]] [[hdfs]]

## Sources
- [[erasure-coding-vs-raid]] — GF arithmetic, generator matrices, MDS/Singleton, SIMD throughput
- [[storj-v3]] — production RS(29,80) deployment
- [[object-storage-domain-knowledge]] — RS in Storj, Walrus, HDFS
