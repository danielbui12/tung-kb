---
title: "Arweave 2.9: Efficient Generation and Dispersion of Entropy During Data Preparation"
type: source
tags: [arweave, packing, randomx, proof-of-work, entropy, mining, decentralized-storage]
created: 2026-06-19
author: Digital History Association (DHA) team
source_url: "local: raw/arweave2_9_2024.md"
source_type: paper
ingested: 2026-06-19
---
# Arweave 2.9: Efficient Generation and Dispersion of Entropy During Data Preparation

**One-line:** Arweave 2.9 replaces 2.8-era data "packing" with a new "activation" scheme built on RandomXSquared that recycles nearly all generated entropy, cutting honest-miner compute by ~99.6% and required disk read rates by ~90% while preserving the network's on-demand mining security margin.

## Summary
The paper (Draft-5, December 2024) proposes the data-preparation algorithm for the next Arweave protocol version, 2.9. Arweave incentivizes miners to store many distinct replicas of the dataset (rather than one shared unpacked copy) by requiring "packing" work via RandomX, a CPU-bound proof-of-work hash from the Monero ecosystem. The core security invariant is that storing data must be cheaper than re-packing it on demand: `ms(dsz, t) < mp(dsz)`. Keeping this invariant intact has historically imposed heavy upfront compute on honest miners (`h`) — exactly the kind of non-storage overhead the protocol's principles aim to minimize. The authors' goal is to raise the cost of the on-demand strategy (`op`) while holding honest-packing cost (`hp`) flat, widening the safety margin and lowering the effective price the network pays for storage.

The key inefficiency identified: Arweave 2.8 runs RandomX to produce a 2 MiB scratchpad but keeps only the first 8 KiB as masking entropy (`de`), discarding ~99.6% of the entropic output. A naive fix — reusing the full scratchpad across consecutive challenges — would let an on-demand miner amortize one RandomX run across 256 chunks, collapsing the packing invariant and making `o` the dominant strategy. The proposed "activation" algorithm instead spreads the full generated entropy across an entire partition's zones, so an on-demand miner must regenerate the entropy in many places (then discard or store it, like an honest miner does). For an `md=1` miner this makes packing ~256x more efficient for `h`; at `pd=32`-equivalent parameters, up to ~8,192x more efficient — all without shifting the `hp:op` ratio.

The entropy engine is **RandomXSquared (RX²)**, a construction layered on RandomX. It runs `al` parallel "lanes" of RandomX, each seeded with a unique nonce to produce its own 2 MiB scratchpad, and `am` times during execution it pauses all lanes and mixes their entropy together via a generator function `agm` applied across the concatenated scratchpad states. The result is one large, non-compressible `de` of size `al * 2 MiB`, split into 8 KiB subchunks distributed across a partition's zones. Because entropy from each lane is entangled with all others at multiple mix stages, no individual subchunk can be randomly accessed without computing all associated lanes — this is the "random-access resistance" property.

A central threat the design closes is the **ISTORE caching attack**. RandomX's design guidance ensures roughly as many memory-store (ISTORE) operations as scratchpad cells, leaving ~66% of cells overwritten. An attacker ("Eva") can instrument the VM to record which cells were written (a small ~1.5% bitmap) plus their final values, then later reconstruct the scratchpad cheaply by skipping program execution — effectively compressing the entropy by ~32.5%. This matters little to Monero or to Arweave 2.8 (which uses only the heavily-written first 8 KiB), but would let a 2.9 on-demand miner perform "packing compression." RX² defeats this by mixing entropy across every byte of every scratchpad multiple times: any ISTORE cache an attacker builds ends up at least as large as the full scratchpad, and the cost to cache scales linearly with work (Table 1: 2x work → ~126.6% storage, 3x → ~189.9%, versus RandomX plateauing near `|RXpad|`).

Proposed global parameters are `al=4`, `am=4`, `ad=3`, with `agm` = local CRC-32C (Castagnoli) followed by a deterministic 6-byte-block shuffle across concatenated scratchpads. CRC32 and shuffle are chosen deliberately for being hardware-accelerated and cheap: `agm` must cost little relative to a full RX² run, because work-truncation resilience depends on the final-mix shortcut being negligible (truncatable work ≈ `½ · c(ags)/c(RX2)`). Expensive mixers (SHA-2, BLAKE-3) are explicitly rejected. The cited results: ~99.6% honest-compute reduction versus 2.7/2.8 at `pd=1` (up to 99.987% versus `pd=32`); read rate dropping to ~1 MB/s (from 50 MB/s at `pd=1`, 200 MB/s in 2.7.x); on-demand safety margin up ~52.87%. The main drawback is increased proof-validation time (~200 ms on reference hardware), mitigated by validation now being parallelizable across up to 4 threads.

