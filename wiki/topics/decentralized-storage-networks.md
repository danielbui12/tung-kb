---
title: Decentralized Storage Networks
type: topic
tags: [decentralized-storage, object-storage, blockchain, comparison]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [object-storage-domain-knowledge, storj-v2, storj-v3, filecoin-spec, arweave-yellow-paper, arweave-2-9, book-of-swarm, swarm-whitepaper, swarm-protocol-spec, erasure-coding-algorithms-comparison]
---
# Decentralized Storage Networks

**Thesis / current understanding:** A decentralized storage network (DSN) stores bytes on a permissionless set of untrusted nodes, replacing a trusted operator with **cryptography + economic incentives**. The master design variable is the *redundancy + proof model*: how the network keeps data durable (replication vs [[erasure-coding]]) and how it proves nodes are still holding it (audits vs [[proof-of-replication]]/[[proof-of-spacetime]] vs [[proof-of-access]] vs Schelling games). Everything else — token design, metadata handling, retrieval path — follows from that choice. All these systems sit on the shared [[content-addressed-data-structures]] substrate (hashes, CIDs, Merkle DAGs).

## Landscape

The four networks in this wiki, plus the [[walrus]] and [[ipfs]] reference points:

| Network | Durability model | Proof of storage | Coordination | Payment model | Token |
|---|---|---|---|---|---|
| [[storj]] (v3) | [[reed-solomon]] [[erasure-coding]] across nodes + repair | random-stripe [[audits]] (Berlekamp-Welch) | trusted-but-blind [[satellite]]s hold metadata | ongoing, off-chain accounting | [[storj-token|STORJ]] |
| [[filecoin]] | per-replica sealed [[sector]]s + repair miners | [[proof-of-replication]] + [[proof-of-spacetime]] (zk-SNARK) on-chain | global [[blockchain|blockchain]] / [[expected-consensus]] | time-bounded paid deals | FIL |
| [[arweave]] | probabilistic, competition-driven replication | [[proof-of-access]] + RandomX mining over the [[blockweave]] | own [[blockchain|blockchain]] | pay-once + [[storage-endowment]] | AR |
| [[swarm]] | neighborhood replication over [[kademlia]] + optional EC | [[redistribution-game]] (staked Schelling game) | [[kademlia]] overlay + [[gnosis-chain]] contracts | prepaid decaying rent ([[postage-stamps]]) | [[bzz-token|BZZ]] |
| [[walrus]] | 2-D "Red Stuff" [[erasure-coding]], self-healing | on-chain Point of Availability | [[sui]] blockchain | deal-based | WAL |
| [[ipfs]] | **none** (substrate only) | none | [[distributed-hash-table]] | none | — |

[[ipfs]] is the common ancestor: it provides content-addressing and P2P transport but *no persistence guarantee*. Filecoin, and to varying degrees the others, add the incentive + proof layer that IPFS lacks.

## Axes of variation

- **Permanence vs rent.** [[arweave]] is pay-once-store-forever (data permanence funded by an endowment). [[filecoin]], [[storj]], and [[swarm]] are contract/rent-based — storage lapses when payment stops. See [[permanent-storage]].
- **Durability mechanism.** [[storj]]/[[walrus]] lean on [[erasure-coding]] (low overhead, needs a read quorum); [[filecoin]] keeps full unique replicas per [[sector]]; [[arweave]] relies on emergent redundancy from many miners each wanting recall-block coverage; [[swarm]] uses proximity-based replication of 4 KB [[chunk]]s.
- **Trust placement.** [[storj]] concedes a trusted-but-blind coordinator ([[satellite]]) for metadata and audits, explicitly avoiding a blockchain on the hot path ([[coordination-avoidance]]). [[filecoin]]/[[arweave]]/[[swarm]] are fully on-chain-coordinated.
- **Workload.** [[filecoin]]/[[arweave]] target bulk/cold/archival storage; [[swarm]] targets small-chunk, low-latency, web-like access; [[storj]] targets S3-compatible hot object storage.
- **Fixed-`n` coding vs rateless coding.** Despite churning, unpredictable peer availability sounding like a natural fit for rateless [[fountain-codes]] (no need to know `n` in advance), both [[storj]] and [[walrus]] chose fixed-`n` [[reed-solomon]] in production — Walrus's v1 paper even proposed RaptorQ before shipping RS. This suggests MDS optimality and mature tooling outweigh rateless convenience once a DSN's storage "channel" is provisioned well enough to fix `n`, though it's an inference from two examples rather than a general law.

## Open tensions / contradictions

- **Erasure coding vs unique replication for durability.** Storj treats per-node [[erasure-coding]] + active repair as sufficient (claims ~11 nines from 29-of-80 RS); Filecoin argues durability must be *proven unique* via [[proof-of-replication]] to resist a node faking multiple replicas from one copy. These are different threat models, not just different parameters.
- **Is "permanent" storage credible?** Arweave's [[storage-endowment]] assumes the real cost of storage keeps falling fast enough for a one-time fee to fund storage forever — an economic bet, not a cryptographic guarantee.
- **Coordination avoidance vs decentralization.** Storj's [[satellite]] model scales and stays cheap but reintroduces a trusted party; the others pay a coordination/latency cost to avoid it.

## Open questions
- How do real-world repair costs ([[repair-bandwidth]]) compare across these networks at scale?
- Where does [[walrus]]-style data-availability (for rollups) diverge from long-term persistence?

## Sources
- [[object-storage-domain-knowledge]] — the comparative framing across all five networks
- [[storj-v3]], [[storj-v2]] — erasure-coded, satellite-coordinated model
- [[filecoin-spec]] — proof-backed, blockchain-coordinated model
- [[arweave-yellow-paper]], [[arweave-2-9]] — permanent-storage / blockweave model
- [[book-of-swarm]], [[swarm-whitepaper]], [[swarm-protocol-spec]] — chunk/Kademlia/postage model
- [[erasure-coding-algorithms-comparison]] — fountain-codes-vs-RS framing for the fixed-`n` vs rateless axis
