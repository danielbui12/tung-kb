---
title: Proof-of-Storage Systems
type: topic
tags: [proof-of-storage, consensus, cryptography, decentralized-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge, filecoin-spec, storj-v2, storj-v3, arweave-yellow-paper, arweave-2-9, book-of-swarm, swarm-protocol-spec]
---
# Proof-of-Storage Systems

**Thesis / current understanding:** The defining problem of a decentralized storage network is verifying that an untrusted node *still holds* specific data over time, without re-downloading it. [[proof-of-storage]] is the umbrella; each network answers it differently, and the answer shapes the whole system — whether storage can back consensus, how often nodes must respond, and how cheating is punished. Approaches split into **interactive challenge-response** (audits, proof-of-retrievability), **self-contained cryptographic proofs** (PoRep/PoSt, compressed via [[zk-snark]]), **storage-as-mining** (proof-of-access), and **staked coordination games** (Schelling redistribution).

## Landscape

- **[[audits]] / [[proof-of-retrievability]]** ([[storj]]) — the verifier challenges a node for a random stripe/shard; the node returns a Merkle proof ([[storj-v2]]) or the data is checked with Berlekamp-Welch ([[storj-v3]]). Privately verifiable, off-chain, cheap. Storj v2's pre-generated Merkle challenges were exploitable (a node could store only expected answers); v3's on-demand random stripes closed that.
- **[[proof-of-replication]] + [[proof-of-spacetime]]** ([[filecoin]]) — PoRep proves a *unique physical replica* tied to miner identity + time, produced by slow [[sealing]] ([[stacked-drg]]). PoSt proves continuous storage: WinningPoSt (instant, gates block production) and WindowPoSt (24h audit). Both compressed to a [[zk-snark]] for cheap on-chain verification using [[poseidon-hash]]. Storage *is* the consensus weight ([[expected-consensus]], [[storage-power-consensus]]).
- **[[proof-of-access]]** ([[arweave]]) — folds a randomly chosen historical "recall block" into the PoW hash over the [[blockweave]], so a miner must store past data to mine. [[arweave-2-9]] reworks the entropy/packing economics (RandomXSquared) to keep honest cost low while raising the cost of on-demand re-fetching.
- **[[redistribution-game]]** ([[swarm]]) — a staked commit-reveal Schelling game: storers in the selected neighborhood prove reserve size via density-based sampling; honest revealers split the redistributed [[postage-stamps]] rent, liars are frozen/slashed.

## Cross-cutting structure

| Property | Storj audits | Filecoin PoRep/PoSt | Arweave PoA | Swarm game |
|---|---|---|---|---|
| Verifier | satellite (private) | on-chain (public) | on-chain (public) | on-chain (public) |
| Backs consensus? | no | **yes** | yes (PoW) | no (rewards only) |
| Cost to prover | low | high (sealing) | mining work | low (sampling) |
| Punishment | disqualification | [[slashing]] [[collateral]] | lost block reward | freeze / [[slashing]] |

Unbiasable randomness is a shared dependency: Filecoin uses [[drand]] + [[vrf]]; Swarm derives it from the game; Arweave from the recall-block selection.

## Open tensions / contradictions
- **Uniqueness vs retrievability.** PoRep proves a replica is *physically unique*; audits only prove *retrievability right now*. The former resists a node faking N replicas from one copy; the latter is far cheaper but trusts replica counts to economics + placement.
- **Public vs private verifiability.** Filecoin/Arweave/Swarm proofs are globally checkable; Storj's are private to the satellite — simpler and cheaper, but reintroduces a trusted verifier.

## Open questions
- How does the prover-side cost of [[sealing]] limit Filecoin's onboarding throughput vs Storj's near-free ingest?

## Sources
- [[filecoin-spec]] — PoRep, PoSt, Stacked DRG, zk-SNARK proof compression
- [[storj-v2]], [[storj-v3]] — Merkle-proof and random-stripe audits
- [[arweave-yellow-paper]], [[arweave-2-9]] — Proof of Access and its packing economics
- [[book-of-swarm]], [[swarm-protocol-spec]] — the redistribution Schelling game
- [[object-storage-domain-knowledge]] — comparative proof framing
