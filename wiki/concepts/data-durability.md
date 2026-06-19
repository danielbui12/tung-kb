---
title: Data Durability
type: concept
tags: [data-durability, redundancy, audits, reliability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge, erasure-coding-vs-raid, storj-v3, raid]
---
# Data Durability

**Definition:** The probability that stored data survives hardware and node failures over a time window — often quoted in "nines" (e.g. eleven 9s = 99.999999999%).

## How it works
Durability is driven by a feedback loop of three mechanisms. **Redundancy** ([[replication]] or [[erasure-coding]]) sets how many simultaneous losses can be tolerated: Storj's RS 29-of-80 can lose 51 of 80 pieces and still reconstruct, yielding roughly eleven 9s. **Repair** continuously rebuilds lost fragments before the surviving count decays below the reconstruction threshold `k`; durability depends as much on repair *speed* relative to failure rate as on the code parameters. **Audits** (or proof-of-storage challenges) detect which fragments are actually missing or corrupt, converting silent loss into known erasures that repair can act on. Placement across independent failure domains keeps failures uncorrelated so they don't cluster within one stripe.

## Why it matters / trade-offs
High durability is the core promise of any storage system, but it is distinct from **availability** (can you read it *right now*) and from **backup** (durability offers no protection against accidental deletion, application bugs, or ransomware — those corrupt the redundant copies too). Pushing durability higher means more parity/copies (storage cost) and faster, more aggressive repair (network and [[repair-bandwidth]] cost). Without checksums and scrubbing, silent corruption can erode durability invisibly.

## Related
[[redundancy]] [[replication]] [[erasure-coding]] [[audits]] [[proof-of-storage]] [[repair-bandwidth]]

## Sources
- [[object-storage-domain-knowledge]] — durability across DSNs, proofs and repair
- [[erasure-coding-vs-raid]] — coding parameters vs fault tolerance
- [[storj-v3]] — eleven-9s from RS 29-of-80
- [[what-is-raid]] — durability vs backup distinction
