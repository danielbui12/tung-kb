---
title: "Storj: A Peer-to-Peer Cloud Storage Network (v2.0)"
type: source
tags: [decentralized-storage, peer-to-peer, proof-of-retrievability, erasure-coding, kademlia, blockchain, storage-marketplace]
created: 2026-06-19
author: Shawn Wilkinson, Tome Boshevski, Josh Brandoff, James Prestwich, Gordon Hall, Patrick Gerbes, Philip Hutchins, Chris Pollard (contributions from Vitalik Buterin)
source_url: "local: raw/storjv2.md"
source_type: paper
ingested: 2026-06-19
---
# Storj: A Peer-to-Peer Cloud Storage Network (v2.0)

**One-line:** A decentralized cloud storage network where client-side-encrypted file shards are stored on untrusted peers ("farmers") whose data retention is enforced through cryptographic challenge-response audits paid via micropayments.

## Summary
Storj v2 (December 2016) describes a peer-to-peer object storage protocol that replaces trusted datacenter providers with an open market of independent storage nodes. Files are encrypted client-side (reference uses AES256-CTR), split into fixed-size shards, and distributed across the network. Because the data owner alone knows the sharding scheme and shard locations, locating a target file without compromising the owner becomes combinatorially hard as the network grows, and client-side encryption protects shard contents even if all shards are gathered.

The network is built atop Kademlia, a distributed hash table, but shards themselves are not stored in the DHT — Kademlia provides routing and message transport. Storj borrows from S/Kademlia by requiring ECDSA-signed messages: each Node ID is `ripemd160(sha256(k_pub))`, which is simultaneously a valid Bitcoin address. This enforces long-term identity and imposes a proof-of-work cost on eclipse and identity-hijacking ("Spartacus") attacks.

Data durability is enforced through proofs of retrievability called "audits" or "heartbeats." The reference scheme uses Merkle trees: the owner generates random challenge salts, builds pre-leaves `H(s + d)` and leaves `H(H(s + d))`, stores the Merkle root and tree depth, and hands the leaves to the farmer. Periodically the owner issues a salt; the farmer must return a compact Merkle proof of exactly `log2(|l|)+1` hashes. Partial audits trade probabilistic confidence for far lower compute and I/O by hashing only byte subsets, and can be mixed with full audits.

Storage relationships are governed by signed contracts negotiated through an OFFER message loop, with parties discovered via Quasar, a Bloom-filter-based topic pub/sub layered on Kademlia. The protocol adds message types for contracting (OFFER, CONSIGN, MIRROR, RETRIEVE, AUDIT), pub/sub (SUBSCRIBE, UPDATE, PUBLISH), and NAT traversal (PROBE, FIND_TUNNEL, OPEN_TUNNEL via the Diglet reverse-tunnel library). Payment is agnostic but the reference assumes Storjcoin micropayment channels that pair payment directly to each audit. Redundancy is the data owner's responsibility, supported by simple mirroring and (planned) k-of-m Reed-Solomon erasure coding. Local on-disk storage uses KFS, an abstraction over 256 size-limited LevelDB "S-Buckets" that bounds compaction cost.

Because the data owner's obligations (contracting, auditing, paying, key and state management) demand high uptime, Storj introduces Bridge — an SPV-wallet-analogous thin-client server that takes over contract negotiation, audits, payments, and file state while the client keeps encryption and keys. Bridge stores only metadata, exposes a RESTful object-store API, and can act as an authorization, permissioning, and intelligent-contracting layer; third parties can run Bridge-as-a-service. The paper closes with future research (federated Bridges, data portability, distributed reputation systems), a catalog of attacks (Sybil, Google/nation-state, Honest Geppetto, Eclipse, Hostage Bytes, Cheating Owner, Faithless Farmer), and probability calculations for erasure-coding failure, eclipse difficulty, file-location ("beach size"), and partial-audit false positives.

