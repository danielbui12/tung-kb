---
title: Ceph
type: entity
tags: [distributed-storage, erasure-coding, storage-system]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Ceph

**What it is:** A distributed storage system supporting erasure-coded pools with tunable parameters.

## Notes
Ceph offers [[erasure-coding|erasure-coded]] pools with a tunable k/m profile (its built-in default profile is 2+2, though most deployments override to 4+2 or wider) as a cheaper alternative to replication for durability against component failure. It is a representative datacenter (single-administrative-domain) distributed store rather than a trustless [[decentralized-storage-networks|decentralized network]] like [[filecoin]] or [[storj]].

Notably, Ceph hosts the MSR Clay [[regenerating-codes|regenerating-code]] plugin (Vajha et al.) since the Nautilus release (2019), which reduces repair *bandwidth* — reporting ~2.9x less repair traffic than plain RS at 1.25x storage overhead — addressing the k:1 read-amplification problem of classical erasure coding via bytes-per-helper rather than read-count. Compare [[hdfs]], which added native Reed-Solomon and LRC to cut 3x replication on cold data, and [[azure-storage]], which attacks the read-count axis instead via [[local-reconstruction-codes]].

## Sources
- [[erasure-coding-vs-raid]] — Ceph default 2+2 profile, Clay regenerating codes since Nautilus
- [[erasure-coding-algorithms-comparison]] — Clay ~2.9x repair-traffic reduction at 1.25x overhead figure
