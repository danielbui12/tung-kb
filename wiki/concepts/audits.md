---
title: Audits
type: concept
tags: [audits, storj, merkle-tree, erasure-coding, proof-of-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v2, storj-v3]
---
# Audits

**Definition:** Audits are probabilistic spot-checks that enforce data durability in [[storj]]: the verifier requests a random stripe or shard from a host and confirms it is held intact using a Merkle proof or erasure-code reconstruction (Berlekamp–Welch).

## How it works
Rather than verifying an entire shard every time, an auditor samples. In Storj v2 the owner issues a random salt and the host returns a compact [[merkle-tree]] proof of `log2(|l|)+1` hashes ([[proof-of-retrievability]]); partial audits hash only a byte subset, accepting a tiny false-positive rate for far lower I/O. In Storj v3 the network uses [[erasure-coding]]: a satellite downloads a random stripe across the nodes holding pieces of a segment and runs Berlekamp–Welch error detection — if enough returned pieces are consistent, the segment decodes and the contributing nodes pass; nodes that fail repeatedly lose reputation and are disqualified. Audit results feed reputation, which drives repair and node selection.

## Why it matters / trade-offs
Sampling makes durability enforcement cheap at scale: a host can be policed without rereading its whole store, and repeated independent audits drive cheating-detection probability toward certainty. Audits underpin the economic loop — they justify payment and trigger repair before redundancy is lost. The limits mirror those of any [[proof-of-storage]] scheme: a probabilistic audit can miss a single dropped byte, passing an audit does not guarantee a host will serve real reads, and the audit authority (the satellite in v3) is a centralization point relative to v2's owner-driven model.

## Related
[[proof-of-retrievability]], [[merkle-tree]], [[erasure-coding]], [[proof-of-storage]], [[redistribution-game]], [[slashing]], [[storj]], [[storage-incentive-mechanisms]]

## Sources
- [[storj-v2]] — Merkle-proof and partial audits, false-positive bounds
- [[storj-v3]] — erasure-code stripe audits, Berlekamp–Welch, reputation/repair
