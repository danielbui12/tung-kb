---
title: Repair Bandwidth
type: concept
tags: [repair-bandwidth, erasure-coding, regenerating-codes, data-durability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [erasure-coding-vs-raid]
---
# Repair Bandwidth

**Definition:** The network and disk I/O required to rebuild a lost fragment in a redundant storage system — the dominant ongoing cost of erasure-coded storage at scale.

## How it works
Classical [[erasure-coding]] (Reed-Solomon) must read `k` surviving fragments to reconstruct a single lost one — a **k:1 read amplification** — because recovery inverts a `k x k` submatrix that needs `k` inputs. By contrast, [[replication]] repairs at 1:1 (copy one surviving replica). At data-center scale this amplification saturates network links and stretches rebuild windows, during which durability is degraded. Two code families attack the problem: **Local Reconstruction Codes (LRC)** add local parities so a single-fragment repair reads only a small group (Azure's `(12,2,2)` reads ~6 instead of 12), trading strict MDS optimality for cheaper common-case repair; **regenerating codes** (MSR/MBR, via network coding) cut how much data each helper node ships rather than how many it contacts, with MSR Clay codes in Ceph since Nautilus (2019).

## Why it matters / trade-offs
Repair bandwidth, not raw storage, often sets the practical limit on how wide `k` can grow: bigger `k` lowers storage overhead (`n/k`) but raises per-failure repair traffic. Faster repair also directly improves [[data-durability]], since fragments must be rebuilt before survivors decay below the threshold `k`. LRC pays extra parity space to save repair I/O; regenerating codes pay code complexity. The right operating point depends on failure rate, link cost, and access pattern.

## Related
[[erasure-coding]] [[reed-solomon]] [[replication]] [[data-durability]]

## Sources
- [[erasure-coding-vs-raid]] — k:1 amplification, LRC and MSR/MBR regenerating codes
