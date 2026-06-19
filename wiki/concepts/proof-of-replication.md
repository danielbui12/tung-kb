---
title: Proof Of Replication
type: concept
tags: [proof-of-replication, filecoin, sealing, zk-snark, proof-of-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, object-storage-domain-knowledge]
---
# Proof Of Replication

**Definition:** PoRep is a proof that a miner has created and is storing a *unique physical replica* of some data, bound to its own identity and the time of creation, so it cannot fake holding many copies it does not actually keep.

## How it works
A miner runs [[sealing]] over a [[sector]] using the [[stacked-drg]] (SDR) construction: a deliberately slow, non-parallelizable, regeneration-resistant encoding. Sealing yields the unsealed-data commitment CommD and the sealed-replica commitment CommR. Uniqueness comes from three inputs folded into the seal: the data itself, the miner's ProverID (identity), and `SealRandomness` drawn from the [[vrf]] chain (time), which blocks replay and long-range attacks. The bulky "vanilla" PoRep is then compressed into a Groth16 [[zk-snark]] cheap enough to verify on-chain. Filecoin Merkle trees use the [[poseidon-hash]]; CommR = H(CommC ‖ CommRLast).

## Why it matters / trade-offs
PoRep defeats *generation attacks* (regenerate data on demand instead of storing it) and *Sybil/outsourcing attacks* (claim N copies while keeping one). It is the gate to earning Filecoin consensus power. The cost is that sealing is intentionally expensive in CPU and time — a real bottleneck miners must provision for — which is why upgrades like Narrow Stacked Expander aim to cut sealing cost. PoRep proves a replica *existed at seal time*; continued storage is proven separately by [[proof-of-spacetime]].

## Related
[[proof-of-spacetime]], [[stacked-drg]], [[sealing]], [[sector]], [[zk-snark]], [[poseidon-hash]], [[vrf]], [[proof-of-storage]], [[collateral]], [[filecoin]]

## Sources
- [[filecoin-spec]] — PoRep definition, SDR sealing, CommD/CommR, SNARK compression
- [[object-storage-domain-knowledge]] — replication as a durability mechanism
