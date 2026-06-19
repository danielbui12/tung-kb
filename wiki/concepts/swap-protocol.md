---
title: SWAP Protocol
type: concept
tags: [swap-protocol, swarm, bandwidth-incentives, payment-channels, off-chain]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-whitepaper, swarm-protocol-spec]
---
# SWAP Protocol

**Definition:** SWAP (Swarm Accounting Protocol) is Swarm's tit-for-tat bandwidth-accounting and settlement layer: peers track their relative service balance and, past a threshold, settle the debt off-chain with signed cheques redeemable at an on-chain chequebook contract.

## How it works
Each pair of connected peers keeps a running balance of bandwidth consumed in each direction. While balances stay within a payment threshold, no money moves — service is exchanged tit-for-tat. When a peer's debt crosses the threshold it sends a **cheque**: an off-chain signed commitment to pay a cumulative amount, redeemable in [[bzz-token]] at the issuer's **chequebook** smart contract. Cumulative totals and serial numbers prevent double-cashing. **Waivers** let opposing cheques offset to save gas, and a third party can cash a cheque on behalf of a node that holds no Ether, enabling zero-cash entry. Pricing is differential by [[proximity-order]]: delivering a [[chunk]] costs more the farther a peer is from the chunk's address, which makes free denial-of-service expensive and pushes prices toward uniform marginal cost. Thresholds are announced per-PO over the wire (the pricing protocol), and settlement runs over pseudosettle/SWAP streams.

## Why it matters / trade-offs
SWAP keeps Swarm cheap or free for light and patient users while letting heavy users pay for speed, and it rewards nodes for relaying and opportunistically caching popular content (a cached chunk served again is pure profit). Settling off-chain avoids a blockchain transaction per request, with on-chain settlement only at thresholds. The model is bilateral, so it secures *bandwidth*, not durable storage — that is handled separately by [[postage-stamps]] — and it relies on the chequebook contract being funded and on honest cumulative accounting.

## Related
[[postage-stamps]] [[proximity-order]] [[chunk]] [[bzz-token]] [[storage-incentive-mechanisms]] [[swarm]]

## Sources
- [[book-of-swarm]] — cheques, chequebook, waivers, differential proximity pricing
- [[swarm-whitepaper]] — SWAP bandwidth accounting and caching incentive
- [[swarm-protocol-spec]] — pricing announcements and pseudosettle wire messages
