---
title: Reputation System
type: concept
tags: [reputation-system, storj, trust, sybil-resistance, decentralized-storage-networks]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v2, storj-v3]
---
# Reputation System

**Definition:** A distributed mechanism for scoring the trustworthiness of storage nodes — their availability, honesty and performance — so the network can prefer reliable nodes and exclude faulty or dishonest ones without a central authority.

## How it works
In [[storj]] v2 a robust distributed reputation system was an explicit *open problem*: the paper reviewed candidates (Eigentrust++, TrustDavis, identity-maintenance-cost schemes, Bitswap-style ledgers) but adopted none, because capturing bandwidth, latency and availability across untrusted peers without prohibitive consensus latency was unsolved. Storj v3 replaces the open question with a concrete, satellite-mediated system in four parts: (1) a **proof-of-work identity** — a node ID derived from a CA public key with a required number of trailing zero bits — that makes Sybil identities costly; (2) a **vetting period** in which new nodes receive only a small fraction of data and have part of their earnings held for ~9 months; (3) a **filtering / disqualification** system where too many failed [[proof-of-storage]] audits or uptime checks moves the node's data away, forfeits held funds and forces re-vetting (audit-DQ threshold ~0.96); and (4) a **preference system** using Power-of-Two-Choices load balancing to steer new data toward better-performing nodes.

## Why it matters / trade-offs
Reputation is what lets a permissionless network of untrusted nodes behave reliably: it turns observed behaviour into an economic signal, complementing [[slashing]] and audits. The v3 design is pragmatic but **per-satellite** — reputation is local to each coordinator at launch, only "global over time" — so it leans on the satellites as trusted-but-blind coordinators rather than achieving fully decentralised, shared trust scoring.

## Related
[[slashing]] [[proof-of-storage]] [[mechanism-design]] [[storj]] [[decentralized-storage-networks]]

## Sources
- [[storj-v2]] — reputation as an open problem; Eigentrust++/TrustDavis survey
- [[storj-v3]] — PoW identity, vetting, disqualification, preference system
