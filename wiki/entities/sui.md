---
title: Sui
type: entity
tags: [blockchain, walrus, data-availability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge]
---
# Sui

**What it is:** The blockchain that Walrus uses for blob lifecycle, committee management, and Point of Availability.

## Notes
Sui is the on-chain coordination layer for [[walrus]]: it tracks the blob lifecycle, manages the BFT storage committee (n = 3f+1, n_shards=1000, quorum 2f+1=667), and records the on-chain Point of Availability that attests a blob is durably stored. This anchors Walrus's data-availability guarantees in consensus while the heavy [[erasure-coding|erasure-coded]] data lives off-chain on storage nodes.

Sui is to [[walrus]] what [[gnosis-chain]] is to [[swarm]] and [[ethereum]] is partly to [[storj]] — the host chain for the storage network's economic and availability state, contrasting with [[filecoin]]'s self-contained purpose-built chain.

## Sources
- [[object-storage-domain-knowledge]] — Sui committee, blob lifecycle, Point of Availability for Walrus
