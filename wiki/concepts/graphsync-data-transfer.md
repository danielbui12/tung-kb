---
title: GraphSync & The Data Transfer Protocol
type: concept
tags: [filecoin, data-transfer, graphsync, ipld, storage-market]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# GraphSync & The Data Transfer Protocol

**Definition:** Filecoin's Data Transfer Protocol (`/fil/datatransfer/1.0.0`) is a negotiation layer that ties a bulk DAG transfer to an agreed storage/retrieval deal via a voucher, then delegates the actual byte movement to a verifiable transport — by default GraphSync, a selector-based [[ipld]] DAG-sync protocol.

## How it works

Data Transfer abstracts the transport medium. It has two phases:

1. **Negotiation** — requestor and responder agree by validating a *voucher*.
2. **Transfer** — bytes move, default over GraphSync.

The client always initiates (clients are usually behind NAT). A **Push request** sends data to the other party (storage deals); a **Pull request** asks the other party to send data (retrieval deals). The request carries a **data-transfer voucher** (distinct from a [[payment-channels]] voucher) pointing at a pre-agreed deal so the responder can link and validate it.

**Roles**: *Requestor* (client) / *Responder* (provider); *Request Validator* (lives in the markets, inspects the voucher, accepts or disregards — the DT module never validates itself); *Transporter* (per-side, drives the underlying transport, tracks progress); *Subscriber* (listens to progress/completion events).

**Push flow** (storage):
```
1. requestor → push request + voucher  (over /fil/datatransfer/1.0.0)
2. responder validates voucher via Validator
3. responder → GraphSync request back to requestor
4. requestor recognizes the transfer, streams data
5. responder receives, emits progress, notifies listeners on completion
```
Push needs a *separate* round trip because GraphSync is request/response with no native push.

**Pull flow — single round trip** (retrieval):
```
1. requestor embeds the pull request + voucher inside a GraphSync request
2. responder forwards it to the DT module, validates via PullValidator
3. responder accepts → GraphSync response + DT-level acceptance
4. requestor receives data, emits progress, notifies on completion
```
Pull piggybacks on GraphSync's built-in **extensions**, so one round trip suffices. The provider MAY accept then *pause* to unseal the requested data, and MAY pause before unsealing to demand an unsealing payment (covering compute, deterring abusive clients).

**GraphSync vs Bitswap**: Bitswap exchanges individual blocks by CID — the requester must know each CID and round-trip per block. GraphSync issues one request carrying a root CID plus an **IPLD selector** describing which subgraph to traverse; the responder walks the [[merkle-dag]], and streams all matching blocks. This is far more efficient for large CAR/UnixFS DAGs (whole files/pieces) where the shape is known but individual CIDs are not. Filecoin moves Piece data this way; Bitswap remains a generic block-exchange fallback (e.g. in [[chainsync]]).

## State & parameters

Tracked per transfer (see go-data-transfer types/statuses):
- **Voucher + voucher type** — opaque blob the registered Validator decodes against deal state.
- **Transfer status** — e.g. Requested, Ongoing, Paused (unsealing / awaiting payment), Completed, Failed, Cancelled.
- **Channel ID** — (initiator, responder, transfer-id) identifying the transfer.
- **Selector + root CID** — the IPLD traversal spec and DAG root.
- **Progress / received bytes** — surfaced to Subscribers via events.

## Why it matters / trade-offs

Coupling transfer to a voucher means providers never serve or accept bytes for a deal they didn't agree to — a DoS and free-loading defense. Selector-based sync collapses many Bitswap round-trips into one negotiated traversal, and the pause/unseal hooks let providers recover unsealing costs in the [[retrieval-market]]. Because DT is transport-agnostic, GraphSync can be swapped for other verifiable transports (even offline) without changing market logic.

## Related
[[filecoin]] · [[ipld]] · [[ipfs]] · [[merkle-dag]] · [[content-identifier]] · [[storage-market]] · [[retrieval-market]] · [[payment-channels]] · [[decentralized-storage]] · [[chainsync]]

## Sources
- [[filecoin-spec]] — Data Transfer phases, push/pull flows, voucher/validator roles, GraphSync as default transport, unseal/payment pausing.
