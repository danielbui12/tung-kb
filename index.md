---
title: Index
type: index
updated: 2026-06-19
---
# Index — tung-kb

Content catalog for the wiki. When answering a query, read this first to locate relevant pages, then drill in.
Updated on every ingest and every filed-back exploration.

**Current domain focus:** decentralized / distributed storage networks and the content-addressing primitives beneath them.

## Topics
_Broad subject areas with an evolving synthesis._
- [[decentralized-storage-networks]] — comparison of Storj, Filecoin, Arweave, Swarm (+ Walrus/IPFS) by durability, proof, coordination, and payment model.
- [[content-addressed-data-structures]] — the shared primitive stack: hashes → CIDs → Merkle DAGs.
- [[storage-redundancy-and-durability]] — replication vs erasure coding vs RAID, and the durability/repair trade-offs.
- [[proof-of-storage-systems]] — how networks prove untrusted nodes still hold data (audits, PoRep/PoSt, PoA, Schelling games).
- [[storage-incentive-mechanisms]] — token/payment/collateral/slashing designs that make honest storage profitable.
- [[replica-sync-and-data-transfer]] — the three meanings of "sync" and how each project keeps replicas/chain/data converged.

## Concepts
_Techniques, algorithms, patterns, principles._

### Foundational
- [[decentralized-storage]] — storing data across permissionless untrusted nodes via crypto + incentives.

### Content addressing & data structures
- [[content-addressing]] — naming data by the hash of its content rather than its location.
- [[location-addressing]] — naming data by where it is stored (URLs); the centralized-web contrast.
- [[cryptographic-hash]] — deterministic, one-way, collision-resistant fingerprint of data.
- [[content-identifier]] — the CID: self-describing content-addressed label (CIDv0 vs CIDv1).
- [[merkle-dag]] — DAG whose edges are child CIDs embedded in parents; self-verifying linked data.
- [[merkle-tree]] — hash tree giving compact (log n) inclusion proofs; ancestor of the Merkle DAG.
- [[directed-acyclic-graph]] — directed, acyclic graph; the shape Merkle DAGs specialize.
- [[deduplication]] — storing redundant data once via shared content-addressed nodes.
- [[chunking]] — splitting files into content-addressed pieces for dedup and fixed-size units.
- [[binary-merkle-tree]] — Swarm's Keccak256 BMT giving the chunk address + inclusion proofs.
- [[chunk]] — Swarm's fixed-size (≤4 KB) basic storage unit.
- [[single-owner-chunk]] — Swarm chunk in a signed, owner-writable address subspace (feeds, PSS).

### Redundancy & durability
- [[erasure-coding]] — split into k+m fragments; any k of n reconstruct, tolerating m losses.
- [[reed-solomon]] — the canonical MDS erasure code (Galois-field polynomial math).
- [[raid]] — disk-array redundancy; RAID-5/6 are fixed-recipe erasure codes in one box.
- [[redundancy]] — controlled extra storage so a surviving subset reconstructs the whole.
- [[replication]] — keeping whole copies (e.g. 3×); simple but storage-heavy.
- [[data-durability]] — probability data survives failures over time.
- [[repair-bandwidth]] — I/O cost to rebuild lost fragments; the EC repair tax.
- [[object-storage]] — flat namespace of immutable objects over S3-style APIs.
- [[block-storage]] — raw fixed-size blocks (a virtual disk).
- [[file-storage]] — hierarchical POSIX directories/files.

### Proofs & consensus
- [[proof-of-storage]] — umbrella: proving a node still holds data without re-reading it.
- [[proof-of-replication]] — Filecoin PoRep: proof of a unique physical replica (via sealing).
- [[proof-of-spacetime]] — Filecoin PoSt: proof of continuous storage over time (Winning/Window).
- [[proof-of-access]] — Arweave: PoW that requires storing a random historical recall block.
- [[proof-of-retrievability]] — challenge-response proof a host holds intact data (Storj audits).
- [[stacked-drg]] — Stacked DRG: the slow, non-parallelizable PoRep sealing construction.
- [[sealing]] — encoding a sector into a unique certified replica (CommR).
- [[sector]] — Filecoin's fixed-size (32/64 GiB) unit of sealed, proven storage.
- [[expected-consensus]] — Filecoin leader election weighted by quality-adjusted storage power.
- [[storage-power-consensus]] — subsystem maintaining the Power Table and brokering EC.
- [[audits]] — probabilistic random-stripe spot-checks of stored data (Storj).
- [[redistribution-game]] — Swarm's staked Schelling game redistributing postage rent to storers.
- [[schelling-game]] — staked coordination game rewarding convergence on the truth; basis of the redistribution game.

### Cryptography & VM
- [[poseidon-hash]] — SNARK-friendly hash used for Filecoin Merkle trees / CommR.
- [[zk-snark]] — succinct ZK proof compressing PoRep/PoSt for on-chain verification.
- [[bls-signatures]] — aggregatable signatures over BLS12-381 (Filecoin blocks/tickets).
- [[vrf]] — verifiable random function; Filecoin tickets and election proofs.
- [[randomx]] — CPU-bound ASIC-resistant PoW hash; Arweave's entropy primitive.
- [[client-side-encryption]] — encrypting data on the owner's device before upload (Storj).
- [[actor-model]] — Filecoin's smart-contract model (actors invoked via messages).
- [[filecoin-vm]] — replicated state machine executing actor code; emits the State Tree.
- [[collateral]] — pledged FIL backing miner promises; forfeited via slashing.
- [[tipset]] — set of same-height blocks sharing parents; the Filecoin chain unit.

