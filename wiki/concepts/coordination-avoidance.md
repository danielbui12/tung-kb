---
title: Coordination Avoidance
type: concept
tags: [coordination-avoidance, scalability, bar-model, storj, decentralized-storage-networks]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3]
---
# Coordination Avoidance

**Definition:** A [[storj]] v3 design principle: minimize synchronous cross-actor coordination on the data path — no global ledger, no Byzantine consensus on hot operations — so the network can scale to exabytes. Storj reasons about participants under the Byzantine-Altruistic-Rational (BAR) fault model.

## How it works
Storj deliberately keeps the read/write hot path coordination-free. Uplinks encrypt, erasure-code (see [[erasure-coding]] and [[client-side-encryption]]), and transfer pieces *directly* to storage nodes, peer-to-peer. Metadata is partitioned per-user and per-Satellite rather than maintained in one global structure, so there is no shared state to agree on for ordinary operations. Authorization uses macaroons (caveatable bearer tokens) so access delegation needs no central check. Bandwidth is accounted with signed, incrementally-restricted allocations (order limits and orders) that make cheating protocol-impossible without any consensus round. The BAR model assumes participants are Byzantine, Altruistic, or Rational, and the system is engineered so rational actors find honesty most profitable rather than relying on a quorum to outvote Byzantine ones.

## Why it matters / trade-offs
Synchronous coordination (a global ledger or BFT consensus) is the scalability ceiling for distributed storage; avoiding it on the hot path is what lets Storj target exabyte scale with low latency. The whitepaper explicitly defers blockchain to monthly payment settlement only. The trade-off is the trusted-but-blind Satellite: it must stay online for repair and metadata, a single-point-of-availability that a fully consensus-based design would not have — the long-term goal to "architect the Satellite out" awaits a fast scalable BFT algorithm.

## Related
[[client-side-encryption]] [[erasure-coding]] [[storj]] [[decentralized-storage-networks]] [[storage-incentive-mechanisms]]

## Sources
- [[storj-v3]] — coordination avoidance as a first-class goal, BAR fault model, per-user/per-Satellite metadata, signed bandwidth allocations, rationale for avoiding Byzantine consensus
