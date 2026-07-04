---
mindmap-plugin: basic
---

# Erasure Coding Algorithms: Comparison and Use Cases

## Key Principles
- Recover lost data without keeping full extra copies: split into `k` data + `m` parity, store `n = k + m`, rebuild from a surviving subset
- Every code trades off on one triangle: `storage overhead` vs `repair cost` vs `read/decode complexity` - no code wins all three
- `MDS` (Maximum Distance Separable) = optimal storage: any `k` of `n` suffice; RS is MDS, most fountain/LDPC codes are not
- The decisive question is often **repair cost**, not storage - that is why LRC and regenerating codes exist
- Erasures (known-position loss) are not errors (unknown-position corruption); corruption detection is a separate checksum layer

## Core Patterns
- **Fixed-rate, storage-optimal** (`Reed-Solomon`): minimum overhead, but repair reads `k` chunks
- **Local parity groups** (`LRC`): make the common single-loss repair cheap by reading only a nearby group
- **Bandwidth-optimal repair** (`regenerating MSR/MBR`): shrink bytes-per-helper, not number of helpers
- **Rateless streaming** (`fountain: LT/Raptor/RaptorQ`): emit unbounded symbols, decode once enough arrive - for unknown/lossy channels
- **Sparse-graph channel coding** (`LDPC`): near-capacity throughput at huge block sizes, tuned for transmission not storage

## Relationships
- **Replication** is the baseline the whole family is measured against (1:1 repair, high overhead)
- **LRC** = Reed-Solomon + local parity; a derivative that trades a little MDS optimality for cheaper repair
- **Regenerating codes** descend from **network coding** theory; attack a *different* repair cost than LRC (bandwidth vs read-count)
- **Fountain codes** drop the fixed-`n` assumption RS depends on; sit closer to transmission than storage
- **LDPC** and **BCH/convolutional/turbo** codes are communications cousins; LDPC is fixed-rate like RS but graph-decoded
- Storage picks (RS, LRC, Clay) vs transmission picks (fountain, LDPC): the split tracks "do you know `n` in advance?"

## Comparisons
- **Reed-Solomon** (MDS, `k:1` repair) vs **LRC** (near-MDS, cheap local repair) vs **regenerating** (MDS-capable, cheap-bandwidth repair)
- *Rateless* fountain codes vs ~~fixed-rate~~ RS/LDPC: rateless handles unknown loss but sacrifices the hard MDS guarantee
- Repair cost axis: replication (1:1) < LRC/regenerating (partial) < Reed-Solomon (`k:1`)

## Advantages
- **RS**: optimal storage, simple math, decode-free systematic reads, mature tooling
- **LRC**: much cheaper common-case repair at small storage premium
- **Regenerating**: cuts repair bandwidth for network-bound repair
- **Fountain**: no need to know `n`; tolerates unpredictable/asymmetric loss; RaptorQ is linear-time + near-optimal
- **LDPC**: near Shannon-capacity at large block sizes; fast hardware decoding

## Disadvantages
- **RS**: repair I/O scales with `k` (painful at high `k` or over WAN)
- **LRC**: worse multi-failure repair; higher overhead than MDS optimum; two-tier complexity
- **Regenerating**: complex subpacketization; few mature implementations
- **Fountain**: no hard MDS guarantee; LT overhead; RaptorQ decoding is complex to implement
- **LDPC**: hard to guarantee exact k-of-n recovery thresholds; not built for storage semantics

## Real-World Use Cases
- **Context or problem:** disk-array durability -> **Principle:** RS with `m=2` -> **Application:** RAID-6 -> **Outcome:** survives 2 disk losses
- **Context or problem:** cheap common-case repair in cloud storage -> **Principle:** local parity groups -> **Application:** Azure LRC (12,2,2) -> **Outcome:** single-loss repair reads ~6 fragments not 12
- **Context or problem:** network-bound repair -> **Principle:** regenerating MSR -> **Application:** Ceph Clay codes -> **Outcome:** ~2.9x less repair traffic than RS at 1.25x overhead
- **Context or problem:** lossy broadcast to unknown receivers -> **Principle:** rateless fountain -> **Application:** RaptorQ in ATSC 3.0 / 3GPP MBMS -> **Outcome:** reliable delivery without retransmission
- **Context or problem:** decentralized storage with churning peers -> **Principle:** storage favors MDS + mature tooling -> **Application:** Storj RS 29-of-80, Walrus 2-D RS -> **Outcome:** both chose RS despite fountain's rateless appeal

