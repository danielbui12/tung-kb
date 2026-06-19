---
title: Slashing
type: concept
tags: [slashing, collateral, cryptoeconomics, storage-incentive-mechanisms, mechanism-design]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge, filecoin-spec, book-of-swarm]
---
# Slashing

**Definition:** The forfeiture of posted collateral as an economic penalty when a node is proven to have faulted or behaved dishonestly, making misbehaviour cost more than the gain it could yield.

## How it works
A node locks up a stake or pledge before it can earn, and the protocol burns or redistributes part or all of that stake when a fault is detected. [[filecoin]] is the most elaborate case: miners post an initial pledge plus vesting block rewards as collateral, and storage faults (a missed [[proof-of-storage]] WindowPoSt) incur graduated penalties — a per-day Fault Fee, a Sector Penalty for undeclared faults, and a Termination Penalty — while consensus faults (double-fork, time-offset, parent-grinding) cost a miner all pledge collateral. [[swarm]]'s [[redistribution-game]] stakes nodes to participate in its Schelling-game proof of storage and freezes or slashes those whose committed reserve sample disagrees with the honest majority; its insurance ("swap, swear, swindle") layer burns ≥95% of a guilty insurer's deposit to close the collude-and-exit attack. Walrus likewise slashes collateral for unavailability.

## Why it matters / trade-offs
Slashing is the bonding glue of [[mechanism-design]] in storage networks: cryptographic proofs detect faults, but only the threat of losing real value makes honest storage the rational strategy and makes Sybil attacks expensive. It complements positive incentives (rewards) and softer signals like [[reputation-system]]s. The hazards are false positives — a node slashed for a transient outage or a buggy proof — and calibration: penalties too small fail to deter, too large drive away honest operators wary of correlated failures or client/coordinator misbehaviour.

## Related
[[redistribution-game]] [[proof-of-storage]] [[reputation-system]] [[mechanism-design]] [[storage-incentive-mechanisms]] [[filecoin]] [[swarm]]

## Sources
- [[object-storage-domain-knowledge]] — slashing across Filecoin/Swarm/Walrus, economics as binding
- [[filecoin-spec]] — fault fees, sector/termination penalties, consensus-fault slashing
- [[book-of-swarm]] — staking, freeze/slash in the redistribution game and insurance burn
