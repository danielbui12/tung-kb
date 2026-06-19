---
title: Lotus
type: entity
tags: [filecoin, client-software, reference-implementation, go]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Lotus

**What it is:** The reference implementation of Filecoin, written in Go.

## Notes
Lotus is the canonical [[filecoin]] node software, implementing the protocol's subsystems — ChainSync, message pool, the [[filecoin]] VM and actors, and the miner subsystems performing [[sealing]] ([[proof-of-replication]]) and [[proof-of-spacetime]] proving. It runs Expected Consensus leader election seeded by [[drand]] and networks over [[libp2p]].

Lotus is to [[filecoin]] what [[bee-client]] is to [[swarm]]: the reference node embodying the spec. Alternative Filecoin implementations include Venus (Go), Forest (Rust), and Fuhon (C++). The spec notes several constants are flagged WIP, so live Lotus values may diverge from spec figures.

## Sources
- [[filecoin-spec]] — Lotus listed as reference Go implementation among Venus/Forest/Fuhon
