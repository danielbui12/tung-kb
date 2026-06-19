---
title: Bee Client
type: entity
tags: [swarm, client-software, reference-implementation]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-whitepaper, book-of-swarm, swarm-protocol-spec]
---
# Bee Client

**What it is:** The reference client implementation of the Swarm network.

## Notes
Bee (github.com/ethersphere/bee) is the reference node software for [[swarm]]; Bee 1.0 was the mainnet launch release that shipped the functionality described in the Swarm whitepaper — the DISC chunk store, [[kademlia]] forwarding/backwarding routing, push/pull syncing, the [[swap-protocol]] bandwidth accounting, [[postage-stamps]] storage, manifests, feeds, and PSS messaging. It implements the versioned [[libp2p]]-based wire protocols (handshake, hive, retrieval 1.4.0, pushsync 1.3.0, pullsync 1.3.0, pricing, pseudosettle) defined in the Swarm protocol spec.

Bee is to [[swarm]] roughly what [[lotus]] is to [[filecoin]]: the canonical node software embodying the protocol. It runs against [[gnosis-chain]] for BZZ settlement and incentive contracts.

## Sources
- [[swarm-whitepaper]] — Bee 1.0 mainnet launch scope
- [[book-of-swarm]] — design intent the Bee client realizes
- [[swarm-protocol-spec]] — wire protocols Bee implements