## Questions
- If rebuild bandwidth (not storage) is the bottleneck, which code and why? (LRC vs regenerating attack different parts of repair)
- Why can't a rateless fountain code also be strictly MDS?
- When would you deliberately pick replication over any erasure code?
- Azure's LRC (12,2,2) is not MDS yet beat RS for them - what was gained and given up?
- Why do P2P networks sound like a fountain-code fit yet Storj/Walrus chose RS?
- How does the erasure-vs-error distinction change which code you reach for?

## Learning Gaps
- Durability math (nines, MTTDL) that justifies a specific `(k, m)` is named but not worked through
- "P2P storage converges on RS" is an inference from two systems, not an established law
- Exact HDFS-EC parameters vary by deployment (RS(6,3)/RS(10,4) are common, not universal)
- Clay's 2.9x repair-traffic figure is one example at 1.25x overhead; ratios depend on `(n, k)`

---

# Detailed Study Notes

## Overview

Erasure coding is a way to survive losing pieces of your data without paying the full cost of keeping extra whole copies.
You take `k` chunks of original data, compute some extra `m` "parity" chunks, and store all `n = k + m`.
If any pieces are lost, you can rebuild the originals from a subset of what survives.
Replication is the brute-force version of the same goal (keep 3 copies); erasure coding gets the same durability for far less storage.

There are many algorithms because they all solve "recover from loss," but they optimize different things: how much storage overhead you pay for a given durability; how expensive repair is (when one chunk dies, how many other chunks and bytes must you read to rebuild it); whether you know how many pieces there are in advance (a fixed disk array knows, a peer-to-peer network where packets arrive unpredictably does not); and decode complexity and speed (matrix math vs sparse-graph passes).

The one mental model to keep: every choice sits on a triangle of **storage overhead vs repair cost vs read/decode complexity**.
No code wins all three; you pick based on whether your bottleneck is space, repair bandwidth, or an unreliable channel.

## Key Principles

- **The core mechanism:** split into `k` data chunks, add `m` parity, store `n = k + m`, and reconstruct from a surviving subset.
- **The tradeoff triangle:** storage overhead, repair cost, and read/decode complexity are in tension; every algorithm is a different point on it.
- **MDS and the Singleton bound:** a Maximum Distance Separable code lets any `k` of `n` chunks reconstruct - the theoretical best storage efficiency. Reed-Solomon is MDS; most fountain/LDPC codes are not.
- **Repair cost is often the real constraint:** RS is storage-optimal but its `k:1` repair cost dominates at scale - the entire reason LRC and regenerating codes exist.
- **Erasure vs error:** erasure codes handle known-location losses (a whole chunk gone); detecting unknown-position corruption is a separate layer (checksums).

Prerequisite vocabulary worth having first:

- Galois fields (`GF(2^8)`, `GF(2^16)`): the finite arithmetic behind RS-style coding; also explains the 255-fragment cap for `GF(2^8)`.
- Systematic vs non-systematic: whether original data is stored verbatim (decode-free reads) or only as encoded symbols.
- Rateless vs fixed-rate: fixed `n` known in advance vs an open-ended symbol stream.
- Repair/read amplification: the ratio of bytes read to bytes rebuilt - the metric that separates RS from LRC/regenerating codes.

## Core Patterns

- **Reed-Solomon (RS):** the classic. Optimal storage (any `k` of `n` chunks rebuild everything), but repairing one lost chunk means reading `k` chunks. Used in RAID-6, CDs, HDFS, Storj, Walrus.
- **Local Reconstruction Codes (LRC):** RS plus small "local" parity groups, so the common case (one chunk lost) reads only a few nearby chunks instead of `k`. Trades a little storage for much cheaper repair. Used in Azure Storage.
- **Regenerating codes (MSR/MBR):** reduce the bytes each helper sends during repair, not the number of helpers. Rooted in network coding. Shipped as Clay codes in Ceph.
- **Fountain codes (LT, Raptor, RaptorQ):** "rateless." Instead of a fixed `n`, they emit a potentially endless stream of encoded symbols, and the receiver reconstructs once enough symbols arrive, regardless of which ones. Built for lossy/unknown channels. Used in broadcast/streaming (ATSC 3.0, 3GPP MBMS).
- **LDPC:** sparse-graph codes decoded by fast iterative "belief" passes. Near-optimal at very large scale, mainly seen in communications (Wi-Fi, 5G, satellite) more than in storage.

## Relationships Between Topics

- **Replication** is the baseline the whole family is measured against.
- **Network coding** is the theory regenerating codes descend from.
- **Durability math (nines, MTTDL)** is what tells you which `(k, m)` to choose.
- **BCH / convolutional / turbo codes** are error-correction cousins (bit flips), as opposed to erasure codes (known-position losses).
- LRC is a derivative of RS; regenerating codes and LRC both cut repair cost but along different axes (bandwidth-per-helper vs number-of-chunks-read); fountain and LDPC lean toward transmission while RS/LRC/Clay lean toward storage.

