---
title: IPLD
type: entity
tags: [content-addressing, data-model, merkle-dag, codecs]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial, anatomy-of-a-cid-tutorial, filecoin-spec]
---
# IPLD

**What it is:** InterPlanetary Linked Data — a data model and set of codecs for content-addressed Merkle-DAGs.

## Notes
IPLD, from [[protocol-labs]], defines common formats and schemas for [[merkle-dag]] structures so disparate systems can reference and traverse each other's [[content-addressing|content-addressed]] data (e.g. a Filecoin deal referencing an IPFS blob, or a smart contract referencing a [[git]] commit). Each node embeds the [[content-identifier]]s (CIDs) of its children, making structure and content addressing inseparable.

Its codecs include `dag-pb` (the legacy [[ipfs]] UnixFS format, multicodec `0x70`) and `dag-cbor` (the structured format used for [[filecoin]]'s on-chain state tree and actor state), plus CAR (Content Addressable aRchive) files for packaging DAGs. The multicodec field of a CID always names an IPLD codec for IPFS content. IPLD is consumed by IPFS and Filecoin, and relies on [[multiformats]] for CIDs.

Compare [[swarm]], which uses its own BMT-hashed chunk model and Swarm hash tree rather than IPLD.

## Sources
- [[merkle-dags-tutorial]] — IPLD as the standard Merkle-DAG format layer
- [[anatomy-of-a-cid-tutorial]] — IPLD codecs in the CID multicodec field
- [[filecoin-spec]] — DAG-CBOR/CAR usage in Filecoin state and data pipeline
