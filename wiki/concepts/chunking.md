---
title: Chunking
type: concept
tags: [chunking, content-addressing, deduplication, merkle-dag]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial, book-of-swarm]
---
# Chunking

**Definition:** Splitting a file into smaller, individually content-addressed pieces (DAG nodes or chunks) instead of treating it as one monolithic blob.

## How it works
A large file is divided into pieces, each hashed to its own [[content-identifier]] and stored as a leaf in a [[merkle-dag]]; an intermediate node lists the child CIDs to reassemble the file in order. Boundaries can be chosen by **fixed size** (simple, but a single inserted byte shifts every later boundary and breaks matches) or by **content-defined chunking** (boundaries set by a rolling hash over the data, so edits only disturb nearby chunks). In Swarm, content is split into fixed-size [[chunk]]s as the unit of storage and routing.

## Why it matters / trade-offs
Chunking unlocks three benefits. It makes [[deduplication]] fine-grained — unchanged chunks are shared across similar files and across versions. It enables **parallel transfer** — chunks can be fetched concurrently from many providers rather than serially from one host. And it gives **fixed-size storage units** that are easier to place, route ([[distributed-hash-table]]/[[kademlia]]), erasure-code, and prove. The trade-offs are per-chunk metadata overhead and the dedup-vs-overhead tension in choosing chunk size: smaller chunks dedup better but multiply node count and CID bookkeeping.

## Related
[[chunk]], [[merkle-dag]], [[content-identifier]], [[deduplication]], [[content-addressing]], [[erasure-coding]], [[distributed-hash-table]]

## Sources
- [[merkle-dags-tutorial]] — chunking files into DAG nodes for fine-grained dedup
- [[book-of-swarm]] — fixed-size chunks as Swarm's unit of storage and routing