## Expanded Explanation

Each family is a different answer to "which cost hurts most." The comparison table sets the axes; the advantages, disadvantages, and tradeoff notes explain the *why* behind each row.

Common misconceptions this expanded view corrects:

- "Erasure coding fixes bit errors." No: erasure codes handle known-location losses. Detecting corruption is a separate layer (checksums).
- "RS is always optimal, so always use it." Optimal on storage, but its `k:1` repair cost is often the real bottleneck at scale.
- "Fountain codes always need more data than the minimum." LT codes carry meaningful overhead, but RaptorQ recovers from exactly `k` symbols in most cases and only rarely needs a few more (per RFC 6330). They are near-MDS in practice, though not MDS by guarantee.
- "RaptorQ is what Walrus uses." The Walrus v1 paper proposed RaptorQ, but the shipped protocol uses Reed-Solomon.
- "More parity always means more safety per byte." Diminishing returns; placement and failure-correlation often matter more than raw `m`.

## Comparisons and Tradeoffs

| Code | Storage overhead | Repair cost | Rate model | Decode method | MDS? |
|---|---|---|---|---|---|
| Replication (baseline) | High (e.g. 3x for 3 copies) | Cheap (1:1, copy one replica) | Fixed | Trivial | N/A |
| Reed-Solomon | Low, tunable (`k`/`n`) | Expensive (`k`:1, read `k` chunks to rebuild 1) | Fixed `(k, n)` | Matrix algebra (Vandermonde/Cauchy) | Yes |
| LRC | Slightly higher than RS | Cheap for single loss (reads its local group, ~6 of 12 in Azure's (12,2,2)) | Fixed, with local + global parity | Matrix algebra in two tiers | No (trades MDS optimality away) |
| Regenerating (MSR/MBR) | Comparable to RS | Cheaper bandwidth per repair (helpers send partial data; Clay ~2.9x less traffic than RS at 1.25x overhead) | Fixed | Linear algebra over subpacketized data | MSR: yes. MBR: no |
| Fountain (LT/Raptor/RaptorQ) | Small reception overhead (RaptorQ usually decodes from exactly `k`, rarely `k`+a few) | N/A in the classic sense; receiver waits for more symbols instead of running a repair step | Rateless, unbounded symbol stream | Belief propagation (LT) or inactivation decoding (Raptor/RaptorQ) | No (near-MDS in practice) |
| LDPC | Low, tunable | N/A, designed for one-shot channel decoding, not storage-style repair | Fixed | Iterative sparse-graph belief propagation | No, but near-capacity at scale |

**When each is / isn't the right choice:**

- Choose **replication** when simplicity and repair speed dominate and storage cost is a non-issue (small hot datasets, metadata).
- Choose **Reed-Solomon** when storage efficiency is the priority and `k` is moderate, or when reads need to be decode-free (systematic). The default for disk/object storage (RAID-6, HDFS-EC, Storj, Walrus).
- Choose **LRC** when single-node/single-disk failure is the dominant failure mode and repair I/O (not storage) is the pain point. This is Azure's production motivation.
- Choose **regenerating codes** when cross-datacenter or network-bound repair bandwidth is the bottleneck rather than local disk I/O. Ceph's Clay code targets this.
- Choose **fountain codes** when the receiver set or loss pattern is unknown/unbounded ahead of time: broadcast/multicast streaming, satellite, lossy P2P transport. A small overhead buys freedom from retransmission protocols.
- Choose **LDPC** for real-time communication channels needing near-capacity throughput at large block sizes (Wi-Fi, 5G, DVB). A much less natural fit for storage's "any k of n" model.
- **The lesson for decentralized/P2P storage:** despite fountain codes' rateless appeal for unpredictable peer availability, both Storj and Walrus landed on Reed-Solomon in production. This suggests that for storage (vs transmission), MDS optimality and mature tooling outweigh the rateless convenience once the "channel" (the storage network) is reasonably provisioned and `n` can be fixed in advance. *This is an inference drawn from two examples, not a settled industry-wide law.*

## Real-World Examples and Use Cases

| System / medium | Code used | Notable parameters |
|---|---|---|
| RAID-6 | Reed-Solomon | `m = 2` parity (survives 2 disk losses) |
| CDs / DVDs, QR codes, Voyager deep-space | Reed-Solomon | Classic bounded-symbol RS |
| HDFS-EC | Reed-Solomon | Common config RS(6,3) or RS(10,4) |
| Azure Storage | LRC | `(12,2,2)`: two groups of 6, 2 local + 2 global parities; single-loss repair reads ~6 fragments |
| Ceph | Clay code (MSR regenerating) | Available as a plugin since Nautilus; ~2.9x less repair traffic than RS at 1.25x overhead |
| Storj | Reed-Solomon | 29-of-80 (~2.75x expansion); heals by re-encoding whole files |
| Walrus | Reed-Solomon | 2-D "Red Stuff" RS over `GF(2^16)`, ~4.5x expansion (v1 paper proposed RaptorQ; shipped uses RS) |
| ATSC 3.0 / 3GPP MBMS broadcast, datacasting | RaptorQ (RFC 6330) | Systematic fountain code, up to 56,403 source symbols |
| Wi-Fi, 5G, DVB satellite | LDPC | Channel coding, not storage |

## Advantages and Disadvantages

**Advantages**

- **Reed-Solomon:** optimal storage efficiency (MDS); simple, well-understood math; systematic form gives decode-free reads; mature tooling everywhere.
- **LRC:** dramatically cheaper common-case repair (single-chunk loss, the overwhelmingly frequent failure) at a small storage premium.
- **Regenerating codes:** cuts repair bandwidth, which matters when the network, not the disk, is the bottleneck (e.g. multi-datacenter repair).
- **Fountain codes:** no need to know `n` in advance; naturally tolerant of unpredictable, asymmetric loss; a sender can just keep emitting symbols. RaptorQ adds linear-time encode/decode and near-optimal recovery.
- **LDPC:** near Shannon-capacity performance at very large block sizes; very fast iterative decoding in hardware; dominant in real-time communication channels.

**Disadvantages**

- **Reed-Solomon:** repair I/O scales with `k`, painful at high `k` (e.g. 29-of-80 at Storj) or across WAN links. Storj notably relies on re-encoding whole files to heal lost pieces rather than cheap in-place repair.
- **LRC:** worse worst-case (multi-failure) repair than pure RS; storage overhead higher than the MDS optimum; more complex encode/decode logic (two parity tiers).
- **Regenerating codes:** mathematically more complex to implement (subpacketization); fewer mature open-source implementations (Ceph's Clay code is a notable exception); less battle-tested than RS.
- **Fountain codes:** not MDS by guarantee; LT alone has meaningful overhead and is rarely used unmodified; RaptorQ's inactivation decoding is more complex to implement correctly than RS matrix math.
- **LDPC:** for storage-style "any k-of-n" reconstruction, sparse-graph designs are harder to guarantee exact recovery thresholds than RS's clean algebraic guarantee; mostly tuned for communication channels, not disk/object storage semantics.

## Questions or Review Prompts

1. If storage is cheap but rebuild bandwidth is the bottleneck, which code do you pick and why? (Hint: LRC vs regenerating codes attack *different* parts of repair cost.)
2. Why can't a rateless fountain code also be strictly MDS?
3. When would you deliberately choose replication over any erasure code?
4. Azure's LRC (12,2,2) is *not* MDS, yet Azure chose it over pure Reed-Solomon. What did they gain, and what did they give up?
5. Why does a decentralized/P2P storage network (unknown, churning node set) sound like a natural fit for fountain codes, yet Storj and Walrus both chose Reed-Solomon?
6. What is the difference between an erasure (known-position loss) and an error (unknown-position corruption), and why does that distinction change which code you reach for?

## Open Gaps or Next Topics

- Durability math (nines, MTTDL) is referenced as the basis for choosing `(k, m)` but not worked through in the source.
- The "P2P storage converges on Reed-Solomon" conclusion is an interpretation from two systems (Storj, Walrus), not an industry-wide law.
- Exact HDFS-EC parameters vary by deployment; RS(6,3)/RS(10,4) are common defaults, not universal.
- Precise repair-traffic ratios for Clay codes depend on chosen `(n, k)`; the 2.9x figure is one published example at 1.25x overhead.

## Sources and Verification

Verified against the following sources during a targeted integrity pass; no contradiction with these sources was found for the claims above.

- RaptorQ / fountain codes: [RFC 6330](https://www.rfc-editor.org/rfc/rfc6330), [Raptor code (Wikipedia)](https://en.wikipedia.org/wiki/Raptor_code)
- Azure LRC: [Erasure Coding in Windows Azure Storage, USENIX ATC'12](https://www.usenix.org/system/files/conference/atc12/atc12-final181_0.pdf)
- Regenerating / Clay codes: [Ceph CLAY plugin docs](https://docs.ceph.com/en/nautilus/rados/operations/erasure-code-clay/), [Clay Codes, USENIX FAST'18](https://www.usenix.org/conference/fast18/presentation/vajha)
- Storj / Walrus parameters: [Storj file-redundancy docs](https://storj.dev/learn/concepts/file-redundancy), [Walrus paper (arXiv 2505.05370)](https://arxiv.org/pdf/2505.05370)
