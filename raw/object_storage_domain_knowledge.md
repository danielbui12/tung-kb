---
mindmap-plugin: basic
---

# Decentralized Object Storage — Domain Knowledge

> Study-structured edition of the Storage-Model deep dive on **Storj, Filecoin, Arweave, Walrus, and Swarm**. The mindmap below is the memorable summary; every claim is expanded in the sections beneath it. Source: `object_storage_model.md`.

## Key Principles
- **No node can be trusted** → cryptography + token incentives replace the trusted operator (this is what makes it a `DSN`).
- **The master fork is the redundancy model** — three families: `erasure coding`, `replication`, and `address-proximity replication` — and that choice cascades into overhead, retrieval, churn-handling, and what the proofs must prove.
- **Content addressing is self-certifying** — naming data by its hash means integrity needs no trusted server (`CID`, `data_root`, `Blob ID`, BMT chunk address).
- **Redundancy model dictates proof model** — replication ⇒ prove copies are *unique*; erasure coding ⇒ prove *fragment possession*; neighborhood replication ⇒ prove *reserve size*.
- **Storage must be proven over time, without re-reading the data** — the core cryptographic puzzle of every DSN.
- **Economics makes the cryptography binding** — slashing / escrow / endowment / postage is why honest storage is the rational choice.

## Core Patterns
- **Encrypt-then-split** (Storj, client-side, zero-knowledge) vs **public content-addressed ingest** (Filecoin/Arweave/Walrus/Swarm).
- **`k`-of-`n` threshold coding** — rebuild from any sufficient subset (Storj 29-of-80; Walrus `f+1`/`2f+1`).
- **Address-determines-placement** — a chunk's hash decides which nodes store it (Swarm Kademlia neighborhood); routing, dedup, and auto-scaling fall out of this.
- **Unique-replica proofs** (Filecoin `SDR` sealing, Arweave `RandomX` packing) vs **fragment-possession audits** (Storj sampling, Walrus challenges) vs **reserve-size proof** (Swarm density sampling).
- **Quorum read + long-tail cancellation** — fetch more than needed, keep the fastest, ignore slow/offline nodes.
- **2-D self-healing** — recover a lost fragment from a thin slice of peers (Walrus `O(blob/n)`), not the whole object.
- **Proof fused with consensus** (Arweave mining = proof; Swarm Schelling round) vs **standalone periodic proof** (Filecoin `WindowPoSt` every 24h).
- **Pay-once endowment** (Arweave permanence) vs **recurring rent** (Storj/Filecoin/Walrus) vs **decaying postage + "eventually forget"** (Swarm).

## Relationships
- **Spectrum:** private efficient cloud (**Storj**) → verifiable cold storage (**Filecoin**) → permanent public archive (**Arweave**) → self-healing storage + data-availability (**Walrus**) → permissionless P2P web platform (**Swarm**).
- **IPFS sits *under* Filecoin** — content-addressing/transport substrate with no persistence; Filecoin is the paid persistence layer beneath it. (The five analyzed are all full storage networks; IPFS is the substrate.)
- **Causal chain:** `Redundancy (Dim 2)` → `Proofs (Dim 3)` → `Retrieval (Dim 4)` are not independent — each is downstream of the redundancy choice.
- **Storage vs Data Availability** — Walrus targets both (on-chain `Point of Availability`); the others mainly target persistence.
- **Permanence vs forgetting** — Arweave keeps data forever; Swarm deliberately *forgets* unpaid data — opposite ends of the durability-lifetime axis.

## Comparisons
- **Erasure coding** (Storj ~2.76×, Walrus ~4.5×, quorum read) vs **replication** (`R`× cost, single-copy read) vs **neighborhood replication** (Swarm: address-placed replica set + optional RS).
- **Poseidon** (SNARK-friendly, Filecoin replica/column trees) vs *SHA-256* (Filecoin piece/data tree only).
- **Reed–Solomon** (Walrus *current* code; Swarm *optional file-level*) vs ~~RaptorQ~~ (Walrus *v1 paper only*).
- **One-time fee + endowment** (Arweave) vs *recurring per-GB / per-deal / per-epoch* (Storj/Filecoin/Walrus) vs *decaying postage + bandwidth* (Swarm).
- **Slashable posted collateral** (Filecoin FIL, Walrus WAL, Swarm BZZ) vs *withheld escrow* (Storj) vs *positive-only rewards* (Arweave).

## Advantages
- **Storj:** zero-knowledge privacy, low overhead, fast parallel reads, cash-flow pricing.
- **Filecoin:** cryptographically verifiable at massive scale; cheap cold storage.
- **Arweave:** true permanence, pay-once, web-simple retrieval, strongest churn tolerance.
- **Walrus:** BFT-grade durability at ~4.5× with cheap self-healing; native on-chain DA.
- **Swarm:** permissionless web/dapp hosting; address-routed dedup + auto-scaling; strong privacy (anonymous browsing, no metadata leakage); BitTorrent-like resilience.

## Disadvantages
- **Storj:** trust in the satellite for coordination; recurring cost; not "permanent."
- **Filecoin:** capital-intensive; slow unsealing makes base-layer retrieval non-interactive; manual repair.
- **Arweave:** permanence bets on centuries of endowment solvency; high real replication cost; public-by-default.
- **Walrus:** newest/least battle-tested; security depends on stake decentralization.
- **Swarm:** "eventually forgets" unpaid data (no native permanence); postage-batch + dual-incentive UX complexity; smaller per-neighbourhood replica set.

## Real-World Use Cases
- **Context:** S3-compatible private app storage → **Pattern:** encrypt-then-erasure-code → **Application:** Storj as a drop-in S3 backend → **Outcome:** confidential, durable, egress-priced object store.
- **Context:** archival of large datasets cheaply with proof → **Pattern:** sealed-sector replication + PoSt → **Application:** Filecoin storage deals (+ Filecoin Plus) → **Outcome:** verifiable multi-copy cold storage.
- **Context:** permanent records (NFTs, legal docs, web archives) → **Pattern:** pay-once endowment + blockweave → **Application:** Arweave permaweb → **Outcome:** data guaranteed for ~200 years, fetched like a URL.
- **Context:** rollup/blockchain data availability → **Pattern:** 2-D coding + on-chain PoA → **Application:** Walrus as a DA layer for Sui → **Outcome:** provably available data at ~4.5× cost with self-healing.
- **Context:** censorship-resistant websites/dapps + private messaging → **Pattern:** address-routed chunks + ENS + feeds/PSS → **Application:** Swarm permissionless web hosting → **Outcome:** decentralized, anonymous, auto-scaling site serving paid for via postage + bandwidth.

