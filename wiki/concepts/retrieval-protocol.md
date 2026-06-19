---
title: Retrieval Protocol
type: concept
tags: [swarm, retrieval, forwarding-kademlia, anonymity, networking]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-protocol-spec, book-of-swarm]
---
# Retrieval Protocol

**Definition:** The protocol that fetches a [[chunk]] from the [[swarm]] network by its address, relaying a request through [[forwarding-kademlia]] hops toward the chunk's neighbourhood and returning the data back along the *same* path (backwarding), so that the requestor's identity is never disclosed.

## How it works
Retrieval uses **forwarding** for the request and **backwarding** for the response. The request is relayed node-to-node, each hop increasing the [[proximity-order]] to the target, until it reaches the storer node closest to the chunk address. The chunk is then passed back hop-by-hop along the exact reverse route. Because intermediate nodes only ever talk to their immediate request peer, no node learns the originator's address — this is the basis of **anonymous retrieval**.

Stream: `/swarm/retrieval/1.4.0/retrieval`, varint-delimited protobuf.

```protobuf
syntax = "proto3";
package retrieval;

message Request {
  bytes Addr = 1;        // chunk address being requested
}

message Delivery {
  bytes Data  = 1;       // chunk payload
  bytes Stamp = 2;       // associated postage stamp
  string Err  = 3;       // non-empty => failure
}
```

Requesting side: open a stream, send `Request{Addr}`, wait for `Delivery`. If `Err` is non-empty, close the stream and re-attempt with the next peer candidate.

Responding side, on receiving `Request`:
1. Look up the chunk in the local store.
2. If not found locally, forward the request deeper into the network (becoming a forwarder).
3. On obtaining the chunk from the network, optionally cache it (opportunistic caching) to serve repeat requests cheaply.
4. Return `Delivery` with the chunk data + stamp, or an unsuccessful `Delivery` on error. Wait for the requester to close its side before closing.

## State & parameters
- **Pending-request table:** each forwarder remembers which downstream peer asked for which chunk, so it can backward the delivery. Records are garbage-collected after a TTL; the originator attaches a time-to-live to bound the wait.
- **Peer roles:** *origin* (downloader, initiates and may retry the same peer multiple times), *forwarder* (relays request, *backwarder* on the return path — gives up after the **first** failure), *storer* (closest node, holds the chunk), *multiplexer* (very close to the neighbourhood — may issue several parallel/repeated fetches given higher hit probability).
- **Kademlia choice priority:** include self (full non-origin nodes) → reachable+healthy peers → reachable peers → all others.
- **No "not found":** there is no explicit negative response; only timeouts. A distant node cannot reliably prove absence, and the chunk may yet arrive via syncing or a late upload, so requests stay open until TTL.
- **Overdraft de-prioritization:** if upstream is in overdraft, lower its priority to allow [[swap-protocol]] balance replenishment.

## Why it matters / trade-offs
Retrieval provides **anonymous, censorship-resistant access**: backwarding leaks no requestor identity (unlike direct or routed delivery). It tolerates churn via **re-requesting** — if a path node drops, the payment commitment voids and the chunk is re-requested from another candidate. Unsolicited deliveries (post-expiry) are penalized and, above a small threshold, the peer is disconnected/blocklisted. Receiving an invalid chunk results in immediate blocklisting. It is the exact reverse of [[push-sync]] and relies on [[content-addressing]] for integrity checks.

## Related
[[push-sync]] · [[pull-sync]] · [[forwarding-kademlia]] · [[kademlia]] · [[proximity-order]] · [[chunk]] · [[postage-stamps]] · [[swap-protocol]] · [[swarm]]

## Sources
- [[swarm-protocol-spec]] — stream ID, `Request`/`Delivery` schemas, requesting/responding flow, retry rules, role behaviors (backwarder vs origin vs multiplexer).
- [[book-of-swarm]] — rationale: backwarding for anonymity, no "not found" semantics, opportunistic caching, re-requesting under churn, unsolicited-chunk DoS protection.
