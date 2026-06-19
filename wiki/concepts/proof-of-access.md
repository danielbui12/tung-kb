---
title: Proof Of Access
type: concept
tags: [proof-of-access, arweave, proof-of-work, blockweave, proof-of-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-yellow-paper, arweave-2-9]
---
# Proof Of Access

**Definition:** PoA is [[arweave]]'s consensus mechanism: an extension of proof-of-work in which a randomly chosen historical "recall block" is folded into the material hashed for the mining puzzle, so mining is only possible if a node actually stores past data.

## How it works
In the [[blockweave]] each block links both to its predecessor and to a *recall block* chosen deterministically but unpredictably (`BHL[indep_hash mod height]`), which cannot be known until the prior block is mined. To compute the PoW hash a miner must possess that recall block's full data; without it, it cannot mine or validate. A node's probability of winning is the product of the fraction of historical blocks it stores and its relative hashing power (`P = (Blocks_local / B_height) · (HP_local / HP_net)`). The reference miner hashes with [[randomx]]. Storing *rare* blocks is rewarded more (fewer competitors), pushing the network toward replicating under-stored data.

## Why it matters / trade-offs
PoA turns storage into the scarce resource backing consensus, but unlike Filecoin's fixed contract-based replica counts it is *probabilistic and competition-driven*: redundancy emerges from miners chasing reward rather than from explicit contracts. This makes replication self-balancing but not individually guaranteed. Later work (Arweave 2.9, RandomXSquared) hardens the cost asymmetry so that storing data stays cheaper than regenerating it on demand, widening the security margin while cutting honest-miner compute.

## Related
[[proof-of-work]], [[blockweave]], [[randomx]], [[proof-of-storage]], [[proof-of-replication]], [[arweave]], [[proof-of-storage-systems]]

## Sources
- [[arweave-yellow-paper]] — PoA, recall block, blockweave, win-probability formula
- [[arweave-2-9]] — strengthening the store-vs-regenerate cost invariant
