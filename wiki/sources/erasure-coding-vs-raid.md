---
title: Understanding Erasure Coding and Its Difference With RAID
type: source
tags: [erasure-coding, reed-solomon, raid, distributed-storage, data-durability, repair-cost, object-storage]
created: 2026-06-19
author: danielbui12
source_url: "local: raw/Understanding Erasure Coding And Its Difference With RAID.md"
source_type: article
ingested: 2026-06-19
---
# Understanding Erasure Coding and Its Difference With RAID

**One-line:** Erasure coding trades a controlled amount of redundancy for the ability to lose fragments and still reconstruct data exactly, and RAID is just that same math confined to one box with a fixed parity recipe.

## Summary
Erasure coding splits an object into `k` data fragments, computes `m` parity fragments derived from them, and stores all `n = k + m` fragments (a stripe) across independent failure domains. Any `k` of the `n` fragments reconstruct the original, so the system survives up to `m` losses. This buys the same or better durability as triple replication at a fraction of the cost: a 10-of-14 scheme tolerates four failures at 1.4x overhead, versus 3x replication tolerating only two. The term *erasure* signals the key assumption — the system knows *which* fragments are missing (dead disk, offline node), which makes the math far cheaper than full error correction over unknown errors.

The machinery relies on finite-field (Galois field) arithmetic, almost always `GF(2^8)`, so every operation on bytes stays a byte with no overflow or rounding. Encoding is one matrix multiply against an `n x k` generator matrix (Vandermonde or the cheaper Cauchy variant), where every `k x k` submatrix is invertible. Arranging the top `k` rows as the identity yields a *systematic* code, where the original data fragments appear verbatim, so happy-path reads need no decoding. Recovery stacks the `k` surviving rows, inverts that submatrix over the field, and multiplies to recover the data. Reed-Solomon is the canonical MDS (Maximum Distance Separable) code, sitting exactly on the Singleton bound for optimal storage efficiency.

The central tension is repair cost. Classical erasure coding must read `k` fragments to rebuild one lost fragment — a k:1 read amplification versus replication's 1:1 — which saturates network bandwidth and lengthens rebuild windows at data-center scale. Two research directions address this: Local Reconstruction Codes (LRC) add local parities so single-failure repair reads only a small group (Azure's `(12,2,2)` reads ~6 instead of 12), trading strict MDS optimality for cheaper common-case repair; and regenerating codes (MSR/MBR, via network coding) reduce how much is downloaded from each helper, with MSR Clay codes shipping in Ceph since Nautilus (2019).

Choosing `k` and `m` balances durability (`m`), storage bill (`n/k`), repair burden (bigger `k` = costlier repairs), and available failure domains (need at least `k+m`). Common defaults: 4+2 for resilience-first small clusters, 10+4 or wider for cost-sensitive scale. Critically, erasure coding is a durability mechanism against component failure — not backup (no protection from `rm`, bugs, ransomware) and not corruption handling on its own (it needs checksums and scrubbing to convert silent errors into known erasures).

RAID and erasure coding are conflated because classic RAID parity *is* erasure coding: RAID-5 is a `(k,1)` XOR code, RAID-6 a `(k,2)` Reed-Solomon code. The difference is scope, placement, and operational model. RAID protects disks inside one array with fixed parity levels and rebuilds whole disks onto hot spares; distributed erasure coding spreads tunable `k+m` fragments across independent domains (disk, host, rack, region), regenerates only lost fragments, and survives whole-rack or whole-site outages.

## Key takeaways
- Any `k` of `n = k + m` fragments reconstruct the original; the system tolerates up to `m` simultaneous losses.
- "Erasure" = a known-missing symbol (located), not an "error" (silently wrong, unlocated); this assumption makes the math cheap.
- 10+4 erasure coding tolerates 4 failures at 1.4x overhead vs 3x replication tolerating only 2.
- Encoding/decoding live in `GF(2^8)`; addition is XOR, multiply/divide use lookup tables or SIMD; `GF(2^8)` caps at 255 fragments.
- Systematic codes store data verbatim, so reads on the happy path cost no decode CPU.
- The repair problem: rebuilding one lost fragment requires reading `k` fragments (k:1 amplification), unlike replication's 1:1.
- LRC and regenerating codes (MSR/MBR) exist specifically to cut repair I/O, often sacrificing strict MDS.
- Erasure coding is not backup and does not catch silent corruption alone — it relies on checksums and scrubbing.
- It suits archival/cold/object storage, not hot low-latency primary workloads (degraded reads add decode latency).
- RAID is erasure coding in one box with fixed parity; distributed erasure coding generalizes to arbitrary `k+m` across independent failure domains.

## Concepts & entities
Wikilinks: [[erasure-coding]], [[reed-solomon]], [[raid]], [[redundancy]], [[data-durability]], [[redundancy-overhead]], [[object-storage]], [[decentralized-storage]], [[replication]], [[galois-field]], [[mds-code]], [[singleton-bound]], [[systematic-code]], [[local-reconstruction-codes]], [[regenerating-codes]], [[repair-bandwidth]], [[generator-matrix]], [[failure-domain]], [[scrubbing]], [[clay-codes]], [[fountain-codes]], [[ceph]], [[hdfs]], [[azure-storage]].

## Notable claims / data
- 3x replication = 3x storage, tolerates 2 failures; RS(10,4) = 1.4x storage, tolerates 4 failures.
- Storage overhead = `n/k`; fault tolerance = `m`. 4+2 = 1.5x, 10+4 = 1.4x, 17+3 = ~1.18x.
- `GF(2^8)` caps total fragments at `2^8 - 1 = 255` (zero element excluded for distinct nonzero evaluation points).
- Well-tuned SIMD Reed-Solomon encodes at several GB/s per core; SIMD vs naive table lookup can differ by an order of magnitude (Jerasure, Intel ISA-L, Backblaze's Java lib).
- Azure LRC `(12,2,2)`: 12 data in two groups of 6 (each with local parity) + 2 global parities = 16 fragments, 1.33x overhead, tolerates any 3 failures (most 4-failure patterns) — comparable to 12+4 RS at half the common-case repair traffic. Introduced by Microsoft Research ~2012.
- Regenerating codes from Dimakis et al. ~2010; MSR Clay codes (Vajha et al.) in Ceph from Nautilus (2019).
- Ceph's built-in default erasure profile is 2+2 (most deployments override to 4+2 or wider).
- Voyager probes used Reed-Solomon as an outer code concatenated with a convolutional inner code.
- RAID write hole (power loss between data and parity writes) historically patched with battery-backed caches or journaling; distributed systems use transactions, versioning, quorum instead.
- AWS S3 is widely believed to use erasure coding but Amazon has never published the scheme.

## Questions raised
- For which workloads does the LRC extra-parity space cost outweigh its repair-I/O savings (and vice versa)?
- How do MSR/MBR operating points compare in real Ceph deployments versus plain Reed-Solomon under sustained failure rates?
- What scrubbing cadence and checksum scheme is needed to keep silent-corruption risk acceptably low at exabyte scale?
- How do fragment placement strategies across failure domains interact with code parameters to bound correlated-failure risk?
- Where do fountain codes (LT/Raptor) fit relative to RS for decentralized/peer-to-peer storage networks?
