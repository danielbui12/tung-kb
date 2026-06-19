---
title: Proof Of Spacetime
type: concept
tags: [proof-of-spacetime, filecoin, consensus, audits, proof-of-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, object-storage-domain-knowledge]
---
# Proof Of Spacetime

**Definition:** PoSt is a proof that a miner is storing a sealed replica *continuously over time*, not merely that it once created it. Filecoin uses two variants: WinningPoSt and WindowPoSt.

## How it works
After [[sealing]] produces a replica ([[proof-of-replication]]), the miner must keep answering challenges over the life of the [[sector]]. **WinningPoSt** is demanded only of a miner elected to produce a block in a given epoch: it proves a live replica at one specific instant, must be returned within a short deadline, and earns the block reward — missing it just forfeits the reward, no penalty. **WindowPoSt** is the continuous audit: each 24-hour proving period is divided into 48 half-hour deadlines; sectors are split into partitions (2349 sectors, 10 challenges each) and one [[zk-snark]] proof per partition is published per deadline, so every sector is audited at least once daily. Missing WindowPoSt produces a fault and triggers [[slashing]].

## Why it matters / trade-offs
PoSt is what keeps consensus power honest: power decays unless storage is continuously re-proven, so a miner cannot seal once and walk away. The two-variant split separates roles — WinningPoSt is an instantaneous liveness check gating block production, WindowPoSt is the economically-enforced durability audit. The trade-off is proving overhead: miners spend ongoing compute generating SNARK proofs across all sectors every day, and accidental downtime risks slashing collateral.

## Related
[[proof-of-replication]], [[sealing]], [[sector]], [[expected-consensus]], [[storage-power-consensus]], [[zk-snark]], [[slashing]], [[audits]], [[proof-of-storage]], [[filecoin]]

## Sources
- [[filecoin-spec]] — WinningPoSt vs WindowPoSt mechanics, deadlines, partitions, faults
- [[object-storage-domain-knowledge]] — continuous durability verification
