---
title: "Swarm: Storage and Communication Infrastructure for a Self-Sovereign Digital Society"
type: source
tags: [decentralized-storage, peer-to-peer, ethereum, content-addressing, incentivization, kademlia, web3]
created: 2026-06-19
author: The Swarm Team
source_url: "local: raw/swarm/swarm-whitepaper.md"
source_type: paper
ingested: 2026-06-19
---
# Swarm: Storage and Communication Infrastructure for a Self-Sovereign Digital Society

**One-line:** Swarm is a peer-to-peer network providing decentralised storage and communication, made economically self-sustaining by Ethereum smart contracts and the BZZ token.

## Summary
Swarm (whitepaper v1.0, June 2021) is a peer-to-peer network of nodes that collectively provide a decentralised storage and communication service. Its mission is to provide scalable base-layer infrastructure for the decentralised internet, extending the blockchain with peer-to-peer storage and communication to realise a "world computer" that serves as a deployment environment for decentralised applications. The system is economically self-sustaining via a built-in incentive system enforced through smart contracts on Ethereum and powered by the BZZ token. Architecturally, Swarm is modular: an underlay p2p network (libp2p), an overlay network with immutable storage, high-level data access APIs, and an application layer.

The core storage model is the DISC (Distributed Immutable Store of Chunks). Nodes build Kademlia connectivity: each node has a Swarm address (distinct from its network address), and proximity is measured by shared address-prefix bits. Nodes form a fully connected neighbourhood with their nearest peers plus balanced peers across each proximity class, yielding a topology where each forwarding hop moves a message at least one step closer to its destination, with hop count logarithmic in network size. The canonical storage unit is a chunk (at most 4 KB) whose address shares the node address space, so chunks are stored by nodes close to the chunk address. Chunks can be padded and encrypted, making them indistinguishable from random data and shielding operators from liability since they don't choose what they store. Insertion uses the push-sync protocol; retrieval routes a request toward the relevant neighbourhood; pull-sync continuously replicates chunks across neighbours for redundancy.

Incentivisation operates on two axes. Bandwidth is handled by the Swarm Accounting Protocol (SWAP): peers track relative bandwidth consumption and, beyond a threshold, the debtor settles via cheques cashable in BZZ on-chain. This keeps Swarm free for light/patient users while letting heavy users pay for speed, and motivates nodes to relay and opportunistically cache (subsequent requests for a cached chunk become pure profit). Storage persistence is handled by postage stamps: a blockchain contract sells postage batches for BZZ, entitling the owner to issue stamps that attach to chunks as a fiduciary signal of storage value. Each node's local storage splits into a fixed-size reserve (neighbourhood chunks prioritised by stamp value, which decays over time like rent) and a cache (pruned by least-recently-requested). This garbage-collection strategy implements eventual forgetting plus automatic scaling of popular content.

Two fundamental chunk types exist: content-addressed chunks, whose address is the BMT (Binary Merkle Tree) hash of span and payload, enabling integrity verification; and single-owner chunks, whose address derives from the owner's account and an identifier, with integrity warranted by the owner's signature, giving each identity a portion of address space to freely assign content.

Above chunks, the Swarm API exposes higher-level constructs that mirror the web. Files larger than 4 KB are split and represented as a balanced Swarm hash-tree of chunks (root hash = file checksum, enabling efficient random/range access). Collections use manifests, compacted Merkle tries that model directory trees, key-value stores, and routing tables, supporting URL-based addressing and logarithmic lookup, plus compact proofs of segment membership. Feeds (single-owner chunks indexed by topic+index) simulate mutable resources; ENS resolves human-readable domains (e.g. swarm.eth) to references. PSS (Postal Service on Swarm) provides anonymous, asynchronous node-to-node messaging via encrypted "Trojan" chunks delivered through push-sync. Pinning plus reactive and proactive recovery protocols let operators preserve and re-seed content the DISC would otherwise forget. The paper describes functionality shipped in the Bee 1.0 mainnet launch.

## Key takeaways
- Swarm = decentralised storage + communication, incentivised by Ethereum smart contracts and the BZZ token.
- DISC stores fixed-size 4 KB chunks; chunk addresses share the node address space, so storage location is determined by proximity (Kademlia).
- Routing is logarithmic-hop forwarding Kademlia; forwarding ambiguity gives originator privacy and enables permissionless publishing and private browsing.
- SWAP settles bandwidth debt peer-to-peer via BZZ cheques; postage stamps prepay storage rent and prioritise reserve retention.
- Local storage = reserve (stamp-prioritised neighbourhood chunks) + cache (LRU); together they yield eventual forgetting and auto-scaling of popular content.
- Two chunk types: content-addressed (BMT hash, integrity by hash) and single-owner (address from owner+identifier, integrity by signature).
- Higher-level constructs: hash-tree files, manifest tries for collections, feeds for mutability, ENS for naming, PSS for anonymous messaging.

## Concepts & entities
Wikilinks: [[swarm]], [[decentralized-storage]], [[disc]], [[chunk]], [[content-addressing]], [[kademlia]], [[postage-stamps]], [[swap-protocol]], [[incentivization]], [[single-owner-chunk]], [[bmt-hash]], [[push-sync]], [[pull-sync]], [[swarm-manifest]], [[swarm-feeds]], [[pss]], [[bzz-token]], [[ethereum]], [[ens]], [[bee-client]], [[libp2p]], [[peer-to-peer]].

## Notable claims / data
- Chunk size: at most 4 kilobytes; intermediate hash-tree chunks pack 128 hashes each.
- Routing hop count is logarithmic in the total number of nodes (guaranteed reachability even in very large networks).
- Neighbourhood is defined by proximity order d with a minimum of 8 nodes; the node also connects to at least 8 balanced peers in each shallower proximity bucket.
- Postage stamp value decays over time as if rent were deducted; when insufficient, the chunk is evicted from reserve to cache.
- Cache eviction = least-recently-requested, treating request recency as a popularity predictor.
- Whitepaper v1.0, June 13, 2021; functionality shipped in Bee 1.0 mainnet launch (github.com/ethersphere/bee).
- Reactive recovery resembles BitTorrent/IPFS seeding; recovery requests are sent via PSS to pinners.

## Questions raised
- How are BZZ cheque settlement and postage-stamp economics parameterised (thresholds, decay rates, batch pricing) in practice?
- What are the concrete redundancy guarantees of pull-sync (replication factor, neighbourhood size minimums) under churn?
- How does "eventual forgetting" interact with legal data-permanence or right-to-erasure requirements?
- What prevents abuse of opportunistic caching or postage stamps (e.g. spam, freeloading, stamp forgery)?
- How does the security/privacy model (Trojan chunks, anonymous forwarding) hold against a global passive adversary or Sybil attacks?
