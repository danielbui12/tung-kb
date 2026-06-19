---
title: Arweave
type: entity
tags: [decentralized-storage, permanent-storage, blockchain, proof-of-access]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-yellow-paper, arweave-2-9, object-storage-domain-knowledge]
---
# Arweave

**What it is:** A permanent ("pay-once, store-forever") decentralized storage protocol funded by a self-sustaining storage endowment.

## Notes
Arweave is the [[permanent-storage]] member of the [[decentralized-storage-networks]] and a distinctive [[proof-of-storage-systems]] design. Its core data structure is the [[blockweave]]: each block links to both the previous block and an unpredictable historical "recall block." Consensus is [[proof-of-access]] (PoA), folding recall-block data into the proof-of-work hash so a miner must possess that data to mine — making storage a prerequisite for rewards and turning replication into a probabilistic competition rather than fixed contracts (contrast [[filecoin]]'s contracted sealed replicas).

Economics are the standout: users pay AR once, most of which funds a storage endowment modeled on an infinite sum of historically declining storage costs (~0.5%/yr assumed conservatively vs ~30% historical). This contrasts with the recurring rent of [[storj]]/[[filecoin]]/[[walrus]] and the deliberate forgetting of [[swarm]]. Together the blockweave and PoA serve the permaweb — on-chain websites and apps. Token: AR (cap 66M; 1 AR = 10^12 Winston).

**Arweave 2.9** replaces data "packing" with an "activation" scheme on RandomXSquared, recycling nearly all RandomX entropy to cut honest-miner compute ~99.6% and disk read rates ~90% while preserving the on-demand mining security margin and defeating ISTORE-caching attacks.

## Sources
- [[arweave-yellow-paper]] — blockweave, Proof of Access, endowment economics
- [[arweave-2-9]] — RandomXSquared activation, entropy dispersion, attack analysis
- [[object-storage-domain-knowledge]] — comparative permanence/proof positioning
