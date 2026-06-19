---
title: Expected Consensus
type: concept
tags: [expected-consensus, filecoin, consensus, vrf, leader-election]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Expected Consensus

**Definition:** Expected Consensus (EC) is Filecoin's probabilistic, Byzantine-fault-tolerant leader election in which a miner's chance of winning a block in each epoch is proportional to its quality-adjusted storage power.

## How it works
Each 30-second epoch, every miner runs a [[vrf]]-based election seeded by [[drand]] beacon randomness. Win probability is set by *Poisson sortition* over the miner's share of quality-adjusted power held in the [[storage-power-consensus]] Power Table, yielding a `WinCount` (so a single lucky miner can earn multiple block rewards in one round while the scheme stays Sybil-invariant). All winning blocks at a given height form a `tipset`; the canonical chain is the heaviest one (GHOST-style cumulative weight), where weight rewards both total network power and the number of blocks per round. EC provides soft finality at roughly 900 epochs and slashes consensus faults — double-fork mining, time-offset mining, and parent-grinding — with the slasher reward growing exponentially with reporting delay.

## Why it matters / trade-offs
EC is what makes *useful storage* the consensus resource: instead of burning electricity (PoW) or staking tokens (PoS), influence tracks provable storage, so Sybil resistance and useful work coincide. Poisson sortition keeps it stake-/power-proportional and unbiasable via the external drand beacon. Trade-offs: finality is only probabilistic (≈900-epoch soft finality, no instant settlement), and the tipset model adds complexity over single-block chains. EC depends on continuous [[proof-of-spacetime]] to keep power honest.

## Related
[[storage-power-consensus]], [[vrf]], [[drand]], [[proof-of-spacetime]], [[proof-of-replication]], [[sector]], [[slashing]], [[filecoin]]

## Sources
- [[filecoin-spec]] — EC leader election, Poisson sortition, tipsets, weight, finality, consensus faults
