---
title: Kademlia
type: concept
tags: [kademlia, distributed-hash-table, peer-to-peer, routing, overlay-network]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec, storj-v2, swarm-whitepaper]
---
# Kademlia

**Definition:** A peer-to-peer overlay and distributed-hash-table routing scheme that organises nodes in a 256-bit address space by XOR distance, giving logarithmic-hop lookups without global state.

## How it works
Each node holds a stable address in the same space as the keys it routes toward. Distance between two addresses is their XOR, and nodes bucket their peers by [[proximity-order]] (the count of matching leading bits, the discrete inverse of normalised XOR distance). A node keeps several peers in each bin so that any target can be reached by repeatedly hopping to a peer at least one bin closer, in O(log n) hops. Classic Kademlia (and [[storj]]) does *iterative* lookup: the originator queries peers and is told who to ask next, using the DHT only for routing and message transport, not for storing the payload. [[swarm]] instead uses a *forwarding* (recursive) variant: each node relays the request itself toward the destination, and the response travels back along the same path ("backwarding"), which hides the originator's identity and gives requestor anonymity. Saturated buckets (multiple peers per bin) absorb churn.

## Why it matters / trade-offs
Logarithmic routing makes the network scale to millions of nodes without any node knowing the whole topology. The forwarding variant trades some latency and per-hop work for built-in privacy and lets [[swarm]] place [[chunk]] content at the nodes closest to its address. Bucket saturation costs memory and bandwidth but is what keeps routing reliable under continuous node arrival and departure.

## Related
[[distributed-hash-table]] [[proximity-order]] [[chunk]] [[swarm]] [[storj]] [[decentralized-storage-networks]]

## Sources
- [[book-of-swarm]] — forwarding/backwarding Kademlia, proximity order, saturated buckets
- [[swarm-protocol-spec]] — hive discovery, PO vs storage radius on the wire
- [[storj-v2]] — Kademlia/S-Kademlia for routing only, signed-message identity
- [[swarm-whitepaper]] — logarithmic-hop overlay and DISC placement
