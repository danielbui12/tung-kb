---
title: Fountain Codes (LT / Raptor / RaptorQ)
type: concept
tags: [fountain-codes, raptorq, rateless, erasure-coding, broadcast]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Fountain Codes (LT / Raptor / RaptorQ)

**Definition:** "Rateless" erasure codes that drop the fixed-`n` assumption entirely — a sender emits a potentially unbounded stream of encoded symbols, and a receiver decodes once *any* sufficient subset has arrived, regardless of which specific symbols were received or lost.

## How it works
LT (Luby Transform) codes were the first practical rateless construction: each output symbol XORs a random-degree subset of source symbols, chosen by a carefully tuned degree distribution, and the receiver decodes via belief propagation as symbols accumulate. Raptor and RaptorQ (RFC 6330) precode the source with an outer block code before an inner LT-like pass, which lets them decode from close to exactly `k` symbols using inactivation decoding — near-linear time and, in most cases, no meaningful overhead over the theoretical minimum. RaptorQ is systematic (source symbols recoverable verbatim in the typical case) and supports up to 56,403 source symbols per RFC 6330.

## Why it matters / trade-offs
Fountain codes fit channels where the receiver set or loss pattern is unknown or unbounded in advance — broadcast/multicast (RaptorQ in ATSC 3.0, 3GPP MBMS), satellite, and lossy P2P transport — because the sender never needs to know how many symbols were lost or which ones; it just keeps emitting. The cost is giving up the hard [[mds-code|MDS]] guarantee: LT alone carries real overhead and is rarely used unmodified, and even RaptorQ is only near-MDS in practice (it rarely, not never, needs a few extra symbols beyond `k`). RaptorQ's inactivation decoding is also substantially more complex to implement correctly than [[reed-solomon]]'s matrix algebra. Despite looking like a natural fit for churning, unpredictable P2P storage networks, both [[storj]] and [[walrus]] chose Reed-Solomon in production over fountain codes — the Walrus v1 paper proposed RaptorQ, but the shipped protocol uses RS. This suggests that once a storage "channel" is provisioned well enough to fix `n` in advance, RS's MDS optimality and mature tooling outweigh fountain codes' rateless convenience; that inference rests on two examples, not a settled law.

## Related
[[reed-solomon]] [[mds-code]] [[erasure-coding]] [[ldpc]] [[storj]] [[walrus]]

## Sources
- [[erasure-coding-vs-raid]] — LT/Raptor mechanics, non-MDS status, fountain-vs-RS open question for decentralized storage
- [[erasure-coding-algorithms-comparison]] — RaptorQ near-MDS-in-practice claim (RFC 6330), ATSC 3.0/3GPP MBMS use, storage-vs-transmission split, Storj/Walrus choosing RS over fountain codes
