---
title: Merkle Tree
type: concept
tags: [merkle-tree, cryptographic-hash, inclusion-proof, data-structure]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial, arweave-yellow-paper, storj-v2]
---
# Merkle Tree

**Definition:** A hash tree in which every leaf is the hash of a data block and every internal node is the hash of its children, so the single root hash commits to all the data beneath it.

## How it works
Data is split into blocks; each block is hashed to form the leaves. Pairs of hashes are concatenated and re-hashed to form parent nodes, repeating until one **root hash** remains. To prove a given block belongs to the tree, you supply the block plus the sibling hash at each level up to the root — an **inclusion proof** of size O(log n) rather than the whole dataset. The verifier recomputes the path and checks it matches the trusted root. Any modification to a block changes its leaf hash and every hash on the path to the root, making tampering detectable from the root alone.

## Why it matters / trade-offs
Merkle trees give compact, verifiable integrity over large datasets and are the ancestor of the [[merkle-dag]] (a tree generalized to a DAG with content-addressed edges). They appear throughout decentralized storage: [[arweave]] uses them in its blockweave and proofs, and [[storj]] used Merkle-tree audits over stored pieces. The cost is proof generation/storage overhead and the need to keep the tree consistent as data changes (which re-hashes a full root-to-leaf path).

## Related
[[merkle-dag]], [[cryptographic-hash]], [[binary-merkle-tree]], [[directed-acyclic-graph]], [[proof-of-storage]], [[arweave]], [[storj]]

## Sources
- [[merkle-dags-tutorial]] — Merkle tree as the ancestor of the Merkle-DAG
- [[arweave-yellow-paper]] — Merkle structures in blockweave and proofs
- [[storj-v2]] — Merkle-tree-based audits of stored data
