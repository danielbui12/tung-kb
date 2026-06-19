---
title: Filecoin
type: entity
tags: [decentralized-storage, blockchain, proof-of-storage, zk-snark]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, object-storage-domain-knowledge]
---
# Filecoin

**What it is:** A decentralized storage network and blockchain whose consensus power derives from provable, useful storage rather than wasted computation.

## Notes
Built by [[protocol-labs]], Filecoin is the paid persistence layer beneath [[ipfs]]/[[content-addressing]] — client files become UnixFS [[merkle-dag]]s identified by CIDs, and internal state is [[ipld]] DAG-CBOR. It is one of the [[decentralized-storage-networks]] and a flagship [[proof-of-storage-systems]] design.

Storage is committed in fixed-size [[sector]]s (32/64 GiB). Each sector is [[sealing|sealed]] via [[proof-of-replication]] (Stacked-DRG PoRep) — a slow, non-parallelizable encoding producing a unique replica tied to data, miner identity, and time, with a Groth16 [[zk-snark]] compressing the proof. Miners then prove continued possession via [[proof-of-spacetime]] (PoSt): WinningPoSt gates block production; WindowPoSt audits all sectors every 24h, with faults slashed.

Consensus is Expected Consensus, a VRF leader election seeded by the [[drand]] randomness beacon, weighting miners by quality-adjusted power. Token: FIL. Reference implementation: [[lotus]]. Networking via [[libp2p]]; CIDs via [[multiformats]].

Unlike [[storj]]'s lightweight economic [[audits]] or [[arweave]]'s mining-fused proof, Filecoin uses heavy trustless cryptographic proofs and must prove replica *uniqueness*. Compare [[swarm]] and [[walrus]].

## Sources
- [[filecoin-spec]] — consensus, PoRep/PoSt, VM/actors, cryptography, token economy
- [[object-storage-domain-knowledge]] — comparative redundancy/proof positioning
