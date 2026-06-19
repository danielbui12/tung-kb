---
title: Content Addressing
type: concept
tags: [content-addressing, cryptographic-hash, decentralized-web, deduplication]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [content-addressing-tutorial, anatomy-of-a-cid-tutorial, merkle-dags-tutorial, object-storage-domain-knowledge, book-of-swarm, filecoin-spec]
---
# Content Addressing

**Definition:** Naming and retrieving data by an identifier derived from the data's own bytes (a cryptographic hash) rather than from where it is stored.

## How it works
A piece of data is run through a [[cryptographic-hash]] to produce a fixed-size, deterministic digest, which becomes its address — typically packaged as a [[content-identifier]] (CID) using [[multiformats]]. Because the same bytes always hash to the same value, identical content shared by different peers yields identical addresses, and any change produces a wholly different one. This makes addresses *self-certifying*: a requester computes the hash of received bytes and checks it against the address it asked for, verifying integrity without trusting the sender. Data is then requested from the whole network by hash, so any peer holding matching bytes can serve it. Hashes also act as links, allowing content-addressed nodes to reference one another and form [[merkle-dag]] structures.

## Why it matters / trade-offs
Content addressing is the foundation of the decentralized web and [[decentralized-storage-networks]]. It enables [[deduplication]] (identical data stored once), location-independent retrieval (availability survives any single host going offline), and verifiable links. The trade-off is immutability: an address binds to exact bytes, so updates require a new address plus a separate mutable-naming layer (e.g. IPNS). Contrast with [[location-addressing]], where the address has no relationship to the contents.

## Related
[[location-addressing]], [[cryptographic-hash]], [[content-identifier]], [[multiformats]], [[merkle-dag]], [[deduplication]], [[content-addressed-data-structures]], [[ipfs]], [[git]]

## Sources
- [[content-addressing-tutorial]] — the location-vs-content model and trust/verification reframing
- [[anatomy-of-a-cid-tutorial]] — how CIDs encode content-derived addresses
- [[merkle-dags-tutorial]] — hashes as structural links between nodes
- [[object-storage-domain-knowledge]] — addressing in storage systems
- [[book-of-swarm]] — content addressing in Swarm
- [[filecoin-spec]] — content addressing in Filecoin state