### Sync, transfer & repair
- [[pull-sync]] — Swarm's pull-based anti-entropy replica sync (offer/want + interval store).
- [[push-sync]] — Swarm's write-time chunk replication with signed receipts.
- [[retrieval-protocol]] — Swarm's forward/backward, anonymity-preserving chunk retrieval.
- [[chainsync]] — Filecoin's blockchain sync FSM (BlockSync + GossipSub + BV0–BV8 validation).
- [[graphsync-data-transfer]] — Filecoin/IPFS bulk DAG transfer (GraphSync vs Bitswap, voucher-bound).
- [[storage-repair]] — Storj's satellite-orchestrated audit → reconstruct → redistribute repair.

### Networking & incentives
- [[kademlia]] — XOR/proximity-order DHT overlay with logarithmic routing.
- [[distributed-hash-table]] — key→value overlay; stores provider records (IPFS) or content (Swarm).
- [[proximity-order]] — leading-bit match between 256-bit addresses; governs chunk placement.
- [[postage-stamps]] — Swarm prepaid, decaying proof-of-payment / storage rent.
- [[swap-protocol]] — Swarm tit-for-tat bandwidth accounting via off-chain cheques.
- [[blockweave]] — Arweave's graph-structured chain linking previous + recall block.
- [[storage-endowment]] — Arweave pool funding perpetual storage from one-time fees.
- [[permanent-storage]] — pay-once-store-forever model (Arweave) vs rent/contract storage.
- [[reputation-system]] — distributed trust scoring of nodes (Storj's staking substitute).
- [[slashing]] — forfeiture of collateral as the penalty for faults/dishonesty.
- [[mechanism-design]] — engineering incentives toward DSIC honest behavior.
- [[coordination-avoidance]] — Storj v3 principle: no global consensus on the hot path.

## Entities
_Tools, libraries, frameworks, systems, languages, people._

### Networks & systems
- [[storj]] — S3-compatible decentralized cloud; erasure coding + satellites; token STORJ.
- [[filecoin]] — storage network + blockchain; consensus from provable storage; token FIL.
- [[arweave]] — permanent storage via blockweave + Proof of Access; token AR.
- [[swarm]] — P2P storage/comms for Web3; chunks + postage + SWAP; token BZZ.
- [[walrus]] — self-healing storage + data availability on Sui; 2-D Red Stuff coding.
- [[ipfs]] — content-addressing + P2P transport substrate (no persistence guarantee).
- [[ceph]] — distributed storage with erasure-coded pools (hosts Clay codes).
- [[hdfs]] — Hadoop FS with native Reed-Solomon + LRC erasure coding.

### Libraries, protocols & standards
- [[ipld]] — InterPlanetary Linked Data; codecs for content-addressed Merkle DAGs.
- [[libp2p]] — modular P2P networking stack (DHT, GossipSub) used by Filecoin & Swarm.
- [[multiformats]] — self-describing value formats (multihash/multicodec/multibase/CID).
- [[drand]] — distributed, verifiable, unbiasable randomness beacon (Filecoin).
- [[ens]] — Ethereum Name Service; maps names to Swarm references.
- [[bee-client]] — Bee: the reference Swarm client.
- [[lotus]] — reference Filecoin implementation (Go).

### Chains
- [[ethereum]] — smart-contract chain hosting Swarm incentives and ENS.
- [[gnosis-chain]] — EVM chain hosting Swarm's postage/redistribution contracts.
- [[sui]] — chain Walrus uses for blob lifecycle and Point of Availability.

### Roles & orgs
- [[satellite]] — Storj v3 trusted-but-blind coordinator (metadata, audits, payments).
- [[storage-node]] — Storj v3 peer storing erasure-coded pieces for pay.
- [[uplink]] — Storj v3 client (encryption, erasure coding, direct transfer).
- [[protocol-labs]] — org behind Filecoin, IPFS, libp2p, IPLD, multiformats.
- [[storj-labs]] — builds/operates the Storj network and approved satellites.
- [[viktor-tron]] — author/lead architect of Swarm.

### Tokens
- [[fil-token]] — Filecoin's currency; fuses payment, collateral, and consensus weight.
- [[bzz-token]] — Swarm's token for storage (postage) and bandwidth (SWAP); on Gnosis Chain.
- [[storj-token]] — Storj's ERC-20 payment token (no consensus role).

## Sources
_One summary page per ingested source._
- [[what-is-raid]] — primer on RAID levels, parity, mirroring, striping (article).
- [[erasure-coding-vs-raid]] — erasure coding math and how RAID is a special case (article).
- [[object-storage-domain-knowledge]] — comparative notes across 5 decentralized object stores (note).
- [[storj-v2]] — Storj P2P cloud storage v2: Kademlia, contracts, audits, micropayments (paper).
- [[storj-v3]] — Storj v3 framework: satellites, erasure coding, S3 compatibility (paper).
- [[arweave-yellow-paper]] — Arweave: blockweave, Proof of Access, endowment economics (paper).
- [[arweave-2-9]] — Arweave 2.9: RandomXSquared entropy/packing redesign (paper).
- [[book-of-swarm]] — comprehensive Swarm design compendium (paper).
- [[swarm-whitepaper]] — Swarm vision: storage + comms for a self-sovereign society (paper).
- [[swarm-protocol-spec]] — Swarm wire-level protocol specification (docs).
- [[content-addressing-tutorial]] — ProtoSchool: content vs location addressing, hashing, CIDs (docs).
- [[merkle-dags-tutorial]] — ProtoSchool: DAGs, Merkle DAGs, dedup, chunking, IPLD (docs).
- [[anatomy-of-a-cid-tutorial]] — ProtoSchool: CID byte anatomy and multiformats (docs).
- [[filecoin-spec]] — the Filecoin Specification: VM, markets, sealing, PoRep/PoSt, consensus (docs).

## Questions
_Filed-back explorations from queries._
- _(none yet)_
