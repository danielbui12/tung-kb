---
title: Decentralized Object Storage — Domain Knowledge
type: source
tags: [decentralized-storage, object-storage, erasure-coding, data-durability, proof-of-storage, blockchain, content-addressing]
created: 2026-06-19
author: Unknown
source_url: "local: raw/object_storage_domain_knowledge.md"
source_type: note
ingested: 2026-06-19
---
# Decentralized Object Storage — Domain Knowledge

**One-line:** A structured comparison of five decentralized object-storage networks (Storj, Filecoin, Arweave, Walrus, Swarm) across ingestion, redundancy, proofs, retrieval, and economics, organized around the redundancy model as the master design variable.

## Summary
The document analyzes how five decentralized storage networks (DSNs) solve the same problem — storing a file durably on a network of untrusted nodes — using cryptography plus token incentives instead of a trusted operator. It frames the comparison along four core dimensions (data ingestion/splitting, distribution/redundancy, cryptographic proofs, retrieval) plus a cross-cutting economics dimension, then provides an implementation-grade deep dive with exact primitives and parameters verified against specs and source code.

The central thesis is that the **redundancy model is the master variable**. The five span three families: erasure coding (Storj 1-D Reed–Solomon ~2.76× overhead, Walrus 2-D "Red Stuff" ~4.5×), explicit replication (Filecoin sealed sectors, Arweave blockweave), and address-proximity neighborhood replication (Swarm Kademlia). This choice cascades into storage overhead, whether reads need a quorum vs a single copy vs a routed lookup, how node churn is absorbed, and — crucially — what the proof system must prove: unique replicas (Filecoin SDR sealing, Arweave RandomX packing), fragment possession (Storj sampling, Walrus challenges), or sufficient reserve size (Swarm density sampling).

Content addressing is the shared backbone for four of the five: naming data by the hash of its content (CID, data_root, Blob ID, BMT chunk address) makes integrity self-certifying with no trusted server. Storj is the exception, addressing by encrypted path because content is encrypted client-side (AES-256-GCM, zero-knowledge — even paths and metadata are hidden from the satellite). The core cryptographic puzzle across all DSNs is proving storage continuously over time without re-reading the full data, and economics (slashing, escrow, endowment, decaying postage) is what makes the cryptography binding.

The networks occupy a spectrum from private efficient cloud (Storj) through verifiable cold storage (Filecoin), permanent public archive (Arweave), self-healing storage plus data-availability (Walrus, with on-chain Point of Availability on Sui), to permissionless P2P web platform (Swarm). IPFS sits beneath the comparison as an addressing/transport substrate with no persistence — Filecoin is the paid persistence layer under it. A key contrast is permanence vs forgetting: Arweave keeps data forever via a pay-once endowment, while Swarm deliberately garbage-collects unpaid data whose postage stamp lapses.

The deep-dive section corrects several common misconceptions, notably that Walrus ships Reed–Solomon (not RaptorQ, which was v1-paper only), that Filecoin uses Poseidon (not SHA-256) for replica/column trees inside zk-SNARKs, and that Arweave's economics assume only ~0.5%/yr cost decline rather than the ~30% historical rate. It flags version-dependent values as learning gaps (Storj repair threshold m, Walrus row/column labeling, Swarm NHOOD_PEER_COUNT).

