---
title: Collateral (Filecoin)
type: concept
tags: [collateral, cryptoeconomics, slashing, filecoin, storage-incentive-mechanisms]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Collateral (Filecoin)

**Definition:** FIL that a Filecoin miner must pledge to back its storage promises. It comes in three forms — an initial pledge, block rewards vesting as collateral, and storage-deal provider collateral — and is forfeited via [[slashing]] when the miner faults.

## How it works
The **initial pledge** posted per [[sector]] combines a *storage pledge* (≈20 days of that sector's expected block reward) and a *consensus pledge* targeting ~30% of circulating supply locked, scaled by the sector's quality-adjusted power over the network baseline. **Block reward as collateral**: rewards earned do not vest immediately but linearly over time, so they remain at stake to back the miner's commitments and are slashed on early sector termination. **Deal collateral** is provider stake tied to specific storage deals. Faults trigger graduated penalties — a Fault Fee per offline day, a Sector Penalty for undeclared faults caught at WindowPoSt (see [[proof-of-spacetime]]), and a larger Termination Penalty — while consensus faults (double-fork, time-offset, parent-grinding) cost the miner *all* pledge collateral.

## Why it matters / trade-offs
Collateral aligns incentives: a miner that drops a [[proof-of-spacetime]] audit or abandons data loses real value, so honest storage is the profit-maximizing strategy. Layering hardware cost with token stake raises the cost of attacks beyond pure capital. The trade-off is capital efficiency — large up-front lockups raise the barrier to entry and tie miner economics to FIL price volatility.

## Related
[[slashing]] [[proof-of-spacetime]] [[proof-of-replication]] [[expected-consensus]] [[storage-incentive-mechanisms]] [[filecoin]]

## Sources
- [[filecoin-spec]] — initial pledge (storage + consensus components), vesting block rewards as collateral, deal collateral, fault/termination/consensus-fault penalties
