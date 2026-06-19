---
title: File Storage
type: concept
tags: [file-storage, storage-model, posix]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge]
---
# File Storage

**Definition:** A storage model that organizes data as a hierarchy of directories and files with POSIX semantics, accessed over network filesystem protocols such as NFS or SMB.

## How it works
Data lives in named files arranged in nested directories, addressed by path (e.g. `/home/user/doc.txt`). Clients perform familiar POSIX operations — open, read, write, seek, append, rename, set permissions and timestamps — and the filesystem maintains the directory tree and metadata. Network file systems like NFS and SMB/CIFS export this hierarchy so multiple clients share the same tree concurrently, with the server arbitrating locking and consistency. It supports in-place partial writes, unlike object storage.

## Why it matters / trade-offs
File storage is the natural fit for shared documents, home directories, and legacy applications that expect a mounted filesystem and POSIX behavior. Its costs are scaling limits and metadata overhead: maintaining a consistent, lockable hierarchy across many clients is expensive, and the tree becomes a bottleneck at very large scale — which is why bulk and cloud-scale data migrate to [[object-storage]]. It contrasts with [[block-storage]] (raw blocks, no built-in filesystem) and [[object-storage]] (flat immutable blobs by ID).

## Related
[[object-storage]] [[block-storage]]

## Sources
- [[object-storage-domain-knowledge]] — file vs object vs block storage model
