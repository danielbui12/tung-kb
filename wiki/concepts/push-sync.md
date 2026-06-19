---
title: Push-Sync
type: concept
tags: [swarm, push-sync, syncing, write-replication, receipts]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-protocol-spec, book-of-swarm]
---
# Push-Sync

**Definition:** The [[swarm]] protocol that delivers a freshly uploaded [[chunk]] to its destination neighbourhood by relaying it through ever-closer peers until it lands on a storer whose [[proximity-order]] to the chunk meets the storage radius, which then returns a signed statement-of-custody receipt back along the same path.

## How it works
Push-sync is the upload-side mirror of the [[retrieval-protocol]]: the chunk is **forwarded** toward its address via the same kind of route a retrieve request would take, and a receipt is **backwarded** to the uploader. A chunk has reached its neighbourhood when PO(peer, chunk) ≥ the peer's storage radius.

Stream: `/swarm/pushsync/1.3.0/pushsync`, varint-delimited protobuf.

```protobuf
syntax = "proto3";
package pushsync;

message Delivery {
  bytes Address = 1;
  bytes Data    = 2;
  bytes Stamp   = 3;   // postage stamp; one push per stamp
}

message Receipt {
  bytes Address   = 1;
  bytes Signature = 2;  // storer's statement of custody
  bytes Nonce     = 3;
  string Err      = 4;  // non-empty => failure
}
```

Three roles: **originator** (first to see the chunk), **forwarder** (moves it closer), **storer** (persists it and replicates to neighbourhood peers).

Pushing side: open a stream, write `Delivery{Address,Data,Stamp}`, await `Receipt`. On a non-empty `Err`, push to the next-best peer.

Storer side: on receiving `Delivery`, if it concludes it is responsible (within depth), persist the chunk, then return a signed `Receipt`. Wait for the pusher to close before closing.

Storage decisions:
- An **origin** peer does *not* store initially, so the chunk is always forwarded into the network; only if no peer can be found may the origin store it.
- A **non-origin** peer stores if the chunk is within depth; if not within depth it may still store if it is the closest peer.
- **Multiplexer:** when proximity ≥ storage radius, push in parallel to the neighbourhood (max **2 peers** at a time) to seed redundant replicas.

## State & parameters
- **Pending-request table:** forwarders remember which peer sent a chunk so receipts can be backwarded; records expire after a short window.
- **skipList:** each pushed (chunk,peer) pair is added to a skiplist to avoid retrying the same pair; **5 min** timeout before retry. Pairs are added even on *success*, because the origin may later judge a receipt too shallow and treat it as failed.
- **Shallow receipts:** if PO(storer, chunk) is outside the storage radius the receipt is shallow → the chunk is re-pushed eventually.
- **Retries:** an *origin* retries the same peer multiple times; a *forwarder* gives up after the first failure. Exhausting retries returns `ErrNoPush` and cancels the context (stopping all goroutines). Overdraft errors de-prioritize the upstream.
- **Multi-stamp:** a chunk stamped with multiple [[postage-stamps]] is pushed once per stamp; receivers persist each accordingly.

## Why it matters / trade-offs
Push-sync provides **write replication and the retrievability guarantee**: tracking receipts per chunk lets an uploader confirm the whole upload is universally retrievable before publishing its address (the backend for an upload progress bar). Forwarding keeps uploads **anonymous**. Invalid chunks in a response → blocklisting; excess unsolicited receipts → disconnection. It establishes the storer placement that the [[retrieval-protocol]] later assumes, and [[pull-sync]] maintains afterward.

## Related
[[retrieval-protocol]] · [[pull-sync]] · [[forwarding-kademlia]] · [[kademlia]] · [[proximity-order]] · [[chunk]] · [[postage-stamps]] · [[single-owner-chunk]] · [[swarm]]

## Sources
- [[swarm-protocol-spec]] — stream ID, `Delivery`/`Receipt` schemas, originator/forwarder/storer roles, storage-decision rules, skipList + 5-min timeout, shallow receipts, multiplexer parallel push.
- [[book-of-swarm]] — rationale: statement-of-custody receipts, upload progress/retrievability guarantee, anonymous uploads, unsolicited-receipt tolerance.