## Key takeaways
- Arweave 2.9 renames "packing" to "activation": same security role, but it reuses nearly all RandomX entropy instead of discarding ~99.6% of it.
- Honest-miner data-preparation compute drops ~99.6% (vs 2.7/2.8 `pd=1`) and up to ~99.987% vs `pd=32`; entropy is distributed across a partition's zones so on-demand miners can't cheaply amortize it.
- RandomXSquared = `al` parallel RandomX lanes mixed `am` times via a cheap `agm` (CRC-32C + 6-byte shuffle), producing a single non-compressible `al * 2 MiB` block that resists random access and work truncation.
- The design specifically neutralizes ISTORE caching ("packing compression") attacks that would otherwise let on-demand miners compress entropy by ~32.5%; under RX², an attacker's cache scales linearly with work and exceeds the scratchpad size.
- Required disk read rate falls to ~1 MB/s (from 50–200 MB/s), and the on-demand safety margin rises ~52.87%; cost is higher (but parallelizable, ~200 ms) proof validation.
- Proposed global params: `al=4`, `am=4`, `ad=3` (effective difficulty / read-rate setting `ed=50`); low-power hardware like a Raspberry Pi 5 could prepare a replica in acceptable time.

## Concepts & entities
Wikilinks: [[arweave]], [[packing]], [[succinct-proofs-of-random-access]], [[proof-of-access]], [[blockweave]], [[permaweb]], [[decentralized-storage]], [[replication]], [[randomx]], [[randomx-squared]], [[data-activation]], [[entropy-generation]], [[istore-caching-attack]], [[work-truncation-attack]], [[on-demand-mining]], [[proof-of-work]], [[verifiable-delay-function]], [[crc32c]], [[feistel-cipher]], [[digital-history-association]].

## Notable claims / data
- 2.8 keeps only the first 8 KiB of a 2 MiB RandomX scratchpad, discarding ~99.6% of generated entropy.
- Activation makes honest packing ~256x more efficient at `md=1`, up to ~8,192x at `pd=32`-equivalent parameters, without changing the `hp:op` ratio.
- Honest-miner compute reduction: ~99.6% vs Arweave 2.7/2.8 at `pd=1`; ~99.987% vs `pd=32` (one `mp` in 2.9 per 8,192 `mp` in 2.8 at `pd=32`).
- Read-rate requirement falls to ~1 MB/s (`ed=50`) versus 50 MB/s (2.8 `pd=1`) and 200 MB/s (2.7.x); example `al=4, ad=3` gives ~5 MiB/s (~20 chunks).
- On-demand mining safety margin (`c(hp):c(op)`) increases ~52.87% with `al=4, ed=50`.
- ISTORE caching against stock RandomX compresses a scratchpad ~32.5% (bitmap ~1.5% of entropy size; ~66% of cells overwritten). Birthday-paradox limits mean more work only reduces compressibility toward ~12.5%.
- Table 1 (cache storage cost): RandomX 1x/2x/3x work = 63.3%/86.6%/95.3% (plateauing at `|RXpad|`); RandomXSquared = 63.3%/126.6%/189.9% (linear, exceeds `|RXpad|`).
- RandomX defaults: ~L3-cache-sized scratchpad, BLAKE-2b seeding + AES1R generator, Argon2-derived ROM, 2048 program iterations; rejected alternatives include Argon2/Cuckoo Cycle for weak truncation resistance and GPU volatility.
- Proposed params: `al=4`, `am=4`, `ad=3`, `agm = CRC-32C → shuffle (6-byte block)`; work-truncation bound ≈ `½ · c(ags)/c(RX2)`.
- Proof validation ~200 ms on reference hardware, parallelizable across up to 4 threads at `al=4`.
- Algorithm 1 zone seed: `s = SHA256(maddr ∥ Pn ∥ z)`; `RX2progs = 2048`; max packing difficulty `ndm = 32` as of 2.8.

## Questions raised
- The paper deliberately forgoes cryptographically secure entropy ("random enough" that `o` can't predict outputs). What concrete cryptanalytic risks arise from CRC32 + static shuffle as the mixer, and how was "sufficiently random" validated beyond the cited simulation?
- How sensitive is the on-demand safety margin to real-world RandomX execution-speed improvements (ASIC/FPGA/optimized GPU) over time, given the whole scheme's security rests on CPU-bound predictability?
- Proof validation rises to ~200 ms; how does this scale with adversarially crafted blocks at high `al`/`am`, and what is the impact on full-node sync and verifier hardware requirements?
- The work-truncation bound depends on `c(ags) ≪ c(RX2)`. What is the empirical margin, and could a faster-than-modeled `agm` implementation reopen truncation attacks?
- How does the proposed global `ad=3` interact with (or deprecate) the 2.8 per-miner custom packing difficulty (`md` up to 32)? Is custom difficulty retained, removed, or subsumed?
- The parameters are global and immutable-ish; what governance/upgrade path adjusts `al`/`am`/`ad` if hardware or attack assumptions shift?

<!-- Note: the paper's reference list shows [1] AES, [2] BLAKE2, [3] Arweave VDF source, [4] Argon2, [5] Balloon hashing (Boneh et al), [6] VDFs, [7] Handbook of Applied Cryptography, [8] Bitcoin, [9] CRC cyclic codes, [10] 2.9 parameter tuning model, [11] birthday paradox multi-collisions, [12] Arweave principles, [13–16] RandomX (tevador), [17] Cuckoo Cycle, [18] Arweave paper (draft-17). -->
