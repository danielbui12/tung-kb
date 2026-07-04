---
title: Repair Bandwidth
type: concept
tags: [repair-bandwidth, erasure-coding, regenerating-codes, data-durability]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Repair Bandwidth

**Definition:** The network and disk I/O required to rebuild a lost fragment in a redundant storage system — the dominant ongoing cost of erasure-coded storage at scale.

## How it works
Classical [[erasure-coding]] (Reed-Solomon) must read `k` surviving fragments to reconstruct a single lost one — a **k:1 read amplification** — because recovery inverts a `k x k` submatrix that needs `k` inputs. By contrast, [[replication]] repairs at 1:1 (copy one surviving replica). At data-center scale this amplification saturates network links and stretches rebuild windows, during which durability is degraded. Two code families attack the problem along different axes: [[local-reconstruction-codes|LRC]] cuts how many fragments must be *read* (Azure's `(12,2,2)` reads ~6 instead of 12); [[regenerating-codes]] (MSR/MBR) cuts how many *bytes* each helper must *send* rather than how many it contacts (Ceph Clay: ~2.9x less traffic at 1.25x overhead).

## Why it matters / trade-offs
Repair bandwidth, not raw storage, often sets the practical limit on how wide `k` can grow: bigger `k` lowers storage overhead (`n/k`) but raises per-failure repair traffic. Faster repair also directly improves [[data-durability]], since fragments must be rebuilt before survivors decay below the threshold `k`. LRC pays extra parity space to save repair I/O; regenerating codes pay code complexity. The right operating point depends on failure rate, link cost, and access pattern — see [[local-reconstruction-codes]] and [[regenerating-codes]] for the mechanics of each.

## Related
[[erasure-coding]] [[reed-solomon]] [[local-reconstruction-codes]] [[regenerating-codes]] [[replication]] [[data-durability]]

## Sources
- [[erasure-coding-vs-raid]] — k:1 amplification, LRC and MSR/MBR regenerating codes
- [[erasure-coding-algorithms-comparison]] — LRC vs regenerating as distinct repair-cost axes (read-count vs bytes-per-helper)
