---
title: Storj
type: entity
tags: [decentralized-storage, object-storage, erasure-coding, s3-compatible]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [storj-v3, storj-v2, object-storage-domain-knowledge, erasure-coding-algorithms-comparison]
---
# Storj

**What it is:** A decentralized, S3-compatible cloud object store that erasure-codes client-side-encrypted data across globally distributed untrusted nodes.

## Notes
Storj is one of the [[decentralized-storage-networks]], built by [[storj-labs]]. It applies [[client-side-encryption]] (AES-256-GCM) and [[reed-solomon]] [[erasure-coding]] (e.g. 29/35/80/110, ~2.76x overhead, ~11 nines durability) so durability is decoupled from expansion factor.

**v2** (2016) was a [[kademlia]]/[[distributed-hash-table]]-routed marketplace of "farmers," using Merkle-tree proof-of-retrievability heartbeats, signed contracts, and Storjcoin micropayments, with a Bridge thin client offloading operations.

**v3** (Tardigrade, 2018) abandoned blockchain on the hot path for coordination avoidance at exabyte scale. Three peer classes: [[uplink]] clients (encryption + erasure coding), [[storage-node]]s (store pieces, paid for at-rest + egress), and trusted-but-blind [[satellite]]s (metadata, [[audits]], repair, reputation, payment). Audits use random-stripe retrieval + Berlekamp-Welch correction rather than pre-generated [[proof-of-storage]] challenges.

Unlike [[filecoin]]'s heavy trustless zk-SNARK proofs or [[arweave]]'s permanence, Storj favors lightweight economic audits and is the only one of the five DSNs private by default (encrypting paths and metadata too). Token: STORJ (ERC-20 on [[ethereum]]). Compare [[swarm]] (Kademlia neighborhood replication) and [[walrus]] (2-D erasure coding).

Notably, Storj chose Reed-Solomon over [[fountain-codes]] despite its churning, unpredictable peer set superficially fitting a rateless code better — evidence that MDS optimality and mature tooling win once a storage network's "channel" is provisioned enough to fix `n` in advance.

## Sources
- [[storj-v3]] — v3 framework, Satellites/Uplinks/nodes, audits, RS parameters
- [[storj-v2]] — v2 Kademlia/contract/micropayment architecture, Merkle audits
- [[object-storage-domain-knowledge]] — comparative parameters and durability figures
- [[erasure-coding-algorithms-comparison]] — RS-over-fountain-codes choice for P2P/churning storage
