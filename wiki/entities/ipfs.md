---
title: IPFS
type: entity
tags: [content-addressing, peer-to-peer, decentralized-web, distributed-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [content-addressing-tutorial, anatomy-of-a-cid-tutorial, merkle-dags-tutorial, filecoin-spec, object-storage-domain-knowledge]
---
# IPFS

**What it is:** The InterPlanetary File System — a content-addressing and peer-to-peer transport substrate for the decentralized web.

## Notes
IPFS, from [[protocol-labs]], names data by the hash of its content rather than its location, using [[content-identifier]]s (CIDs). Files become UnixFS [[merkle-dag]]s; CIDs are built from [[multiformats]] prefixes; data is structured via [[ipld]]; blocks move over Bitswap atop [[libp2p]] (including a [[kademlia]] [[distributed-hash-table]] for provider routing). This makes retrieval verifiable and trustless — any peer holding matching bytes can serve them.

Critically, IPFS provides *no native persistence guarantee*: it is an addressing/transport layer, not a storage incentive system. Content stays available only while some node pins or caches it. [[filecoin]] is the paid persistence layer built underneath it, enforcing storage deals over time. In the [[decentralized-storage-networks]] comparison IPFS therefore sits beneath the others as shared plumbing rather than as a competing durable store.

Compare [[swarm]], which embeds similar content-addressing but adds [[postage-stamps]] incentives and stores chunk content (not seeder pointers) at the closest nodes. CIDs originated here but now serve [[git]], Ethereum, and Filecoin alike.

## Sources
- [[content-addressing-tutorial]] — content vs location addressing, CID basics
- [[anatomy-of-a-cid-tutorial]] — CID structure and multiformats
- [[merkle-dags-tutorial]] — UnixFS Merkle-DAG structure and dedup
- [[filecoin-spec]] — IPFS as the persistence substrate under Filecoin
- [[object-storage-domain-knowledge]] — IPFS as no-persistence substrate
