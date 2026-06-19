---
title: Merkle DAGs
type: source
tags: [merkle-dag, content-addressing, data-structures, ipld, deduplication, distributed-web, directed-acyclic-graph]
created: 2026-06-19
author: ProtoSchool / Unknown
source_url: local: raw/Merkle-DAGs/
source_type: docs
ingested: 2026-06-19
---
# Merkle DAGs

**One-line:** A tutorial on how content identifiers (CIDs) are composed into Merkle DAGs to form verifiable, distributable, deduplicating data structures that underpin the decentralized web.

## Summary
A [[content-identifier]] acts as a fingerprint for a blob of data: it is primarily a [[cryptographic-hash]] of the data itself, giving a unique, succinct, location-independent name. Because the name derives from content rather than location, it can replace URLs as a link. But links do more than identify content — they are the structural glue of data. Lists, dictionaries, catalogs, linked lists, and file hierarchies all encode relationships between pieces of data via links. On the decentralized web, where peers have little or zero mutual trust, we need a way to link data structures together while preserving the ability to verify their integrity, so that a bad actor cannot silently insert or alter items mid-traversal.

The chosen abstraction is the graph: nodes are objects, edges are relations. A graph is *directed* when edges carry a one-way sense (a directory contains a file, but not vice versa), giving rise to parent/child, ancestor/descendant, root, leaf, and intermediate nodes. A graph is *acyclic* when no path leads from a node back to itself. A [[directed-acyclic-graph]] (DAG) is both, and is a natural fit for hierarchical data such as a file tree.

A [[merkle-dag]] is a DAG in which each node is encoded concretely and edges are expressed by embedding the CIDs of child nodes inside a parent node's data. Leaf nodes (e.g. files) are encoded as name plus content, then hashed to a CID. Intermediate nodes (e.g. directories) are encoded as a name plus a list of child CIDs, then hashed. Because a parent's data contains its children's CIDs, the parent's CID transitively depends on every descendant. Any change to a node propagates new CIDs up to all its ancestors, but never sideways into unrelated branches. This forces bottom-up construction (parents cannot be named until children's CIDs exist) and makes cycles impossible — a self-referential path cannot be constructed with cryptographic hashing, which is a security guarantee against infinite traversal loops.

Merkle DAGs inherit three key properties from CIDs. *Verifiability*: anyone retrieving data by content address can recompute the CID to confirm authenticity, giving permanence and tamper protection; a root CID uniquely identifies the entire DAG beneath it. Any node can serve as a root of its own subgraph, so DAGs are recursive — subgraphs can be retrieved, shared, and embedded into multiple larger DAGs simultaneously, since a node's CID depends on descendants, not ancestors. When data has no natural single root (e.g. an org chart with two top managers), one can add a synthetic root node, or treat it as multiple separate DAGs.

*Distributability*: anyone holding a DAG can serve it, children can be fetched in parallel from many providers, serving is not confined to central data centers, and any node's subgraph can be distributed independently. This solves the bottlenecks of location-addressed distribution of large datasets (single maintainer, monolithic archives, hard-to-find mirrors, serial downloads). *Deduplication*: redundant sections are encoded as shared links rather than copied. File versioning reuses unchanged nodes across DAG versions (the mechanism Git uses for source control); at larger scale, shared web themes/libraries (Bootstrap, common JS) could be downloaded once and reused. Granularity is flexible: files can be split via [[chunking]] into many nodes, enabling deduplication across similar files. Finally, Merkle DAGs are a foundational building block for Git, Ethereum, [[ipfs]], and Filecoin; the [[ipld]] (InterPlanetary Linked Data) project defines common Merkle-DAG data formats and schemas so disparate systems can reference and traverse each other's content-addressed structures.

## Key takeaways
- A Merkle DAG encodes edges by embedding child CIDs inside parent node data, making content addressing and the DAG structure inseparable.
- Embedding child CIDs makes cycles cryptographically impossible, so any content-addressed graph is necessarily acyclic.
- A node's CID depends on all its descendants; changes propagate upward to ancestors but never sideways to other branches, forcing bottom-up construction.
- A root CID verifiably identifies the entire DAG, extending CID guarantees (permanence, tamper-resistance) from data to data structure.
- Any node can be a root: subgraphs can be independently retrieved, shared, and embedded into multiple larger DAGs at once.
- Distributability lets anyone serve a DAG and fetch its nodes in parallel from many providers.
- Deduplication via shared links underpins versioning (as in Git) and avoids re-downloading common data; chunking enables fine-grained dedup across files.
- IPLD standardizes Merkle-DAG formats so Git, Ethereum, IPFS, and Filecoin can interoperate over content-addressed data.

## Concepts & entities
Wikilinks: [[merkle-dag]], [[directed-acyclic-graph]], [[merkle-tree]], [[content-addressing]], [[content-identifier]], [[cryptographic-hash]], [[deduplication]], [[chunking]], [[ipld]], [[ipfs]], [[data-structure]], [[graph-node]], [[graph-edge]], [[versioning]], [[git]], [[filecoin]], [[ethereum]].

## Notable claims / data
- Any graph representation that embeds child CIDs is necessarily a DAG — cryptographic hashing makes self-referential cycles impossible.
- A change to one node (e.g. photoshopping "tabby.png") changes its CID and all ancestor CIDs, but leaves sibling branches' CIDs untouched.
- Comparing two directory copies reduces to comparing their root CIDs — identical roots mean identical contents.
- Two versions of a directory can be stored without doubling storage, by reusing the unchanged shared nodes (green/orange/blue node distinction in the tutorial).
- Git uses Merkle DAGs to track source code changes; Ethereum, IPFS, and Filecoin also build on Merkle DAGs.
- IPLD (https://ipld.io) defines formal schemas for Merkle-DAG-based formats, enabling cross-system references (e.g. a Filecoin deal referencing an IPFS blob, or a smart contract referencing a Git commit).

## Questions raised
- What concrete serialization/codecs does IPLD specify for nodes, and how do they affect a node's CID?
- How is chunking boundary selection done in practice (fixed-size vs. content-defined chunking) to maximize deduplication?
- How are multi-root datasets (no natural single root) handled in real systems — synthetic roots vs. multiple DAGs?
- What are the trade-offs of node granularity (whole-file vs. chunked) on retrieval performance and dedup efficiency?
- How do different projects (Git vs. Ethereum vs. IPFS) differ in their concrete Merkle DAG node formats despite the shared abstraction?
