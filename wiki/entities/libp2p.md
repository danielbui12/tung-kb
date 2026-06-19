---
title: libp2p
type: entity
tags: [peer-to-peer, networking, modular-stack, dht]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, swarm-protocol-spec]
---
# libp2p

**What it is:** A modular peer-to-peer networking stack providing transport, security, multiplexing, discovery, and pub/sub for distributed applications.

## Notes
libp2p, originally extracted from [[ipfs]] by [[protocol-labs]], composes networking concerns into swappable modules: multiaddr addressing, dialing/listening over live full-duplex connections, secure channels (TLS/secio/Noise), stream multiplexing (mplex), a [[kademlia]] [[distributed-hash-table]] for peer/content routing, GossipSub pub/sub, and PeerID derivation from node keys.

[[filecoin]] uses libp2p for ChainSync (GossipSub/BlockSync/Bitswap/GraphSync) and peer discovery. [[swarm]] uses libp2p as its *underlay* transport beneath its Kademlia overlay, fixing the node key type to ECDSA secp256R1 (serving both message signing and peer-id derivation), using varint-delimited Protobuf serialization and `dnsaddr` bootstrap, and naming every stream `/swarm/{Protocol}/{Version}/{Stream}`.

## Sources
- [[filecoin-spec]] — libp2p in Filecoin's ChainSync and routing
- [[swarm-protocol-spec]] — libp2p as Swarm's underlay, key type, serialization
