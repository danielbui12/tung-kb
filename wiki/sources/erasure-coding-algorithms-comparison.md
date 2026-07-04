---
title: Erasure Coding Algorithms: Comparison and Use Cases
type: source
tags: [erasure-coding, reed-solomon, lrc, regenerating-codes, fountain-codes, ldpc, mds-code]
created: 2026-07-04
author: danielbui12 (composed study notes)
source_url: "local: raw/Erasure-coding-algorithms-comparision.composed.md"
source_type: note
ingested: 2026-07-04
---
# Erasure Coding Algorithms: Comparison and Use Cases

**One-line:** A comparative survey of the erasure-coding family — Reed-Solomon, LRC, regenerating (MSR/MBR), fountain (LT/Raptor/RaptorQ), and LDPC — organized around one triangle (storage overhead vs repair cost vs decode complexity) and one dividing question: do you know `n` in advance?

## Summary
Every erasure code answers "recover from loss" the same structural way (`k` data + `m` parity → `n = k+m`, rebuild from a surviving subset) but optimizes different corners of a storage-overhead / repair-cost / decode-complexity triangle — no code wins all three. **MDS** (Maximum Distance Separable, the Singleton bound) is the ceiling: any `k` of `n` suffice with zero wasted redundancy. Reed-Solomon sits exactly on this bound; most fountain and LDPC codes do not.

Reed-Solomon is storage-optimal but pays `k:1` repair (reading `k` chunks to rebuild one) — the real bottleneck at scale, and the reason two derivative families exist: **Local Reconstruction Codes (LRC)** add small local parity groups so the common single-loss case reads only a nearby group instead of all `k` (Azure's `(12,2,2)` reads ~6 of 12), trading away strict MDS optimality; **regenerating codes (MSR/MBR)**, descended from network coding, instead shrink the *bytes* each helper sends without reducing how many helpers are contacted — Ceph's Clay codes report ~2.9x less repair traffic than RS at 1.25x overhead.

**Fountain codes** (LT, Raptor, RaptorQ) drop the fixed-`n` assumption entirely: a sender emits a potentially unbounded stream of symbols and a receiver decodes once *any* sufficient subset arrives, regardless of which. This suits unknown/lossy/broadcast channels (RaptorQ in ATSC 3.0, 3GPP MBMS) but sacrifices the hard MDS guarantee — RaptorQ is near-MDS in practice (decodes from `k` symbols, rarely needs a few more per RFC 6330) but not MDS by guarantee. **LDPC** is a sparse-graph, fixed-rate, graph-decoded cousin tuned for near-Shannon-capacity channel throughput (Wi-Fi, 5G, satellite) rather than storage's "any k of n" model.

The storage/transmission split tracks whether `n` is known in advance: disk arrays and object stores (which know their fragment count) pick RS/LRC/Clay; broadcast and lossy-P2P channels (which don't) pick fountain/LDPC. Notably, despite the rateless appeal of fountain codes for churning P2P peer sets, both Storj (RS 29-of-80) and Walrus (2-D RS) chose Reed-Solomon in production — suggesting MDS optimality and mature tooling outweigh rateless convenience once the storage "channel" is provisioned enough to fix `n` in advance. This mirrors and reinforces the same conclusion already on [[decentralized-storage-networks]], now with an explicit reason (the storage-vs-transmission split) rather than just an observed pattern.

## Key takeaways
- Every code trades off storage overhead, repair cost, and read/decode complexity — pick based on the actual bottleneck.
- MDS = optimal storage (any `k` of `n` suffice); RS is MDS, most fountain/LDPC codes are not.
- Repair cost, not storage, is often the deciding factor — the whole reason LRC and regenerating codes exist.
- LRC and regenerating codes attack *different* parts of repair cost: read-count (LRC) vs bytes-per-helper (regenerating/MSR-MBR).
- Fountain codes trade the hard MDS guarantee for not needing to know `n` in advance — built for transmission, not storage.
- LDPC is fixed-rate like RS but graph-decoded; near-capacity at huge block sizes, tuned for channels not disks.
- Erasure (known-position loss) vs error (unknown-position corruption) is a distinct axis from all of the above; corruption detection is a separate checksum layer.
- Storj and Walrus both chose RS over fountain codes despite fountain's apparent fit for churning P2P peer sets.

## Concepts & entities
[[erasure-coding]], [[reed-solomon]], [[mds-code]], [[local-reconstruction-codes]], [[regenerating-codes]], [[fountain-codes]], [[ldpc]], [[repair-bandwidth]], [[raid]], [[ceph]], [[hdfs]], [[storj]], [[walrus]], [[azure-storage]].

## Notable claims / data
- RAID-6 = RS with `m=2`, survives 2 disk losses.
- Azure LRC `(12,2,2)`: single-loss repair reads ~6 fragments instead of 12.
- Ceph Clay (MSR regenerating): ~2.9x less repair traffic than RS at 1.25x overhead.
- RaptorQ (RFC 6330) used in ATSC 3.0 and 3GPP MBMS broadcast/datacasting, up to 56,403 source symbols, systematic and near-linear-time.
- Storj: RS 29-of-80. Walrus: 2-D "Red Stuff" RS over `GF(2^16)`. Both explicitly chose RS despite the Walrus v1 paper having proposed RaptorQ.
- HDFS-EC common configs RS(6,3)/RS(10,4) are typical, not universal — vary by deployment.

## Questions raised
- If rebuild bandwidth (not storage) is the bottleneck, LRC or regenerating codes — and by which axis (read-count vs bytes-per-helper)?
- Why can't a rateless fountain code also be strictly MDS?
- When would you deliberately pick replication over any erasure code?
- What did Azure gain and give up by choosing non-MDS LRC `(12,2,2)` over pure RS?
- How does the erasure-vs-error distinction change which code family is even in scope?

## Sources verified against
RFC 6330, Raptor code (Wikipedia), Azure LRC (USENIX ATC'12), Ceph CLAY docs + Clay Codes (USENIX FAST'18), Storj file-redundancy docs, Walrus paper (arXiv 2505.05370).
