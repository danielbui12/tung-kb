---
title: Content-Addressed Data Structures
type: topic
tags: [content-addressing, merkle-dag, multiformats, data-structures]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [content-addressing-tutorial, anatomy-of-a-cid-tutorial, merkle-dags-tutorial, filecoin-spec, book-of-swarm]
---
# Content-Addressed Data Structures

**Thesis / current understanding:** The shared substrate beneath every decentralized storage network is a small stack of primitives that name data by *what it is* ([[content-addressing]]) rather than *where it lives* ([[location-addressing]]). A [[cryptographic-hash]] gives each blob a unique, verifiable fingerprint; that fingerprint, wrapped in self-describing metadata, becomes a [[content-identifier]] (CID); CIDs embedded inside other blobs form a [[merkle-dag]]. This stack delivers integrity, deduplication, and location-independent retrieval — the properties that make permissionless storage possible at all.

## Landscape

The layered primitives, bottom to top:

1. **[[cryptographic-hash]]** — deterministic, one-way, collision-resistant digest. Identical bytes → identical hash; any change → a different hash. The root of all integrity here.
2. **[[content-identifier]] (CID)** — a self-describing label built from [[multiformats]] components: a **multihash** (hash-algorithm id + length + digest), a **multicodec** (data encoding, e.g. `dag-pb`), and a **multibase** (string encoding). [[anatomy-of-a-cid-tutorial|CIDv0]] is an implicit base58btc `Qm…` multihash; CIDv1 makes version/codec/base explicit.
3. **[[merkle-dag]]** — a [[directed-acyclic-graph]] whose edges *are* child CIDs embedded in parent nodes, making the whole structure self-verifying and acyclic. Generalizes the [[merkle-tree]]. Powers [[chunking]], [[deduplication]], and versioning (the Git model).

## How the systems use it

- [[ipfs]] / [[ipld]] define the canonical CID + Merkle-DAG model (UnixFS file trees, dag-pb/dag-cbor codecs). [[multiformats]] originated here and now spans [[ipfs]], [[ipld]], [[libp2p]], and [[filecoin]].
- [[filecoin]] is built directly on this stack: client files are UnixFS Merkle DAGs, Pieces are CAR-serialized IPLD DAGs, and the VM [[actor-model|State Tree]] is DAG-CBOR.
- [[swarm]] uses a *different* content-addressing scheme: fixed-size 4 KB [[chunk]]s addressed by a [[binary-merkle-tree]] (Keccak256) rather than multiformats CIDs, plus [[single-owner-chunk]]s for mutable data.
- [[arweave]] and [[storj]] use Merkle trees for proofs and integrity but key data differently (transaction data roots; HMAC-derived piece IDs).

## Open tensions / contradictions
- **CIDs vs fixed-size chunk addresses.** The IPFS/multiformats CID is algorithm-agnostic and self-describing but variable-shape; Swarm's fixed Keccak256 BMT address is simpler and uniform but locks in one hash and one chunk size. Interop between the two ecosystems is non-trivial.
- **Immutability vs mutable references.** Content addressing is inherently immutable; mutable data needs an overlay — Swarm [[single-owner-chunk]]s/feeds, IPNS, ENS-style naming. Each adds a trust or signature assumption.

## Open questions
- How do [[single-owner-chunk]]-style mutable pointers compare to IPNS for guarantees and latency?

## Sources
- [[content-addressing-tutorial]] — content vs location addressing, hashing, CID basics
- [[anatomy-of-a-cid-tutorial]] — CID byte anatomy, multiformats, CIDv0 vs CIDv1
- [[merkle-dags-tutorial]] — DAGs, Merkle DAGs, dedup, chunking, IPLD
- [[filecoin-spec]] — production use of the IPLD/CID/Merkle-DAG stack
- [[book-of-swarm]] — the alternative BMT/chunk addressing scheme
