---
title: Local Reconstruction Codes (LRC)
type: concept
tags: [lrc, erasure-coding, repair-bandwidth, azure-storage]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Local Reconstruction Codes (LRC)

**Definition:** An erasure code that adds small "local" parity groups on top of a Reed-Solomon-style base, so the common case of a single lost fragment is repaired by reading only its nearby group instead of the full `k` fragments — at the cost of giving up strict [[mds-code|MDS]] optimality.

## How it works
LRC splits the `k` data fragments into groups (e.g. Azure's `(12,2,2)`: 12 data fragments split into two groups of 6), computes a small local parity for each group, and adds global parities across the whole stripe for multi-failure protection. A single lost fragment is reconstructed from its own local group plus its local parity — Azure's `(12,2,2)` reads ~6 fragments instead of the 12 a flat RS(12,4)-style code would require. Decoding is two-tiered: the local tier handles the common single-loss case cheaply; the global tier (spanning all groups) handles rarer multi-failure patterns, at higher read cost, similar to plain RS.

## Why it matters / trade-offs
LRC exists because [[repair-bandwidth]], not storage, is the real constraint at scale for the overwhelmingly common single-fragment failure. It pays for this with a small storage premium over MDS-optimal RS (Azure's `(12,2,2)` is 1.33x overhead, comparable to 12+4 RS but with roughly half the common-case repair traffic) and with worse *multi*-failure repair than pure RS, plus two-tier encode/decode complexity. It is a different axis of repair-cost reduction than [[regenerating-codes]]: LRC cuts how many fragments must be *read*; regenerating codes cut how many *bytes* each helper must *send*. Both are derivatives of [[reed-solomon]] rather than a wholesale replacement.

## Related
[[reed-solomon]] [[mds-code]] [[regenerating-codes]] [[repair-bandwidth]] [[erasure-coding]] [[azure-storage]]

## Sources
- [[erasure-coding-vs-raid]] — LRC mechanics, Azure `(12,2,2)` parameters, USENIX ATC'12 origin (~2012)
- [[erasure-coding-algorithms-comparison]] — LRC as one of two repair-cost-optimized derivatives of RS, comparison table placement