## Questions
- Why does "quorum to read" fall directly out of erasure coding, and "one copy to read" out of replication?
- Why must Filecoin prove replica *uniqueness* while Storj need not?
- What property of Poseidon matters *inside* a zk-SNARK that SHA-256 lacks?
- Why is Walrus recovery `O(blob/n)` rather than `O(blob)`?
- How does Swarm's reserve-sample maximum prove a node stores *enough* chunks without challenging individual chunks?
- If storage costs stopped falling, which protocol's economics is most at risk?

## Learning Gaps
- ⚠️ **Storj repair threshold `m`** is version/satellite-dependent (code 35, SaltLake 52, glossary 54) — not a fixed constant.
- ⚠️ **Walrus row/column labeling** differs between whitepaper and source code (math is invariant; naming isn't).
- ⚠️ **Swarm `NHOOD_PEER_COUNT = 4`** (Sept-2025 spec) vs the 2021 whitepaper's ~8-node neighbourhood — version/context-dependent.
- Real-world **benchmark latency/throughput numbers** are not in scope here — would need follow-up.
- **Walrus & Swarm** figures are recent (Walrus 2025 mainnet; Swarm constants from a 2025 spec draft) and may shift; Swarm facts verified against user-provided docs, not the live Bee codebase.

---

# Overview

Five decentralized object-storage networks solve the same problem — *store a file durably on a network of untrusted strangers* — with sharply different bets. This document analyzes each across **four core dimensions** (ingestion, redundancy, proofs, retrieval) plus a **cross-cutting economics dimension**, and then opens every box with an **implementation-grade deep dive**.

**The one idea to keep:** the **redundancy model** is the master variable, and the five span all three families — **erasure coding** (Storj 1-D, Walrus 2-D), **explicit replication** (Filecoin deals, Arweave weave), and **address-proximity neighborhood replication** (Swarm). It sets storage overhead, decides whether reads need a quorum / a single copy / a routed lookup, shapes how node churn is absorbed, and even determines what the proof system must prove (unique replicas, fragment possession, or sufficient reserve size).

> **Scope note:** the original brief named "five" protocols but listed four; **Swarm** is now included as the fifth, completing the set. IPFS is the usual *substrate* (addressing/transport, no persistence) and is described in Background, not analyzed as a peer.

---

# Background — What "Decentralized Object Storage" Means

**Object vs. file vs. block storage.**
- **Block storage** — raw fixed-size blocks (a virtual disk); a filesystem sits on top. Low-level (e.g. AWS EBS).
- **File storage** — hierarchical directories/files with POSIX semantics (e.g. NFS).
- **Object storage** — flat namespace of **immutable objects** (bytes + metadata + ID), accessed over HTTP-style APIs, no in-place edits. The S3 model. **All five protocols are object stores** — `PUT` a blob, get an ID, `GET` it back.

**What makes them a DSN (Decentralized Storage Network).** Bytes live on a permissionless network of independently-run nodes coordinated by **cryptography + economic incentives** instead of trust. Three problems follow, which the dimensions map onto:
1. Can't trust any node → encrypt and/or content-address, then split (Dim 1).
2. Nodes vanish or cheat → add redundancy (Dim 2) and *prove* storage cryptographically (Dim 3).
3. No central index → coordinate retrieval via content hashes / on-chain metadata / gateways / address-routing (Dim 4); pay for it with a token (Dim 5).

**Content addressing — the shared backbone.** Most DSNs name objects by the **hash of their content** (CID in IPFS/Filecoin, `data_root`/TX-id in Arweave, Blob ID in Walrus, BMT chunk hash in Swarm). It's *self-certifying*: re-hash what you received to confirm integrity, no trusted server needed. **Storj is the exception** — it addresses by *encrypted path* because content is encrypted client-side.

**Storage vs. Data Availability (DA).** *Storage* = "bytes persist and can be retrieved later." *DA* = "bytes are provably published and retrievable **right now**." Walrus targets both (its on-chain Point of Availability); Filecoin/Arweave are primarily persistence; Storj is hot operational storage; Swarm is operational P2P serving.

**Where IPFS fits.** IPFS is **not** storage — it's a content-addressing + P2P *transport* protocol with **no persistence guarantee** (data lives only while pinned). **Filecoin is the incentive/persistence layer under IPFS.** IPFS belongs *beneath* this list as the addressing substrate; the five analyzed here (including **Swarm**, a full P2P storage network with its own DISC layer) are the apples-to-apples comparison.

---

# Dimension 1 — Data Ingestion & Splitting

> *How is the file handled the moment it leaves the client? Encrypted locally? Sliced how? Which primitives?*

### Storj — encrypt-then-erasure-code, all client-side (zero-knowledge)
1. **Client-side encryption (mandatory).** Object encrypted with **AES-256-GCM** before leaving the device; keys derived from a passphrase via hierarchical deterministic derivation. **Paths and metadata are encrypted too** — the satellite sees only ciphertext. Hence "zero-knowledge."
2. **Segmenting** into ≤ **64 MB** segments.
3. **Striping** each segment into fixed-size **stripes** (the RS encoding boundary).
4. **Erasure encoding** each stripe with Reed–Solomon → concatenated into **pieces**; **80 pieces, any 29 rebuild** the segment.

*Net:* the smallest unit (a piece) is an encrypted, erasure-coded fragment — useless without ≥29 pieces *and* the passphrase.

### Filecoin — content-addressed public data
1. **No native encryption** (public; encrypt yourself if needed).
2. **Chunk → IPLD/UnixFS Merkle DAG** (~256 KiB chunks) → root **CID**.
3. **Serialize into a CAR file.**
4. **Piece prep (`CommP`):** Fr32-pad (254→256 bits for the BLS12-381 field) → binary **SHA-256 (trunc-254)** Merkle tree → **Piece CID**; padded to a power of two.
5. **Sector packing** into **32/64 GiB** sectors — the proving unit.

*Net:* a content-addressed, Merkle-committed piece whose `CommP` anchors all later proofs.

### Arweave — permanent public permaweb
1. **No native encryption** (and no delete — encrypt sensitive data first).
2. **Transaction-based ingestion** (`data` field + tags).
3. **Chunking** into ≤ **256 KiB** chunks.
4. **Merkle commitment** → `data_root`; each chunk carries a Merkle path (enables offset challenges).
5. **Bundling (ANS-104)** — many data items per transaction (via Irys/Bundlr).

*Net:* ≤256 KiB chunks anchored to a `data_root`, woven permanently into the global *blockweave*.

### Walrus — 2-D "Red Stuff" on Sui
1. **No mandatory encryption** (opt-in **Seal** layer).
2. **Blob ID** derived from the erasure-encoded commitment; lifecycle registered on **Sui**.
3. **2-D matrix** formation — the defining feature.
4. **Primary** + **secondary encoding** (two orthogonal axes) → each node gets **one sliver pair**; with `n = 3f+1` nodes the blob becomes an `(f+1) × (2f+1)` matrix.

*Net:* blob expanded ~4.5× into per-node sliver pairs whose 2-D structure enables cheap self-healing.

### Swarm — everything is a ≤4 KB chunk (the DISC)
1. **Optional client-side encryption** (pad-to-4 KB + encrypt → indistinguishable from random; the *reference* carries the decryption key).
2. **Chunking** into **≤4096-byte** payloads (+ 8-byte span) — the universal unit.
3. **Addressing — two chunk types:** *content-addressed* (address = hash of span + **BMT** hash of payload) and *single-owner* (address = `hash(id, owner)`, owner-signed → basis for mutable **feeds**).
4. **Files → 128-ary Swarm hash tree** (each intermediate chunk packs up to 128 child references); **file address = root chunk hash**.
5. **Collections → manifests** (compacted Merkle trie) → URL routing; **ENS** for human names.

*Net:* the object becomes many ≤4 KB content-addressed chunks under a Merkle root; the **chunk is the universal unit** for storage, redundancy, proofs, and retrieval.

### Comparison — Dimension 1
| | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **Encrypted by default?** | ✅ AES-256-GCM (incl. paths/metadata) | ❌ public | ❌ public | ❌ public (opt-in Seal) | ❌ public (optional client-side) |
| **Splitting unit** | Segment → stripes → pieces | Chunks → DAG → Piece → Sector | Chunks (≤256 KiB) | 2-D matrix → sliver pairs | ≤4 KB chunks → 128-ary hash tree |
| **Redundancy method** | Reed–Solomon (1-D) | Replication (sealed sectors) | Replication (weave) | Red Stuff (2-D) | Neighborhood replication (+ optional RS) |
| **Content addressing** | Encrypted paths | CID + Piece CID | `data_root` / TX-id | Blob ID (on Sui) | BMT hash / single-owner address |
| **Core primitives** | AES-256-GCM, HMAC KDF, RS | SHA-256 Merkle, BLS12-381, Fr32 | SHA Merkle, ANS-104 | 2-D RS, Merkle commitments | BMT (Keccak), ECDSA, manifests |

---

# Dimension 2 — Distribution & Redundancy

> Three families: **erasure-coded** (Storj, Walrus — fragments, low overhead, quorum read), **replicated** (Filecoin, Arweave — whole copies, simple, costly), and **address-proximity replication** (Swarm — the chunk's address picks its replica set).

### Storj — Reed–Solomon `(29 / 35 / 80 / 110)`
| Symbol | Value | Name | Meaning |
|---|---|---|---|
| k | 29 | Minimum | pieces to reconstruct |
| m | 35 | Repair threshold | repair triggers here (⚠️ version-dependent) |
| o | 80 | Optimal | upload succeeds at 80 placed |
| n | 110 | Total | coded & pushed in parallel; keep first 80 (long-tail cancellation) |

- **Overhead ≈ 80/29 ≈ 2.76×**; survives losing **51 of 80** pieces; ~**11 nines** durability.
- **Diversity:** 80 distinct nodes, no two pieces in the same /24 subnet or operator.
- **Repair loop:** at threshold, satellite reconstructs missing pieces and re-distributes — redundancy is *maintained*.

### Filecoin — replication via sealed sectors
- Unit = **sector** (32/64 GiB). **Each replica is cryptographically unique** (sealing binds it to provider identity).
- **Replication factor = client's choice** (`R` deals with `R` distinct miners). **No automatic repair** — re-deal to restore.
- *Trade-off:* fast single-copy reads, but cost scales linearly (5 copies = 5×).

### Arweave — full replication via the blockweave
- Each block references the previous block **and** a random **recall block**; to mine you must prove access to that recall data → rational miners **store as much of the weave as possible**, strongest incentive for *rare* data.
- **No fixed factor, no rent.** Pay once → **endowment** funds ~200 years; protocol assumes a conservative **0.5%/yr** cost decline (⚠️ *not* ~30% historical).

### Walrus — 2-D Red Stuff across `n = 3f+1`
- BFT committee of `n = 3f+1` (stake-weighted on Sui), tolerates `f` Byzantine; each node holds one **sliver pair**.
- Blob = `[f+1, 2f+1]` matrix; reconstruct from **`f+1`** (one dimension) or **`2f+1`** (other).
- **Cheap self-healing:** a node recovers a lost sliver from **row/column intersection symbols** (~1/n of the blob), not the whole object.
- **~4.5× overhead** for BFT durability; shards reassigned per epoch.

### Swarm — address-proximity neighborhood replication (+ optional RS)
- **DISC rule:** each chunk is stored by the nodes whose **overlay address is Kademlia-closest** to the chunk address — its **neighborhood**; **pull-sync** keeps every neighbor holding a redundant copy.
- **Redundancy factor = neighborhood size** (≥ `NHOOD_PEER_COUNT = 4`; whitepaper says ~8). As the network grows, the **storage radius** deepens so neighborhoods stay small but numerous (auto-scaling). Fixed per-node **reserve** (`NODE_RESERVE_DEPTH = 23` → ~2²³ chunks).
- **Auto-scaling:** forwarders may **opportunistically cache** popular chunks.
- **Optional file-level erasure coding:** per-level **Reed–Solomon** in the hash tree (e.g. **m = 112 of 128 → 16 parity chunks**), configurable redundancy level; **RACE mode** bounds retrieval latency.
- **"Eventually forgets":** chunks whose postage stamp lapses are garbage-collected — protects *paid* content, not everything forever.

### Comparison — Dimension 2
| | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **Class** | Erasure (1-D RS) | Replication (sealed) | Replication (incentive) | Erasure (2-D) | Address-proximity replication (+ optional RS) |
| **Key param** | 29/35/80/110 | `R` deals | ~full network | `[f+1,2f+1]`, `n=3f+1` | neighbourhood ≥4 replicas; optional RS m/128 |
| **Overhead** | ~2.76× | `R`× | high | ~4.5× | ~neighbourhood size (+ parity if RS on) |
| **Fault tolerance** | lose 51/80 | lose `R−1` providers | ≥1 miner keeps it | up to `f` Byzantine | until a neighbourhood drops below redundancy |
| **Repair** | satellite at threshold 35 | manual (new deal) | emergent (incentive) | cheap 2-D self-heal | continuous pull-sync + pinning/recovery |

---

# Dimension 3 — Cryptographic Verification & Proofs

> Defeat two cheats: **generate-on-demand** (discard, refetch on challenge) and **single-copy/Sybil** (claim N copies, keep 1). Each proof system is shaped by its redundancy model. *("Proof of Resiliency" in the brief isn't standard — the real analogue is Arweave's SPoRes.)*

### Storj — random sampling audits + reputation (light)
- Random **stripe** challenged; nodes return one erasure share; **Reed–Solomon/Berlekamp–Welch consistency** catches wrong or missing data. Anchored by signed **SHA-256 piece hashes** + order limits (no Merkle tree in prod audits).
- Failures drop **beta-reputation** → **disqualification** + escrow forfeiture. Probabilistic spot-checking, not zero-knowledge proofs.

### Filecoin — PoRep + PoSt (heavy, zk-SNARK)
- **PoRep (once, at sealing):** slow, memory-hard **SDR** sealing → provably **unique** replica; compressed to a Groth16 zk-SNARK. Commitments `CommD`/`CommR`.
- **PoSt (over time):** **WindowPoSt** proves *every* sector every **24h** (48 × 30-min deadlines); **WinningPoSt** ties a fast proof to block rewards (Expected Consensus). Misses → slashing.

### Arweave — PoA → SPoRA → SPoRes (fused with mining)
- **PoA/SPoA:** must access a random recall block to mine. **SPoRA:** mining throughput ∝ (bytes stored × disk speed).
- **Packing (RandomX):** per-replica unique encoding (anti-dedup). **VDF hash chain:** rate-limits hashing so storage, not hashpower, wins.
- **SPoRes:** proves storage of *N unique copies*; detection independent of dataset size.

### Walrus — PoA + async challenge protocol (on Sui)
- **Point of Availability:** collect **2f+1 signed acks** → **Certificate of Availability** published on Sui (the storage-guarantee start).
- **Async challenges:** random blob subsets challenged each epoch; **cost ~logarithmic** in stored blobs. Self-healing cheaply restores anything lost.

### Swarm — postage stamps + redistribution (Schelling) game
- **Postage stamps = proof of *payment*** (right to occupy reserve space; set GC priority) — not proof of storage.
- **Redistribution game = proof of *storage*:** a **staked commit-reveal Schelling game** on-chain (Gnosis), ~every `ROUND_LENGTH = 152` blocks. A pseudo-randomly selected **neighborhood** (anchor = **XOR of revealed nonces**) reveals a **reserve sample** (`2^SAMPLE_DEPTH = 16` lowest transformed addresses); the agreeing "truth" wins a stake-weighted **pot** of postage revenue.
- **Proof of reserves (density estimation):** bounding the sample's max value statistically forces a **minimum reserve size** — a node storing too few chunks can't produce a dense-enough sample. No per-chunk or zk proofs.
- **Penalty:** non-revealers / dishonest stakers are **slashed**.

### Comparison — Dimension 3
| | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **Named proof(s)** | Audits + reputation | PoRep + PoSt | PoA/SPoA→SPoRA→SPoRes | PoA + challenge protocol | Redistribution (Schelling) game + proof-of-reserves |
| **Crypto weight** | Light | Heavy (zk-SNARK, SDR) | Medium-heavy (packing+VDF) | Medium (sigs + log challenges) | Light-medium (staked commit-reveal) |
| **Proves copy uniqueness?** | N/A | ✅ SDR→CommR | ✅ RandomX packing | N/A | N/A (proves reserve *size* via density) |
| **Proves persistence?** | periodic audits | ✅ WindowPoSt 24h | continuously (mining) | epoch challenges | per-round sampling (~152 blocks) |
| **Penalty** | DQ + escrow | slashing | lost rewards | lost rewards + shard reassign | stake slashing |

---

# Dimension 4 — Retrieval Mechanics

> Erasure-coded systems shrug off offline nodes (need only a subset); replicated systems need *a* copy — trivial when many exist (Arweave), painful when one must do slow work first (Filecoin); Swarm routes to the chunk's neighborhood by address.

- **Storj:** satellite gives metadata; uplink fetches pieces **P2P in parallel**, keeps the **first 29** (long-tail cancellation), RS-reconstructs, decrypts locally. **Up to 51 of 80 holders can be offline.** Repair loop keeps live count high.
- **Filecoin:** retrieval deal → provider must **unseal** (slow) → serve; **FilCDN/Saturn** caches hot data (L1/L2, IPFS-addressed, verifiable). **No base-layer failover** — depends on having multiple deals.
- **Arweave:** fetch by TX-id via **HTTP gateways** (ar.io/arweave.net), verified against `data_root`. **Strongest churn tolerance** — permanent + massively replicated, any node serves.
- **Walrus:** read **`f+1`** (fast) or **`2f+1`** (async-safe) slivers + Red Stuff decode; most reads via **aggregators/caches** (HTTP/CDN). Tolerates ~2/3 offline; self-heals losses.
- **Swarm:** request routed by **forwarding Kademlia** toward the chunk's neighborhood (O(log N) hops); the first holder **backwards** the chunk along the same path, caching en route. Requestor is anonymous (forwarded ≈ originated). Churn absorbed by neighborhood replication + pull-sync + optional **pinning/recovery** (reactive via PSS).

### Comparison — Dimension 4
| | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **How you fetch** | parallel pieces + RS + decrypt | deal + unseal / FilCDN | HTTP gateway by TX-id | f+1 slivers + decode / aggregator | forwarding-Kademlia to neighbourhood; backwarding |
| **Need full copy?** | No (29 of 80) | Yes (unseal) | Yes (already plaintext) | No (f+1 slivers) | per-chunk (each ≤4 KB by address) |
| **Latency** | fast | slow base / fast cache | web-fast | fast via caches | O(log N) hops; faster on cached routes |
| **Churn tolerance** | lose 51/80 | weak at base | very strong | lose ~2f, self-heals | strong — replicas + pull-sync; pinning/recovery |

---

# Dimension 5 (Cross-Cutting) — Economics & Incentives

> The engine that makes the cryptography binding: who pays, who earns, what's staked, what happens to cheaters.

- **Storj** — token **STORJ**; **recurring** per-GB storage + egress. Node stake = **withheld escrow** (75/50/25/0% over months 1–15, 50% returned at month 16). Cheats → DQ + forfeiture. Low-commitment, cash-flow model.
- **Filecoin** — token **FIL**; **recurring per deal** (+ retrieval market). Node stake = **posted, slashable collateral**; earns block rewards + fees; missed PoSt → slashing. **Filecoin Plus** gives **10×** power for verified data. Capital-intensive; slashing makes the heavy proofs binding.
- **Arweave** — token **AR**; **one-time upfront fee** (~1/21 to miner, rest to **endowment** for ~200 yrs). No collateral/slashing — positive-only rewards enforced by "can't mine what you don't hold." Risk is *endowment solvency* over centuries.
- **Walrus** — token **WAL** (on Sui); **recurring per epoch**. Node stake = **delegated WAL** (shard count = stake share); honest nodes earn, non-responders lose rewards + shards reassigned/slashed. PoS security adapted to storage.
- **Swarm** — token **BZZ** (Gnosis Chain). **Two systems:** **SWAP** (per-connection **bandwidth** balance; cheques cashed in BZZ; relaying+caching is profitable) + **postage stamps** (prepaid **rent** that decays, sets reserve GC priority). The **redistribution game** pays the stamp **pot** to honest staked storers; node stake = **slashable BZZ**. Pay-as-you-go rent; "eventually forgets" unpaid data.

### Comparison — Dimension 5
| | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **Token** | STORJ | FIL | AR | WAL (Sui) | BZZ (Gnosis) |
| **Payment** | recurring GB + egress | recurring per deal | **one-time** | recurring per epoch | postage rent (decaying) + SWAP bandwidth |
| **Stake at risk** | withheld escrow | **slashable collateral** | none | **slashable stake** | **slashable BZZ stake** |
| **Cheater penalty** | DQ + escrow forfeit | slashing | lost rewards | lost rewards + reassign | stake slashing |
| **Economic risk** | per-node churn | capital intensity | endowment solvency | stake centralization | unpaid data GC'd; stamp-batch UX |

---

# Closing Synthesis — At a Glance

| Dimension | **Storj** | **Filecoin** | **Arweave** | **Walrus** | **Swarm** |
|---|---|---|---|---|---|
| **1. Ingestion** | client AES-256-GCM → segment → stripe → pieces | public; chunk → DAG → Piece (`CommP`) → sector | public; chunk → `data_root` → weave | public; blob → 2-D matrix → sliver pairs | optional-encrypt; ≤4 KB chunks → BMT → 128-ary tree |
| **2. Redundancy** | RS 29/35/80/110, ~2.76× | replication, `R`× | ~full replication | 2-D Red Stuff, ~4.5× | neighborhood replication (+ optional RS) |
| **3. Proofs** | audits + reputation | PoRep + PoSt (zk-SNARK) | SPoA→SPoRA→SPoRes (+VDF) | PoA + challenge protocol | redistribution (Schelling) game + proof-of-reserves |
| **4. Retrieval** | parallel pieces, any 29 | deal + unseal; FilCDN | HTTP gateways | f+1 slivers; aggregators | forwarding-Kademlia; path caching |
| **5. Economics** | recurring; escrow | recurring; collateral | **one-time; endowment** | recurring; staked committee | postage + SWAP; slashable BZZ |
| **Philosophy** | private S3 cloud | verifiable cold storage | permanent archive | self-healing storage + DA | permissionless P2P web that "forgets" unpaid data |
| **Default privacy** | **encrypted (zero-knowledge)** | public | public & permanent | public (opt-in Seal) | public (optional client-side) |

**Throughline:** the **redundancy model** is the determinative choice, and these five span all three families — erasure coding (Storj, Walrus), explicit replication (Filecoin, Arweave), and address-proximity neighborhood replication (Swarm). It sets overhead, retrieval (quorum vs. copy vs. routed lookup), churn absorption, and what proofs must prove (unique replicas, fragment possession, or reserve size).

---

# Technical Deep Dive (Implementation-Grade)

> Exact primitives/parameters, verified against specs and source code. Where production code diverges from a whitepaper, **code is authoritative**; version-dependent/uncertain items marked ⚠️.

## DD-1. Storj
- **Key derivation:** root via **Argon2**; sub-keys via **HMAC-SHA512** (trunc 32 B), BIP-32 style. Path keys: `DeriveKey(key, "path:"+comp)`; content key: `DeriveKey(key, "content")` — per-component chaining enables prefix listing / subtree sharing.
- **Path encryption = deterministic-nonce AES-256-GCM (NOT AES-SIV):** `nonce = HMAC-SHA512(derivedKey, "nonce")`.
- **Content cipher:** AES-256-GCM (32 B key, 12 B nonce, per-block `startingNonce + blockNumber`, start derived from segment number). Alt `EncSecretBox` = XSalsa20-Poly1305. Default enc block = 29×256 = **7424 B**.
- **Erasure:** `storj.io/infectious`, RS via **Berlekamp–Welch** (corrects *errors*, not just erasures), **GF(2⁸)**, **Vandermonde** matrix. Stripe = k×256 = **7424 B**. Piece = i-th share of every stripe concatenated → **110 pieces/segment**. `pieceID = HMAC(rootPieceID, nodeID)`.
- **RS tuple `29/35/80/110-256B`**: long-tail tol = 30, offline-without-repair = 45, buffer = 6. ⚠️ **`m` version-dependent** (35 code / 52 SaltLake / 54 glossary).
- **Audit:** random stripe index → one ~256 B share/node → `infectious.Correct()` flags non-reconciling shares; needs ≥29; signed SHA-256 piece hashes + signed order limits; **no Merkle tree** in prod; timeouts → containment.
- **Reputation:** beta trackers with λ forgetting; `AuditLambda=0.999`, `AuditWeight=1.0`, **`AuditDQ=0.96`**, `InitialAlpha=1000` → ~40 consecutive fails to DQ. ⚠️ **0.6 = suspension/offline threshold, not DQ**.
- **Escrow:** 75/50/25/0% over months 1–15; 50% of accumulated returned at month 16.
- **Retrieval/selection:** download fans out **39 order limits** (29+10). `DistinctIP=true`, **/24 IPv4** max 1 piece/subnet, /64 IPv6, `NewNodeFraction≈0.01`, Power-of-Two-Choices; DRPC over TLS/Noise-IK on TCP/QUIC. Repair re-uploads up to o=80.

## DD-2. Filecoin
- **Fr32:** 2 zero bits per 254 data bits → 256-bit units for BLS12-381 scalar field `Fr`. **CommP/PieceCID** = binary Merkle `sha2-256-trunc254-padded` (top 2 bits zeroed). Usable payload = 127/128 of padded power-of-two piece. **CommD/TreeD** = same SHA-254.
- **SDR sealing:** exactly **11 layers** (32/64 GiB); DRG `BASE_DEGREE=6` + bipartite expander `EXP_DEGREE=8` (Feistel) = **14 parents/node**; sequential `Sha254` labeling → memory-hard.
- **Hash split (the easy-to-get-wrong part):** TreeD/CommD/**CommP = SHA-256(trunc254)**; TreeC/**CommC = Poseidon arity 11** (column tree over the 11 labels); TreeR/**CommRLast = Poseidon arity 8** (replica octree); **`CommR = Poseidon₂(CommC, CommRLast)`** — *not* a SHA concat. Poseidon is SNARK-friendly → cheap on-chain verify.
- **Proof flow:** seal binds to on-chain `SealRandomness` (PreCommit) + `InteractiveRandomness` (Commit). **Commit1** = vanilla Merkle inclusion (**176** challenges interactive / **2253** NI-PoRep); **Commit2** = Groth16 zk-SNARK over BLS12-381 (3 group elements); **SnarkPack** aggregates ~8192 proofs in ~8 s (verify ~33 ms).
- **PoSt:** WindowPoSt 24h = 48×30-min deadlines, partition = **2349 sectors** (32 GiB), **10 challenges/sector** into TreeR (Poseidon+Groth16). WinningPoSt: **66 challenges, 1 sector**; Expected Consensus VRF over DRAND, Poisson sortition `λ=(minerPower/totalPower)×5`.
- **Penalties:** Fault Fee (~2.14 days reward/faulty-day), bigger Sector Penalty for undeclared miss, capped Termination Penalty.
- **Retrieval:** unseal (reverse SDR) latency tax; retrieval-market payment channels; FilCDN/Saturn L1 (datacenter ≥10 Gbps) + L2 caches, IPFS-addressed/verifiable.

## DD-3. Arweave
- **Chunking:** `DATA_CHUNK_SIZE=262144` (256 KiB), `MIN_CHUNK_SIZE=32 KiB`, last-chunk rebalancing `ceil(rest/2)`. Merkle all SHA-256 w/ byte-offset notes: leaf=`SHA256(SHA256(chunk)‖SHA256(maxByteRange))`, branch=`SHA256(SHA256(left.id)‖SHA256(right.id)‖SHA256(left.maxByteRange))`; top=`data_root`. `data_path` 96 B/branch, validate by offset. Format-2 tx signs header. `tx_root` Merkles all `data_root`s → the weave; possession proof = chunk + `data_path` + `tx_path`. deepHash/ANS-104 use **SHA-384**; DataItem id = `SHA256(signature)`; bundlers = Irys (ex-Bundlr).
- **Partitions:** **3.6 TB** (`PARTITION_SIZE=3_600_000_000_000`, ≈13.73 M chunks). **Packing per-miner unique:** key(2.6) = `SHA256(chunk_offset‖tx_root‖miner_address)` → RandomX 2 MiB scratchpad, first 256 KiB encrypts the chunk. ⚠️ **pack/unpack roughly symmetric** (not slow-pack/fast-verify).
- **Format evolution:** `spora_2_6` (45 diff, 360 RandomX programs/chunk) → **composite packing** (**2.8**, 32 sub-chunks of 8192 B) → **`replica_2_9`** (current default, `replica_format=1`: 8 MiB entropy → 1024×8192 B slices → 1024 sectors/partition; read-I/O ~5 MiB/s; packing compute −96.9% vs spora_2_6; hardfork block 1602350 ~2025-02-03; composite deprecated ~2025-04-04). 2.7 added Merkle rebasing + VDF retargeting.
- **VDF clock:** SHA-256 hash chain ~1 hash/sec (retargets); step = **25 checkpoints** (`VDF_CHECKPOINT_COUNT_IN_STEP=25`, ~40 ms each); `VDF_SHA_1S=15,000,000` hashes/step. Sequential → only *storing more* wins more.
- **Two recall ranges** from `H0=f(VDF_step, partition, mining_address)`: Range 1 partition-local, Range 2 global (needs Range 1 first → full weave ~doubles odds). Range size era-dependent: 2.6 = 100 MiB = 400 chunks; post-2.8 base = 25 MiB ≈ 100 chunks; attempts/sec/partition ≈ 400 (spora_2_6) / 320 (replica_2_9).
- **SPoRes** (yellowpaper v17): proves *n* unique copies, detection independent of dataset size, superlinear in sampling time.
- **Retrieval/economics:** `GET /{txid}`, `GET /tx/{id}/offset`, self-validating via `data_path→data_root→tx_root`; ar.io gateways add manifests/Range/ArNS/sandboxing (301→Base32(txid) subdomain), GraphQL+REST. Endowment (`ar_pricing`): `N_REPLICATIONS=20`, `PRICE_DECAY_ANNUAL={995,1000}=0.5%/yr` (⚠️ not ~30% historical), `USD_PER_GBY_2019≈$0.000925/GiB-yr`; miner 1/21 immediately, ≥1/20 to endowment; **no endowment withdrawals since genesis**; supply 55M → 66M cap, yearly-halving emission.

## DD-4. Walrus
- ⚠️ **Reed–Solomon, not RaptorQ** (RaptorQ = v1 paper; v2 + code use RS for perfect robustness). `reed-solomon-simd` over **GF(2¹⁶)** (16-bit symbols); `EncodingType::RS2` Display = "RedStuff/Reed-Solomon".
- **Symbol size computed per-blob** (`required_alignment=2 B`, `max_symbol_size=65534`), not fixed.
- **Twin-code 2-D matrix:** with `n_shards`, `f=(n−1)/3`: `source_symbols_primary = n−2f` (symbols in a *secondary* sliver), `source_symbols_secondary = n−f` (symbols in a *primary* sliver); two orthogonal 1-D RS encodings; each node = one sliver pair. ⚠️ **row/column labeling differs whitepaper vs code** (math invariant).
- **Thresholds** (`min_n_correct = n−f = 2f+1`): **`f+1`** = low-threshold dimension (fast read + cheap self-recovery); **`2f+1`** = high-threshold (async-challenge-safe). **Expansion ≈ 4.5×** (vs ~3× plain RS, ~25× full replication).
- **Blob ID (Blake2b-256)**, domain-separated Merkle (`LEAF_PREFIX=[0]`): per sliver pair two Merkle roots `(primary_hash, secondary_hash)` = `SliverPairMetadata`; Merkle-tree all `n_shards` pairs → `merkle_root`; **`BlobId = Blake2b256(encoding_type ‖ unencoded_length_LE ‖ merkle_root)`**. Sliver verified by Merkle inclusion vs committed roots. Metadata ~`n·64+32` B (≈64 KiB @ 1000 shards), may be `(f+1)`-of-`n` coded.
- **On-chain (Sui):** `n_shards=1000` (2f+1 quorum = **667**; Move check `3·weight ≥ 2·n_shards+1`); shards constant, node count varies, stake-weighted via **WAL**, refreshed per epoch.
- **Write → PoA:** register Blob ID on Sui → distribute slivers → collect **2f+1 signed acks** → **Certificate of Availability** → publish on Sui.
- **Async challenge:** epoch-end on-chain randomness PRF picks a random blob subset/node; ~2f+1 pairwise challenges → **Certificate of Storage**; cost `O(challenged blobs)` sub-linear.
- **Reconfiguration:** staged — new writes → epoch e+1 committee, reads continue old, new committee bootstraps via Red Stuff recovery, transfer at 2f+1 ready.
- **Self-healing:** request one intersection symbol per signatory; secondary recovery `f+1`, primary `2f+1`; per-node bandwidth `O(|blob|/n)` vs `O(|blob|)` per node for classic RS.
- **Read:** fast `f+1` / async-safe `2f+1` slivers, re-encode + recompute Blob ID to verify; aggregators reconstruct + serve HTTP; caches = CDN.

## DD-5. Swarm
- **Chunk:** ≤ **4096-byte payload** + **8-byte span**. **BMT hash** = binary Merkle tree (Keccak256) over the payload's **128 × 32-byte segments**; **content-address = Keccak256(span ‖ BMT-root)**.
- **Single-owner chunk:** content = `id ‖ signature ‖ span ‖ payload` (signature signs `id ‖ BMT(span‖payload)`); **address = Keccak256(id ‖ owner-account)**. Basis for **feeds** (id from topic + index).
- **Swarm hash tree:** intermediate chunk = up to **128 references** (4096 ÷ 32); **file address = root chunk address**. **Manifests** = compacted Merkle trie → URL/path routing, key–value maps, virtual hosting.
- **Network stack:** **libp2p** underlay — peer keys **ECDSA secp256r1** (required for alt-clients), TLS/secio, mplex, **protobuf varint-delimited**; `dnsaddr` bootstrap. **Kademlia** overlay: **proximity order (PO)** = shared address-prefix bits; **neighborhood** = peers with PO ≥ storage radius; `NHOOD_PEER_COUNT = 4`. ⚠️ spec constant 4 vs whitepaper ~8 — version/context-dependent.
- **Protocol stream IDs:** handshake `/swarm/handshake/1.0.0` (Syn/SynAck/Ack; exchanges Overlay + NetworkID + light-node flag) · hive `/swarm/hive/1.1.0/peers` · retrieval `/swarm/retrieval/1.4.0` · pushsync `/swarm/pushsync/1.3.0` · pullsync `/swarm/pullsync/1.3.0` (+ `/cursors`) · pricing `/swarm/pricing/1.0.0` · settlement `/swarm/pseudosettle/1.0.0`.
- **Push-sync (upload):** **originator → forwarder(s) → storer**; chunk travels until **PO(peer, chunk) ≥ storage radius**; storer signs a **Receipt**; **shallow receipt** (PO < radius) → re-push; forwarders push to **≤2 closest peers** (multiplex); 5-min chunk/peer skiplist; invalid chunk → blocklist.
- **Pull-sync (consistency):** subscription syncing chunk **ranges per bin** (timestamp-ordered) via `Syn/Ack(cursors)` → `Get` → `Offer` → `Want(bitvector)` → `Delivery`; **interval store** fills gaps; **epoch timestamp** fingerprints reserve (reset → full re-sync); fresh node bootstraps from lowest bin ID.
- **Reserve vs cache:** **reserve** = fixed size (`NODE_RESERVE_DEPTH = 23` → ~2²³ chunks), prioritized by **stamp value**; **cache** = forwarded/popular chunks, GC'd **least-recently-requested** first; evicted reserve chunks fall into cache.
- **Redistribution-game constants:** `PHASE_LENGTH = 38`, `ROUND_LENGTH = 152` (= 4 × phase), `SAMPLE_DEPTH = 4` (16-chunk sample), `MAX_SAMPLE_VALUE ≈ 1.2844 × 10⁷²`, `MINIMUM_STAKE`, `MIN_STAKE_AGE ≈ 1.5 rounds`, `NHOOD_PEER_COUNT = 4`, `PRICE_2X_ROUNDS = 64`.
- **Price oracle (storage rent):** `Price(γ) = Price(prev) · 2^{σ·d}`, supply signal `d = NHOOD_PEER_COUNT − |honest revealers|`, reactivity `σ = 1 / PRICE_2X_ROUNDS` — exponential response to under/over-supply.
- **Randomness:** round seed = **XOR of revealed obfuscation keys** (secure if ≥1 honest party); drives the neighbourhood-selection anchor + reserve-sample salt.
- **Optional erasure coding:** per-level **Reed–Solomon** in the hash tree — e.g. **m = 112 of n = 128** → 16 parity chunks per intermediate node; **RACE mode** bounds retrieval latency; parity chunks are childless.

---

# Comparisons & Tradeoffs (Consolidated)

- **Three redundancy families.** Erasure coding (Storj ~2.76×, Walrus ~4.5×) is storage-efficient but needs a *quorum* of fragments and a repair/heal loop. Replication (Filecoin `R`×, Arweave ~full) is simple with single-copy reads, but cost scales with copies. Swarm's **address-proximity replication** routes placement by hash (small neighborhood replica set + optional RS) — dedup and auto-scaling fall out for free, but no single node "owns" a whole object.
- **Light vs heavy proofs.** Storj's statistical audits and Swarm's staked Schelling sampling are cheap (rely on economics); Filecoin's zk-SNARK PoRep/PoSt are expensive but trustless on-chain. Arweave folds proof into mining; Walrus uses signatures + logarithmic challenges.
- **Privacy posture.** Only Storj is private by default (and hides filenames). The rest are public; Swarm adds network-level privacy (anonymous routing, no metadata leakage) even though chunks are public.
- **Permanence vs duration vs forgetting.** Arweave = pay-once-forever; Storj/Filecoin/Walrus = rent for a chosen term; Swarm = decaying postage that **deliberately forgets** unpaid data.
- **Coordination trust.** Storj leans on a satellite (semi-centralized coordinator); Filecoin/Arweave/Walrus/Swarm coordinate on their own chains (Sui for Walrus, Gnosis for Swarm).

# Advantages & Disadvantages (Per Protocol)

- **Storj** — ➕ zero-knowledge privacy, low overhead, fast reads, cash-flow pricing. ➖ satellite trust, recurring cost, not permanent.
- **Filecoin** — ➕ verifiable at scale, cheap cold storage, strong uniqueness proofs. ➖ capital-intensive, slow unsealing, manual repair.
- **Arweave** — ➕ true permanence, pay-once, web-simple retrieval, best churn tolerance. ➖ centuries-long solvency bet, high real replication cost, public.
- **Walrus** — ➕ BFT durability at ~4.5×, cheap self-healing, native DA. ➖ newest/least proven, security depends on stake decentralization.
- **Swarm** — ➕ permissionless web/dapp hosting, address-routed dedup + auto-scaling, strong network privacy, BitTorrent-like resilience. ➖ no native permanence (forgets unpaid data), dual-incentive UX complexity, smaller per-neighbourhood replica set.

# Real-World Examples & Use Cases

- **Private S3-compatible app storage** → Storj (encrypt-then-erasure-code) → confidential, durable, egress-priced object store.
- **Cheap verifiable archival of large datasets** → Filecoin storage deals + Filecoin Plus → multi-copy cold storage with cryptographic proofs.
- **Permanent records (NFTs, legal docs, web archives)** → Arweave permaweb → pay-once data guaranteed ~200 years, fetched like a URL.
- **Rollup / blockchain data availability** → Walrus on Sui → provably available data at ~4.5× cost with self-healing.
- **Censorship-resistant websites/dapps + private messaging** → Swarm (address-routed chunks, ENS, feeds, PSS) → anonymous, auto-scaling, permissionless web serving paid via postage + bandwidth.

# Questions to Check Understanding

1. Why does "quorum to read" fall out of erasure coding and "one copy to read" out of replication — and what does each pay (overhead vs. read simplicity)?
2. Why must Filecoin prove replica *uniqueness* while Storj doesn't?
3. Why SHA-256 for CommP but Poseidon for the replica/column trees — what matters *inside* a zk-SNARK circuit?
4. Why is Walrus recovery `O(|blob|/n)` and what does the 2-D matrix give that 1-D RS can't?
5. Trace how Arweave's recall block + VDF + per-miner packing make "store the whole weave" profit-maximizing.
6. Rank the five by graceful read-time churn handling; why is Filecoin's base layer weakest?
7. What attack does Walrus's higher `2f+1` threshold defend against, and why is `f+1` safe for self-recovery?
8. How does Swarm's reserve-sample maximum prove a node stores *enough* chunks without challenging individual chunks? Why does a larger stored set yield a *smaller* expected sample maximum?
9. Contrast Swarm's "eventually forget unpaid data" with Arweave's "store forever" — which use case fits each, and what's the failure mode of using the wrong one?
10. If storage costs stopped falling, which protocol's economics is most at risk, and which least?

# Common Misconceptions (Corrected)

1. **"Walrus uses RaptorQ."** ❌ v1 paper only; shipped code uses **Reed–Solomon** (GF(2¹⁶)). ⚠️
2. **"Filecoin proofs are SHA-256 Merkle trees."** ❌ Only CommP/CommD; replica/column trees + CommR use **Poseidon**.
3. **"Arweave relies on ~30%/yr cost decline."** ❌ Protocol assumes **0.5%/yr**; ~30% is just the safety margin. ⚠️
4. **"Storj hides contents but the provider sees filenames."** ❌ **Paths/metadata are encrypted** (deterministic-nonce AES-256-GCM, not AES-SIV).
5. **"Erasure coding ≈ replication."** ❌ Same fault tolerance, far less overhead — but needs a quorum to read.
6. **"More hardware = more Arweave rewards."** ❌ A sequential **VDF** caps attempts/sec; only *more stored data* wins more.
7. **"Storj DQs at 0.6."** ❌ That's the suspension/offline threshold; **audit-DQ = 0.96**. ⚠️
8. **"Swarm uses erasure coding like Storj."** ❌ Base redundancy is **neighborhood replication**; RS is an *optional, file-level* add-on.
9. **"Swarm stores data permanently like Arweave."** ❌ The opposite — it **forgets** content whose postage stamp lapses unless re-funded/pinned.
10. **"Swarm proves storage by challenging each chunk."** ❌ No per-chunk challenges; a **staked Schelling game** + **density-based reserve-size estimation** does it.

# Open Gaps / Next Topics

- ⚠️ Storj repair threshold `m` is version/satellite-dependent (35/52/54) — confirm against the live satellite if load-bearing.
- ⚠️ Walrus row/column labeling differs between whitepaper and code (math invariant).
- ⚠️ Swarm `NHOOD_PEER_COUNT = 4` (2025 spec) vs whitepaper ~8 — confirm against the live Bee codebase; Swarm facts verified against user-provided docs, not source code.
- **Benchmarks** (latency/throughput/$ per TB) are out of scope — a natural follow-up.
- **Walrus & Swarm** parameters are recent and may evolve.
- A sixth peer (Sia, Crust) or the IPFS pinning ecosystem could extend the comparison.

---

## Sources
- **Storj:** [v3 whitepaper](https://static.storj.io/storjv3.pdf), [encryption docs](https://www.storj.io/object-storage/compliance/gdpr), [file redundancy](https://docs.storj.io/dcs/concepts/file-redundancy), [file repair](https://storj.dev/learn/concepts/file-repair), [glossary](https://storj.dev/learn/concepts/definitions), [storj/common encryption](https://github.com/storj/common/tree/main/encryption), [storj/infectious](https://github.com/storj/infectious), [audit-service wiki](https://github.com/storj/storj/wiki/audit-service), [held amount FAQ](https://storj.dev/node/faq/held-back-amount)
- **Filecoin:** [Spec — Piece/CommP](https://spec.filecoin.io/systems/filecoin_files/piece/), [Spec — SDR & notation](https://spec.filecoin.io/algorithms/sdr/notation/), [Spec — PoSt](https://spec.filecoin.io/algorithms/pos/post/), [Spec — Expected Consensus](https://spec.filecoin.io/algorithms/expected_consensus/), [Docs — Proofs](https://docs.filecoin.io/basics/the-blockchain/proofs), [rust-fil-proofs constants.rs](https://github.com/filecoin-project/rust-fil-proofs/blob/master/filecoin-proofs/src/constants.rs), [Poseidon in Filecoin](https://hackmd.io/@7dpNYqjKQGeYC7wMlPxHtQ/ByGEnfrh8), [SnarkPack](https://research.protocol.ai/blog/2021/snarkpack-how-to-aggregate-snarks-efficiently/), [retrieval market](https://docs.filecoin.io/basics/what-is-filecoin/retrieval-market), [Saturn/FilCDN](https://docs.filecoin.io/basics/how-retrieval-works/saturn)
- **Arweave:** [2.6 spec](https://2-6-spec.arweave.net/), [HTTP API](https://docs.arweave.org/developers/arweave-node-server/http-api), [ArweaveTeam/arweave](https://github.com/ArweaveTeam/arweave), [ANS-104](https://github.com/ArweaveTeam/arweave-standards/blob/master/ans/ANS-104.md), [yellow paper](https://www.arweave.org/yellow-paper.pdf), [SPoRA announcement](https://arweave.medium.com/the-arweave-network-is-now-running-succinct-random-proofs-of-access-spora-e2732cbcbb46), [ar.io gateways](https://ar.io)
- **Walrus:** [Whitepaper v2 (arXiv 2505.05370)](https://arxiv.org/html/2505.05370v2), [Red Stuff explainer](https://www.walrus.xyz/blog/how-walrus-red-stuff-encoding-works), [Proof of Availability](https://www.walrus.xyz/blog/how-walrus-proof-of-availability-works), [MystenLabs/walrus — walrus-core](https://github.com/MystenLabs/walrus/tree/main/crates/walrus-core/src), [Walrus Docs](https://docs.wal.app)
- **Swarm** (user-provided primary sources): Swarm Whitepaper v1.0 (June 2021); Swarm Protocol Specification draft (Sept 2025); *The Book of Swarm* (§5.1 erasure coding). Public: [ethswarm.org](https://www.ethswarm.org), [The Book of Swarm PDF](https://www.ethswarm.org/The-Book-of-Swarm.pdf), [Bee client](https://github.com/ethersphere/bee).

> **Verification note:** figures cross-checked against protocol specs + source code; where production code diverges from a whitepaper, code is authoritative. Version-dependent/uncertain items are marked ⚠️ (e.g. Storj `m`, Arweave recall-range size, Walrus row/column labeling, Swarm `NHOOD_PEER_COUNT`). Swarm facts come from the three user-provided documents; no contradiction was found across them in this pass, though the 2021 whitepaper and 2025 spec differ on some constants.
