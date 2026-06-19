---
title: Directed Acyclic Graph
type: concept
tags: [directed-acyclic-graph, data-structure, merkle-dag]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial]
---
# Directed Acyclic Graph

**Definition:** A graph whose edges have a one-way direction and which contains no cycles — no path leads from any node back to itself.

## How it works
A graph models objects as **nodes** and relationships as **edges**. An edge is *directed* when it carries a one-way sense (a directory contains a file, but not the reverse), which gives rise to parent/child, ancestor/descendant, root, leaf, and intermediate nodes. A graph is *acyclic* when following directed edges can never return to a starting node. A DAG combines both properties, making it a natural fit for hierarchical and dependency data such as file trees, build graphs, and version histories. Acyclicity guarantees traversals terminate — there are no infinite loops.

## Why it matters / trade-offs
The DAG is the structure that [[merkle-dag]]s specialize: when edges are expressed as embedded [[content-identifier]]s, acyclicity is enforced automatically (a node cannot reference its own not-yet-computed hash). DAGs are more expressive than trees because a node may have multiple parents, which is exactly what enables [[deduplication]] (one shared node referenced from many places). The trade-off versus a plain tree is more complex traversal and the need to avoid accidentally introducing cycles in mutable, non-content-addressed variants.

## Related
[[merkle-dag]], [[merkle-tree]], [[content-identifier]], [[content-addressed-data-structures]], [[deduplication]]

## Sources
- [[merkle-dags-tutorial]] — directed vs. acyclic definitions and DAGs as the basis for Merkle-DAGs