## Key takeaways
- The redundancy model determines everything downstream: overhead, retrieval pattern, churn handling, and proof requirements form a causal chain (Dim 2 → Dim 3 → Dim 4).
- Three redundancy families: erasure coding (efficient, needs quorum read + repair loop), replication (simple single-copy reads, cost scales with copies), and address-proximity replication (dedup + auto-scaling fall out of hash-based placement).
- Proof systems map to redundancy: replication ⇒ prove copies are unique; erasure coding ⇒ prove fragment possession; neighborhood replication ⇒ prove reserve size.
- Light/economic proofs (Storj audits, Swarm staked Schelling game) vs heavy trustless proofs (Filecoin zk-SNARK PoRep/PoSt); Arweave fuses proof into mining; Walrus uses signatures + logarithmic challenges.
- Only Storj is private by default (encrypts contents AND filenames/metadata); the rest are public, though Swarm adds network-level privacy via anonymous routing.
- Payment models diverge sharply: pay-once endowment (Arweave) vs recurring rent (Storj/Filecoin/Walrus) vs decaying postage that deliberately forgets unpaid data (Swarm).
- Code is authoritative over whitepapers where they diverge; several parameters are version-dependent and flagged with caution.

## Concepts & entities
Wikilinks: [[decentralized-storage]], [[object-storage]], [[block-storage]], [[file-storage]], [[erasure-coding]], [[data-durability]], [[content-addressing]], [[reed-solomon]], [[proof-of-storage]], [[proof-of-replication]], [[proof-of-spacetime]], [[zk-snark]], [[poseidon-hash]], [[merkle-tree]], [[data-availability]], [[kademlia]], [[schelling-game]], [[slashing]], [[storj]], [[filecoin]], [[arweave]], [[walrus]], [[swarm]], [[ipfs]], [[red-stuff-encoding]], [[postage-stamps]], [[sui]], [[gnosis-chain]].

## Notable claims / data
- Storj: Reed–Solomon 29/35/80/110, ~2.76× overhead, ~11 nines durability, survives losing 51 of 80 pieces; AES-256-GCM with HMAC-SHA512 key derivation; audit-DQ threshold = 0.96 (NOT 0.6, which is suspension).
- Filecoin: SDR sealing in exactly 11 layers; 32/64 GiB sectors; WindowPoSt every 24h (48 × 30-min deadlines); CommP/CommD use SHA-256(trunc254), but TreeC/TreeR/CommR use Poseidon (arity 11 and 8); Commit2 = Groth16 over BLS12-381; Filecoin Plus gives 10× power for verified data.
- Arweave: ≤256 KiB chunks; 3.6 TB partitions; per-miner unique RandomX packing; VDF ~1 hash/sec rate-limits hashing; endowment assumes 0.5%/yr cost decline; miner gets 1/21 immediately, rest to endowment; current packing format replica_2_9 (hardfork ~2025-02-03).
- Walrus: n = 3f+1 BFT committee on Sui (n_shards=1000, 2f+1 quorum=667); Reed–Solomon over GF(2^16); ~4.5× expansion; recovery bandwidth O(|blob|/n) vs O(|blob|) for classic RS; BlobId = Blake2b256(encoding_type ‖ length ‖ merkle_root).
- Swarm: ≤4096-byte chunks + 8-byte span; BMT (Keccak256) addressing; NHOOD_PEER_COUNT=4 (spec) vs ~8 (whitepaper); NODE_RESERVE_DEPTH=23 (~2^23 chunks); redistribution game ROUND_LENGTH=152 blocks; libp2p underlay with ECDSA secp256r1; optional file-level RS (e.g. m=112 of 128).

## Questions raised
- Why does "quorum to read" fall out of erasure coding and "one copy to read" out of replication, and what does each pay (overhead vs read simplicity)?
- Why must Filecoin prove replica uniqueness while Storj need not?
- What property of Poseidon matters inside a zk-SNARK circuit that SHA-256 lacks?
- Why is Walrus recovery O(|blob|/n) rather than O(|blob|), and what does the 2-D matrix give that 1-D RS can't?
- How does Swarm's reserve-sample maximum prove a node stores enough chunks without challenging individual chunks (why does a larger stored set yield a smaller expected sample maximum)?
- If storage costs stopped falling, which protocol's economics is most at risk?
- Out of scope here: real-world benchmark latency/throughput/$ per TB numbers, and a possible sixth peer (Sia, Crust) or the IPFS pinning ecosystem.
