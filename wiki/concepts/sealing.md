---
title: Sealing
type: concept
tags: [sealing, filecoin, proof-of-replication, stacked-drg]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Sealing

**Definition:** Sealing is the computation-heavy encoding of a [[sector]] into a unique, certified replica (committed as CommR); it is the prerequisite a Filecoin miner must complete before that sector can earn consensus power.

## How it works
A miner fills a sector with deal data or committed-capacity data, producing the unsealed-data commitment CommD. It then runs the [[stacked-drg]] (SDR) [[proof-of-replication]] encoding, a deliberately slow and non-parallelizable process that binds the replica to (i) the data, (ii) the miner's ProverID, and (iii) `SealRandomness` drawn from the [[vrf]] chain at seal time. The output is the sealed-replica commitment CommR (where CommR = H(CommC ‖ CommRLast)). The bulky vanilla proof is compressed into a Groth16 [[zk-snark]] for cheap on-chain verification, and the sector then begins continuous [[proof-of-spacetime]] proving.

## Why it matters / trade-offs
Sealing is the gate between raw disk space and *provable, power-earning* storage — it is what makes a replica unforgeable and time-stamped. Its deliberate expense is the security property (a replica cannot be cheaply regenerated on challenge) but also the dominant operational cost and bottleneck for miners, demanding significant CPU and time. The planned Narrow Stacked Expander upgrade targets this directly, aiming to lower sealing cost and retrieval latency.

## Related
[[sector]], [[proof-of-replication]], [[stacked-drg]], [[proof-of-spacetime]], [[zk-snark]], [[vrf]], [[storage-power-consensus]], [[filecoin]]

## Sources
- [[filecoin-spec]] — sealing pipeline, CommD/CommR, SealRandomness, SNARK compression
