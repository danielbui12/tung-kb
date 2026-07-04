---
title: Azure Storage
type: entity
tags: [azure, erasure-coding, lrc, cloud-storage]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Azure Storage

**What it is:** Microsoft's cloud storage service, notable in this wiki for introducing production [[local-reconstruction-codes|LRC]] as an alternative to plain Reed-Solomon.

## Notes
Azure Storage's `(12,2,2)` LRC profile (12 data fragments split into two groups of 6, each with a local parity, plus 2 global parities) is the reference production deployment of [[local-reconstruction-codes]]: it is not [[mds-code|MDS]] and carries slightly more overhead than optimal RS(12,4) (1.33x), but single-fragment repair — the dominant failure mode — reads only ~6 fragments instead of 12. First described in the USENIX ATC'12 paper "Erasure Coding in Windows Azure Storage" (Microsoft Research, ~2012). Contrast [[ceph]], which pursues the opposite repair-cost axis (bandwidth-per-helper via MSR Clay [[regenerating-codes]]) rather than read-count.

## Sources
- [[erasure-coding-vs-raid]] — Azure LRC parameters, origin, overhead vs repair-savings figures
- [[erasure-coding-algorithms-comparison]] — LRC `(12,2,2)` repair-read comparison to plain RS
