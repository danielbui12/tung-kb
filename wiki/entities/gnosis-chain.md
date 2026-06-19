---
title: Gnosis Chain
type: entity
tags: [blockchain, evm, swarm, settlement]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, object-storage-domain-knowledge]
---
# Gnosis Chain

**What it is:** An EVM-compatible blockchain hosting Swarm's storage/redistribution contracts and BZZ settlement.

## Notes
Gnosis Chain is where [[swarm]]'s on-chain [[storage-incentive-mechanisms]] live in production: the [[postage-stamps]] (postage) contract, the [[redistribution-game]]'s game/staking/price-oracle contracts, and [[swap-protocol]] chequebook settlement, all denominated in the BZZ token. Using an EVM sidechain rather than [[ethereum]] mainnet keeps the per-chunk storage rent and frequent commit/reveal redistribution rounds cheap.

It plays the role for [[swarm]] that [[sui]] plays for [[walrus]] — the chain anchoring the storage network's economic state. Contrast [[filecoin]], which is its own purpose-built blockchain rather than settling on a host chain.

## Sources
- [[book-of-swarm]] — Swarm contract suite and BZZ settlement
- [[object-storage-domain-knowledge]] — Gnosis Chain as Swarm's settlement layer
