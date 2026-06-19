---
title: Proximity Order
type: concept
tags: [proximity-order, kademlia, xor-distance, swarm, routing]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec]
---
# Proximity Order

**Definition:** The number of matching leading bits between two 256-bit addresses — the discrete, logarithmic inverse of normalised XOR distance — used to bin peers and to decide which nodes are responsible for storing a chunk.

## How it works
For addresses `a` and `b`, the proximity order (PO) is the index of the first bit at which they differ; identical prefixes mean higher PO and therefore closer proximity, ranging 0–256. Because it counts a prefix rather than a magnitude, PO partitions the address space into nested bins: bin `d` holds all peers sharing exactly `d` leading bits. [[swarm]] uses PO twice. In routing, each [[kademlia]] forwarding hop must move a message to a peer of strictly higher PO to the destination, guaranteeing logarithmic progress. In storage, a node compares its PO to a [[chunk]]'s address against its **storage radius**: if PO meets or exceeds the radius the node falls inside the chunk's neighbourhood of responsibility and must store it. The same comparison governs push-sync (forward until PO ≥ radius) and pull-sync bins.

## Why it matters / trade-offs
PO is the single coordinate tying routing and storage together: it is cheap to compute (a leading-zero count on the XOR), monotone, and makes neighbourhoods self-defining. Its discreteness means distance comes in coarse bins rather than a continuum, so within one bin peer selection needs a secondary rule, and very deep bins are sparsely populated in small or clustered networks.

## Related
[[kademlia]] [[distributed-hash-table]] [[chunk]] [[swarm]] [[single-owner-chunk]]

## Sources
- [[book-of-swarm]] — PO definition, neighbourhood depth, responsibility by proximity
- [[swarm-protocol-spec]] — PO vs storage radius in push/pull-sync, bin structure
