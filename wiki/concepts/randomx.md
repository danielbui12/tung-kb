---
title: RandomX
type: concept
tags: [randomx, proof-of-work, asic-resistance, arweave, monero]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-2-9, arweave-yellow-paper]
---
# RandomX

**Definition:** A CPU-bound, ASIC-resistant proof-of-work hash originally from the [[monero]] ecosystem. It is [[arweave]]'s entropy primitive for data packing/activation; version 2.9 wraps it in "RandomXSquared" to recycle generated entropy.

## How it works
RandomX runs a randomly-generated program inside a virtual machine against a large (L3-cache-sized) scratchpad, using an Argon2-derived ROM, BLAKE2b seeding, and an AES-based generator over ~2048 program iterations. Tying the work to general-purpose CPU features (large memory, branchy integer/float code) erases the advantage of fixed-function ASICs. Arweave uses a RandomX run to produce a 2 MiB scratchpad of entropy that masks ("packs") a data chunk, so storing a packed replica is cheaper than re-deriving it on demand. Arweave 2.9's **RandomXSquared (RX²)** runs `al` parallel RandomX lanes and mixes their scratchpads `am` times with a cheap mixer (CRC-32C plus a 6-byte shuffle), yielding one large non-compressible entropy block that resists random access and work-truncation.

## Why it matters / trade-offs
ASIC resistance keeps mining accessible to commodity hardware, supporting decentralization. For Arweave, RandomX's tunable cost lets the protocol hold the invariant that on-demand re-packing costs more than honest storage. RX² additionally recycles ~99.6% of entropy that 2.8 discarded, cutting honest compute and disk read rates dramatically — at the cost of higher (but parallelizable) proof validation and an ongoing dependence on RandomX staying genuinely CPU-bound.

## Related
[[proof-of-access]] [[arweave]] [[monero]] [[content-addressing]]

## Sources
- [[arweave-2-9]] — RandomX scratchpad packing, RandomXSquared lanes/mixing, ISTORE-cache and work-truncation resistance
- [[arweave-yellow-paper]] — RandomX as Arweave's mining/packing primitive
