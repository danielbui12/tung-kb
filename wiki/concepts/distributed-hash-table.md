---
title: Distributed Hash Table
type: concept
tags: [distributed-hash-table, kademlia, peer-to-peer, content-addressing, overlay-network]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, storj-v2, object-storage-domain-knowledge]
---
# Distributed Hash Table

**Definition:** A decentralised key-to-value store spread over many peers, where each key is deterministically assigned to the node(s) whose address is closest to it, enabling lookup without any central index.

## How it works
Keys and node identifiers share one address space (typically 256 bits). A routing overlay such as [[kademlia]] lets any node find the peers responsible for a key in O(log n) hops. What gets *stored* at those responsible nodes differs sharply by system. [[ipfs]]'s DHT stores **provider records** — pointers saying which peers hold a given CID — so a lookup returns an address list and the data is fetched separately. [[storj]] uses the DHT purely for routing and peer discovery; shards live on contracted nodes, never in the DHT. [[swarm]]'s DISC (Distributed Immutable Store for Chunks) goes further: it stores the [[chunk]] **content itself** at the nodes closest to the chunk address, so a successful lookup returns the data directly.

## Why it matters / trade-offs
A DHT removes the central directory that would otherwise be a bottleneck and censorship point, and its deterministic placement means any node can locate data from the key alone. Storing pointers (IPFS) keeps the DHT small but adds a fetch hop and depends on providers staying online; storing content (Swarm) gives single-hop retrieval and deduplication but binds storage location to the hash, requiring incentives so closest nodes actually retain data. Routing tables must be maintained against churn.

## Related
[[kademlia]] [[proximity-order]] [[chunk]] [[content-addressing]] [[swarm]] [[ipfs]] [[storj]]

## Sources
- [[book-of-swarm]] — DISC storing chunk content at closest nodes vs classic seeder-pointer DHTs
- [[storj-v2]] — Kademlia DHT for routing/transport, shards stored off-DHT
- [[object-storage-domain-knowledge]] — DHT placement across DSN families
