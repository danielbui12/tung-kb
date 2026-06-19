---
title: Redistribution Game
type: concept
tags: [redistribution-game, swarm, schelling-game, incentives, slashing]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec]
---
# Redistribution Game

**Definition:** Swarm's on-chain commit–reveal Schelling game that redistributes accumulated postage-stamp rent to nodes that prove they are storing their share of data, paying an honest majority and freezing or slashing liars.

## How it works
Uploaders pre-pay storage rent into postage batches; that rent accumulates in a pot to be redistributed. Four contracts (postage, game, staking, price oracle) run the game over commit / reveal / claim phases. A random neighbourhood-selection *anchor* picks a winning neighbourhood. Staked storer nodes in it commit to a *reserve sample* — the first k chunks of their reserve ordered by a salted modified-BMT hash — then reveal it. Because honest nodes storing the same area of responsibility independently arrive at the *same* sample, this is a [[schelling-game|Schelling point]]: stake-weighted honest-majority consensus selects the truthful sample, probabilistically pays the whole pot to one winner, and freezes/slashes nodes whose reveals diverge (liars and saboteurs). Correctness rests on a density-based *reserve-size proof* and inclusion proofs enforcing the "rules of the reserve." The number of honest revealers `r` also feeds the price oracle (`p_{t+1} = 2^{σ(4−r)} p_t`).

## Why it matters / trade-offs
It solves storage payment without per-chunk metering: instead of proving every chunk to every payer, nodes are *sampled* and rewarded in expectation, while [[collateral]] [[slashing]] makes lying unprofitable. The same revealer count doubles as a supply/demand price signal driving automatic price discovery. Trade-offs: rewards are probabilistic (high variance for small storers), it assumes an honest stake-weighted majority per neighbourhood, and it proves storage of a *sample*, not every chunk, so it is an economic rather than absolute durability guarantee.

## Related
[[audits]], [[collateral]], [[slashing]], [[merkle-tree]], [[proof-of-storage]], [[content-addressing]], [[swarm]], [[storage-incentive-mechanisms]]

## Sources
- [[book-of-swarm]] — Schelling game, reserve sample, four contracts, price-oracle signal
- [[swarm-protocol-spec]] — postage stamps, staking, commit/reveal/claim mechanics
