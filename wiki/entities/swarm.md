---
title: Swarm
type: entity
tags: [decentralized-storage, web3, kademlia, storage-incentives]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-whitepaper, swarm-protocol-spec, object-storage-domain-knowledge]
---
# Swarm

**What it is:** A permissionless peer-to-peer storage and communication layer for Web3 — the "hard disk" of the Ethereum world computer.

## Notes
Swarm is the permissionless-web-platform member of the [[decentralized-storage-networks]], designed by [[viktor-tron]]. Its store (DISC) holds fixed-size (≤4KB) content-addressed [[content-addressing|chunks]] at the nodes closest to each chunk address over a [[kademlia]] overlay ordered by proximity. It uses *forwarding* (recursive) Kademlia plus reverse-path "backwarding," which gives logarithmic routing and built-in requestor anonymity. Two chunk types exist: content-addressed (BMT/Keccak256 hash) and single-owner (signed, writable subspace) — the latter underpins feeds and PSS messaging. Redundancy comes from address-proximity neighborhood replication rather than [[erasure-coding]] (contrast [[storj]]/[[walrus]]) or contracted replicas (contrast [[filecoin]]).

Incentives run on two axes: bandwidth via the [[swap-protocol]] (off-chain cheques settled on-chain), and storage via [[postage-stamps]] (prepaid, decaying rent). Accumulated rent is redistributed by the [[redistribution-game]], a staked Schelling game whose proof-of-density reserve-sampling check is a [[proof-of-storage]] of sufficient reserve size. Unpaid data is deliberately garbage-collected — Swarm forgets, unlike [[arweave]]'s permanence.

Token: BZZ on [[gnosis-chain]] (incentive contracts on [[ethereum]]). Networking via [[libp2p]]; naming via [[ens]]. Reference client: [[bee-client]].

## Sources
- [[book-of-swarm]] — full design: DISC, chunks, SWAP, stamps, redistribution game
- [[swarm-whitepaper]] — architecture overview and Bee 1.0 launch
- [[swarm-protocol-spec]] — wire protocols and incentive-layer constants
- [[object-storage-domain-knowledge]] — comparative redundancy/proof positioning
