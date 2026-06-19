---
title: Deduplication
type: concept
tags: [deduplication, content-addressing, chunking, storage-efficiency]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [merkle-dags-tutorial]
---
# Deduplication

**Definition:** Storing identical data only once and referencing it from every place it is needed, rather than copying it repeatedly.

## How it works
Under [[content-addressing]], identical bytes always produce the identical [[content-identifier]], so redundant data is naturally recognizable. In a [[merkle-dag]], a repeated section is encoded as a shared link (the same child CID embedded in multiple parents) instead of duplicated content. File **versioning** exploits this directly: a new version reuses the CIDs of all unchanged nodes and only adds nodes for what changed — the mechanism [[git]] uses for source control. [[Chunking]] increases granularity: splitting files into many small content-addressed pieces lets deduplication work *across* similar files, not just across whole identical files, since unchanged chunks are shared even when surrounding bytes differ.

## Why it matters / trade-offs
Deduplication cuts storage and bandwidth dramatically — common assets (shared libraries, web frameworks, repeated dataset regions) are stored and transferred once. It is a core efficiency property of content-addressed systems and underpins cheap versioning. The trade-offs are bookkeeping overhead (tracking references so shared nodes are not deleted while still referenced) and chunk-boundary selection: poor chunking can prevent matches, so content-defined chunking is often preferred over fixed-size splits.

## Related
[[content-addressing]], [[content-identifier]], [[merkle-dag]], [[chunking]], [[git]]

## Sources
- [[merkle-dags-tutorial]] — dedup via shared links, versioning (Git), and chunking-enabled cross-file dedup
