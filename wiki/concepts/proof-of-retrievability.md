---
title: Proof Of Retrievability
type: concept
tags: [proof-of-retrievability, storj, merkle-tree, audits, proof-of-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v2, storj-v3]
---
# Proof Of Retrievability

**Definition:** PoR is a challenge-response proof that a host still holds an intact copy of specific data, verified without re-downloading the data — in [[storj]] v2 implemented as Merkle-proof "heartbeats."

## How it works
The Storj v2 reference scheme is Merkle-tree based. The data owner generates random challenge salts, builds pre-leaves `H(s + d)` and leaves `H(H(s + d))`, stores the [[merkle-tree]] root and depth, and hands the leaves to the host ("farmer"). Periodically the owner issues a salt; the farmer must return a compact proof of exactly `log2(|l|)+1` hashes that the owner checks against the stored root. *Partial audits* hash only a subset of bytes, trading a tiny false-positive probability for far lower compute and I/O; consecutive audits drive that probability toward zero. Each audit can be paired with a micropayment, making retention continuously incentivized.

## Why it matters / trade-offs
PoR is the integrity-and-availability pillar of an untrusted storage market: it lets a payer detect a host that has silently dropped data, before relying on it. But it has structural limits the Storj papers flag openly. A proof of *retrievability to the owner* is not *publicly verifiable* — third parties cannot check it — and a host can pass an audit yet still refuse to serve real reads (the "hostage bytes" problem), while a "cheating owner" can withhold payment after a valid audit. These gaps motivate reputation systems and the more centralized satellite/audit model of [[storj]] v3, and overlap with [[audits]] as a durability tool.

## Related
[[audits]], [[merkle-tree]], [[proof-of-storage]], [[erasure-coding]], [[storj]], [[proof-of-storage-systems]], [[storage-incentive-mechanisms]]

## Sources
- [[storj-v2]] — Merkle-proof heartbeats, partial audits, false-positive bounds
- [[storj-v3]] — PoR limitations and the move to satellite-driven auditing
