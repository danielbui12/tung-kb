---
title: Merkle DAG
type: concept
tags: [merkle-dag, content-addressing, directed-acyclic-graph, deduplication, ipld]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial, filecoin-spec, book-of-swarm]
---
# Merkle DAG

**Definition:** A [[directed-acyclic-graph]] in which each node embeds the [[content-identifier]]s of its child nodes, so a node's own CID transitively depends on every descendant.

## How it works
Leaf nodes (e.g. files) are encoded as name plus content, then hashed to a CID. Intermediate nodes (e.g. directories) are encoded as a name plus a list of child CIDs, then hashed. Because a parent's bytes contain its children's CIDs, the parent's CID depends on the whole subtree beneath it. This forces **bottom-up construction** (a parent cannot be named until its children's CIDs exist) and makes **cycles cryptographically impossible** — you cannot embed a node's CID inside itself. A change to one node propagates new CIDs upward to all ancestors but never sideways to unrelated branches, so a single root CID verifiably identifies the entire DAG. Any node can serve as the root of its own subgraph, making DAGs recursive: subgraphs can be retrieved, shared, and embedded into multiple larger DAGs at once.

## Why it matters / trade-offs
Merkle DAGs extend [[content-addressing]]'s integrity guarantees from individual blobs to whole data structures, and are the basis of [[ipld]], [[ipfs]], [[filecoin]] state, [[git]], and [[ethereum]]. They enable verifiability, parallel distributable retrieval, versioning, and [[deduplication]] via shared links (aided by [[chunking]]). The trade-off is immutability and write amplification: any edit re-hashes the entire path to the root.

## Related
[[merkle-tree]], [[directed-acyclic-graph]], [[content-identifier]], [[content-addressing]], [[deduplication]], [[chunking]], [[content-addressed-data-structures]], [[ipld]], [[ipfs]], [[git]]

## Sources
- [[merkle-dags-tutorial]] — node encoding, CID embedding, acyclicity, verifiability/distributability/dedup
- [[filecoin-spec]] — Merkle-DAG/IPLD as Filecoin state model
- [[book-of-swarm]] — content-addressed graph structures in Swarm
