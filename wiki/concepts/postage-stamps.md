---
title: Postage Stamps
type: concept
tags: [postage-stamps, swarm, storage-incentive-mechanisms, garbage-collection, bzz-token]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec, swarm-whitepaper]
---
# Postage Stamps

**Definition:** Swarm's prepaid, signed proof-of-payment attached to a chunk: a stamp asserts that storage rent has been bought on-chain for that chunk, signalling its importance and its priority against garbage collection.

## How it works
An uploader buys a **postage batch** on-chain, paying [[bzz-token]] into a smart contract that is drawn down as per-block storage rent via a price oracle. The batch's *depth* sets how many chunks it may stamp (2^depth slots), and a *uniformity depth* arranges those slots into power-of-2 buckets so that over-issuance or a targeted-DoS skew toward one address region becomes locally detectable and exponentially costly. The owner then issues a stamp per [[chunk]] — batch ID, bucket slot, timestamp and an owner signature — which travels with the chunk during push-sync. A storer node keeps a fixed-size **reserve** prioritised by remaining stamp value; as a batch's balance is spent down, its chunks' value decays like rent, and when value runs out or the reserve overflows the chunk is evicted from reserve to the LRU cache and eventually garbage-collected.

## Why it matters / trade-offs
Postage stamps are how Swarm pays for *storage* (as opposed to [[swap-protocol]], which pays for bandwidth) and how it decides what to forget: rent that lapses means data is dropped, the deliberate counterpart to Arweave's [[permanent-storage]]. The bucket/uniformity scheme curbs spam and skew without per-chunk on-chain transactions. The trade-off is that persistence is conditional — unpaid or under-funded content disappears — and uploaders must estimate batch depth and balance up front. Collected rent is later redistributed to storers via the [[redistribution-game]].

## Related
[[swap-protocol]] [[redistribution-game]] [[chunk]] [[bzz-token]] [[permanent-storage]] [[storage-incentive-mechanisms]] [[swarm]]

## Sources
- [[book-of-swarm]] — batch depth/uniformity, reserve rules, rent and GC priority
- [[swarm-protocol-spec]] — stamp on the wire, reserve-to-cache eviction
- [[swarm-whitepaper]] — stamps as prepaid storage signal, value decay
