---
title: Replica Sync and Data Transfer
type: topic
tags: [sync, replication, data-transfer, decentralized-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-protocol-spec, book-of-swarm, filecoin-spec, storj-v3, storj-v2, arweave-yellow-paper]
---
# Replica Sync and Data Transfer

**Thesis / current understanding:** "Sync" means three *different* things across these systems, and conflating them is the usual source of confusion. (1) **Replica sync** — keeping the *copies of stored data* converged across nodes. (2) **Chain/state sync** — keeping the *ledger* consistent. (3) **Data transfer** — the bulk-movement layer underneath both. Only **[[swarm]]** has a true peer-to-peer *replica*-sync protocol; **[[storj]]** does centrally-orchestrated *repair*; **[[filecoin]]** has chain sync + a deal-bound transfer layer but **no replica auto-repair**; **[[arweave]]** has propagation, not sync.

## Landscape

| System | Replica convergence | Mechanism | Coordinator? | Page |
|---|---|---|---|---|
| [[swarm]] | peer anti-entropy | [[pull-sync]] (offer/want + interval store) + [[push-sync]] (write replication) | none (Kademlia neighbors) | [[pull-sync]], [[push-sync]] |
| [[storj]] | orchestrated repair | [[storage-repair]] (audit → RS-reconstruct → redistribute) | [[satellite]] (trusted) | [[storage-repair]] |
| [[filecoin]] | **none** (no auto-repair) | client makes redundant deals; lost sectors just lose power | n/a | [[storage-repair]] (contrast) |
| [[arweave]] | emergent | miners independently fetch to cover recall blocks | n/a | — |

| System | Chain/state sync | Data transfer (bulk) |
|---|---|---|
| [[filecoin]] | [[chainsync]] (FSM + BlockSync + GossipSub + BV0–BV8) | [[graphsync-data-transfer]] (GraphSync/Bitswap, voucher-bound) |
| [[swarm]] | n/a (incentives on [[gnosis-chain]]) | [[retrieval-protocol]] (forward/backward, anonymous) |
| [[storj]] | n/a (off-chain accounting) | direct uplink↔node transfer, satellite-brokered |

## The three replica strategies, compared

- **Swarm — pull-based eventual consistency.** Each node runs one worker per (peer, bin), pulling ranges its [[pull-sync|interval store]] marks unfilled; an `Offer/Want` bitvector exchange avoids re-sending chunks it already holds; an epoch fingerprint forces a full re-sync if a peer wiped its reserve. [[push-sync]] handles write-time replication with signed receipts. No coordinator — neighborhoods self-heal as topology shifts, and the [[redistribution-game]] economically *forces* nodes to retain what they sync.
- **Storj — threshold-triggered repair.** [[audits]] update node/piece health; when a segment's healthy pieces fall to the repair threshold (≈ `m` in the `k ≤ m ≤ o ≤ n` ladder), the [[satellite]] downloads `k` surviving pieces, [[reed-solomon]]-reconstructs, re-encodes only the *missing* pieces, uploads them to fresh high-reputation nodes, and rewrites the segment pointer. Cheap egress (only missing pieces) but centralized — repair only runs while the satellite runs.
- **Filecoin — no replica sync at all.** Each replica is an independently [[sealing|sealed]] [[sector]] ([[proof-of-replication]] makes them non-identical *by design*), so there is nothing to "sync" and no repair primitive. Durability = economic incentive ([[proof-of-spacetime]] + [[slashing]]) plus the client striking multiple deals. Filecoin's sync machinery is for the *chain* ([[chainsync]]) and *deal data movement* ([[graphsync-data-transfer]]), not replicas.

## Open tensions / contradictions
- **Anti-entropy vs orchestration.** Swarm's coordinator-free pull-sync is robust to any single failure but offers only *eventual* consistency and probabilistic redundancy; Storj's satellite gives crisp durability accounting and targeted repair but reintroduces a trusted party and a single operational dependency.
- **"Repair" is not universal.** A mental model where "the network heals lost replicas" is true for Storj/Swarm but **false for Filecoin** — a frequent and costly misconception when porting designs.

## Open questions
- How does Swarm pull-sync's redundancy degrade under rapid neighborhood churn vs Storj's repair-threshold guarantees? (Swarm rationale partly in [[book-of-swarm]], not fully quantified.)
- What are Filecoin BlockSync's concrete rate-limit/`GoAway` parameters on mainnet vs the spec?

## Sources
- [[swarm-protocol-spec]], [[book-of-swarm]] — pull-sync, push-sync, retrieval wire protocols and rationale
- [[filecoin-spec]] — ChainSync FSM, BlockSync, GraphSync/Bitswap data transfer
- [[storj-v3]], [[storj-v2]] — audit-driven satellite repair pipeline
- [[arweave-yellow-paper]] — propagation (Wildfire/blockshadows), not replica sync
