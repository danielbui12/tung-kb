---
title: Block Storage
type: concept
tags: [block-storage, storage-model]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge]
---
# Block Storage

**Definition:** A low-level storage model that exposes raw, fixed-size blocks — a virtual disk — onto which a client installs and manages its own filesystem.

## How it works
The storage layer presents an array of equally sized blocks addressed by offset, with no notion of files, objects, or metadata beyond the block index. The consumer (an OS or database) writes and overwrites individual blocks freely, supporting in-place edits and low-latency random access. A filesystem (ext4, NTFS, XFS) or a raw application like a database engine sits on top to impose structure. Cloud examples are AWS EBS and Azure Managed Disks, typically attached to a single compute instance like a physical disk.

## Why it matters / trade-offs
Block storage gives the most control and the best random-access latency, making it the substrate for boot volumes, databases, and transactional workloads. The costs are that it is usually attached to one host at a time, harder to scale horizontally than object storage, and pushes all higher-level structure (files, naming, integrity) onto the client. It contrasts with [[object-storage]] (immutable blobs by ID, no in-place edits) and [[file-storage]] (a shared POSIX hierarchy).

## Related
[[object-storage]] [[file-storage]]

## Sources
- [[object-storage-domain-knowledge]] — block vs object vs file storage model
