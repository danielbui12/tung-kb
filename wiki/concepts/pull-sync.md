---
title: Pull-Sync
type: concept
tags: [swarm, pull-sync, syncing, eventual-consistency, interval-store]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-protocol-spec, book-of-swarm]
---
# Pull-Sync

**Definition:** A subscription-style [[swarm]] protocol that synchronizes [[chunk]]s between neighbourhood nodes — bootstrapping fresh nodes by filling storage within their storage radius and guaranteeing **eventual consistency** by gradually migrating chunks to their proper storer nodes as the topology changes under churn.

## How it works
Pull-sync is **node-centric** (fill a node's storage) rather than chunk-centric. When two nodes connect they sync **both ways** over distinct independent streams; the consumer is the *downstream/client*, the provider the *upstream/server*. Per [[proximity-order]] bin, the upstream offers chunks ordered by storage timestamp (bin ID). To save bandwidth, addresses are offered for approval *before* data is sent (an offer/want round-trip).

Streams: `/swarm/pullsync/1.3.0/pullsync` and `/swarm/pullsync/1.3.0/cursors`, varint-delimited protobuf.

```protobuf
syntax = "proto3";
package pullsync;

message Syn {}
message Ack { repeated uint64 Cursors = 1; uint64 Epoch = 2; }
message Get { int32 Bin = 1; uint64 Start = 2; }
message Chunk { bytes Address = 1; bytes BatchID = 2; }
message Offer { uint64 Topmost = 1; repeated Chunk Chunks = 2; }
message Want { bytes BitVector = 1; }   // bitmap of desired offered chunks
message Delivery { bytes Address = 1; bytes Data = 2; bytes Stamp = 3; }
```

Flow:
1. **Cursors:** open the `cursors` stream, send `Syn` (with epoch timestamp); receive `Ack{Cursors[], Epoch}`; close.
2. **Get:** send `Get{Bin, Start}` for an interval.
3. **Offer:** upstream returns `Offer{Topmost, Chunks[]}` (addresses only).
4. **Want:** downstream inspects addresses (it knows from the address whether it already has the chunk — relies on [[content-addressing]] integrity) and replies `Want{BitVector}`.
5. **Delivery:** upstream streams `Delivery` messages for the wanted chunks (data + stamp), then closes.

Syncing runs in parallel with one worker per unique (peer, binID) pair. A fresh node bootstraps from the **lowest bin ID** (oldest chunks) upward, then transitions to **live syncing** once it reaches a bin ID with no chunk.

## State & parameters
- **Interval store:** persists `(start, end)` intervals with the last synced bin ID, tracking which ranges remain. `Next()` returns the first unfulfilled range (start/end **inclusive**; `end == 0` ⇒ no upper limit). It covers gaps between syncing sessions — on disconnect the downstream notes the last synced chunk's timestamp so it can resume.
- **Epoch timestamp:** persisted per peer on first cursor fetch; it is the **fingerprint of the peer's reserve**. If the reserve is wiped, the epoch changes → triggers a **reset of stored intervals** for that peer, forcing pull-sync from scratch.
- **Restart triggers:** Kademlia topology changes restart pull-sync; if nothing changes for a configurable window (**5 min** in Swarm) the scheduler restarts it automatically.
- **Range:** within the neighbourhood sync bins from storage radius up to **31**; for non-neighbours sync only the bin equivalent to your peer's — a safety net so lost chunks eventually migrate to their destination.
- **Multi-stamp:** the same chunk can arrive multiple times under different [[postage-stamps]]; the puller stores each accordingly.

## Why it matters / trade-offs
Pull-sync delivers **eventual consistency and redundancy restoration**: as neighbourhoods reorganize under churn, nodes re-replicate so every chunk in their area of responsibility remains redundantly retrievable. Each pulled chunk travels a route that is also a valid [[push-sync]] forwarding path, so syncing converges the network toward correct storer placement assumed by the [[retrieval-protocol]]. On upstream timeout/error/unexpected close, the peer is rescheduled for later; an invalid chunk → **immediate blocklisting** of the upstream.

## Related
[[push-sync]] · [[retrieval-protocol]] · [[forwarding-kademlia]] · [[kademlia]] · [[proximity-order]] · [[chunk]] · [[postage-stamps]] · [[reserve-sampling]] · [[swarm]]

## Sources
- [[swarm-protocol-spec]] — stream IDs, `Syn`/`Ack`/`Get`/`Offer`/`Want`/`Delivery` schemas, cursor→get→offer→want→deliver flow, interval store semantics, epoch fingerprint, restart triggers, bin range.
- [[book-of-swarm]] — rationale: node-centric bidirectional syncing, offer/want bandwidth round-trip, eventual consistency & redundancy restoration under churn, value cutoff v.
