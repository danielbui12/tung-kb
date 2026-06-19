---
title: Object Storage
type: concept
tags: [object-storage, storage-model, erasure-coding]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge]
---
# Object Storage

**Definition:** A storage model that manages data as a flat namespace of immutable objects — each a blob of bytes plus metadata and a unique ID — accessed over HTTP, S3-style APIs.

## How it works
An object bundles its payload, arbitrary user metadata, and an identifier (often a content hash or path key) into one unit retrieved by ID rather than by file path or block offset. The namespace is flat — no true directory tree, though prefixes simulate folders. Objects are immutable: there are no in-place partial writes; updating means replacing the whole object. This immutability and the lack of POSIX semantics let object stores scale horizontally to exabytes and distribute objects freely across nodes. It contrasts with [[block-storage]] (raw fixed-size blocks presented as a virtual disk) and [[file-storage]] (a hierarchical POSIX filesystem).

## Why it matters / trade-offs
Object storage is the flagship use case for [[erasure-coding]]: large immutable objects with relaxed latency requirements map perfectly onto split-into-fragments redundancy, and content-addressed IDs make integrity self-certifying. It powers cloud stores (AWS S3, Azure Blob) and every decentralized storage network. The trade-off is that it is unsuited to workloads needing in-place edits, low-latency random access, or filesystem semantics — those belong on block or file storage.

## Related
[[block-storage]] [[file-storage]] [[erasure-coding]] [[content-addressing]] [[decentralized-storage]]

## Sources
- [[object-storage-domain-knowledge]] — object vs block vs file model, role in DSNs
