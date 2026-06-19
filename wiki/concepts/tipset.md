---
title: Tipset
type: concept
tags: [tipset, consensus, blockchain, expected-consensus, filecoin]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Tipset

**Definition:** A tipset is the set of all valid blocks produced at the same height that share the same parent set. The Filecoin chain is a chain of *tipsets*, not of individual blocks.

## How it works
Filecoin's [[expected-consensus]] elects multiple block producers per 30-second epoch via [[vrf]] sortition, so a single height can carry several valid blocks rather than exactly one. All blocks at a given height that agree on their parents are grouped into one tipset, and the next epoch's miners build on the whole tipset. Chain selection picks the *heaviest* tipset chain (greatest cumulative weight), where weight rewards both total network power and the number of blocks per round — a GHOST-style rule, with VRF tickets used for tie-breaking. Soft finality is reached after roughly 900 epochs.

## Why it matters / trade-offs
Allowing many winners per round and folding them into a tipset increases throughput and smooths reward variance, while the heaviest-weight rule keeps the chain Sybil-invariant: producing more blocks requires more provable storage power, not just more identities. The cost is added complexity — clients reason about sets of blocks per height, and miners that produce no block in an epoch still advance the chain by building on the prior tipset.

## Related
[[expected-consensus]] [[vrf]] [[bls-signatures]] [[filecoin-vm]] [[filecoin]]

## Sources
- [[filecoin-spec]] — tipset as all valid blocks at a height sharing parents; heaviest-weight chain selection, ~900-epoch soft finality