## Key takeaways
- Trust is removed by combining client-side encryption (confidentiality), proof-of-retrievability audits (integrity/availability), and direct micropayments (incentive).
- Node IDs double as Bitcoin addresses via ECDSA key-hashing, making message signing the basis of identity and a proof-of-work deterrent against Sybil/Eclipse/Spartacus attacks.
- Merkle-tree audits give exact (no false positive/negative) verification; partial audits give cheaper probabilistic verification, with consecutive audits driving false-positive odds toward zero.
- Redundancy is the data owner's job: simple mirroring or k-of-m erasure coding, where availability depends on the least-available `m+1` nodes rather than the single least-available shard.
- The Bridge thin-client model is the key usability bet — it delegates operational burden (not data or keys) to a high-uptime server, mirroring Bitcoin SPV wallets.
- Several core problems are explicitly unsolved: distributed reputation, publicly verifiable proof of storage, and the "cheating owner" who withholds payment after a valid audit.

## Concepts & entities
Wikilinks: [[storj]], [[decentralized-storage]], [[proof-of-storage]], [[proof-of-retrievability]], [[erasure-coding]], [[reed-solomon]], [[distributed-hash-table]], [[kademlia]], [[merkle-tree]], [[audits]], [[redundancy]], [[data-durability]], [[reputation-system]], [[content-addressing]], [[blockchain]], [[micropayment-channels]], [[sybil-attack]], [[eclipse-attack]], [[client-side-encryption]], [[sharding]], [[publish-subscribe]], [[bloom-filter]], [[nat-traversal]], [[thin-client]], [[storjcoin]], [[bridge-server]], [[quasar]], [[kfs]], [[farmer]].

## Notable claims / data
- File location security scales with the square of network size; locating a 10-shard file by 50 random draws from a 100-shard network has probability ~5.9e-4 (hypergeometric "beach size").
- Storj Merkle proofs are exactly `log2(|l|)+1` hashes regardless of tree size.
- Partial-audit false positives are vanishingly small: a 512-byte audit of an 8192-byte shard with 512 adversarially modified bytes yields false-positive probability ~1.5e-15.
- k-of-n erasure coding: an 18-of-? scheme (n=18, k=6) at per-shard uptime p=0.9 gives failure probability ~5.3e-10; at p=0.98, ~6.4e-19.
- Eclipse difficulty: against a 500-node network, 100 hashes succeed with probability ~8.8e-1, but 900 hashes against a 900-node network only ~8.0e-2 — difficulty grows with node count.
- KFS defaults: S = 32 GiB per S-Bucket, B = 256 buckets → 8 TiB capacity; chunk size C = 128 KiB; shards sorted by XOR of first `g = ceil(log2 B)` bits of Node ID and shard hash.
- A Sybil attacker controlling 50% of the network isolates only ~12.5% of honest nodes (messages sent to 3 neighbors).
- Audit payments are extremely small, often below $0.000001 per audit, motivating Storjcoin's high payment granularity and large supply.
- A reference CLI (github.com/storj/core-cli) reliably streams 1080p video from the network; first client is JavaScript with C/Python/Java in progress.

## Questions raised
- How can a distributed reputation system reliably capture bandwidth, latency, and availability without prohibitive consensus latency? (Bitswap, Eigentrust++, TrustDavis, and identity-maintenance-cost schemes all reviewed, none adopted.)
- Is there any publicly verifiable proof of storage, or any way to independently verify a privately verifiable audit was issued/answered — needed to solve the "cheating owner" attack?
- How will federated Bridges sync file metadata/pointers and permissions across servers without reintroducing centralization or trust concentration?
- Can secure person-to-person file sharing (proxy re-encryption, Keybase-anchored identity, smart-contract permissioning) be made practical, given that access to disseminated data can never truly be revoked?
- How does the Bridge model's reliance on trusted servers compare, in practice, to the fully decentralized ideal the paper opens with — and how much trust delegation is acceptable?
