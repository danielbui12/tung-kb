---
title: Decentralized Storage
type: concept
tags: [decentralized-storage, distributed-systems, blockchain]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [object-storage-domain-knowledge, storj-v3, filecoin-spec, arweave-yellow-paper, book-of-swarm]
---
# Decentralized Storage

**Definition:** Storing data across a permissionless network of independently-operated, untrusted nodes, coordinated by cryptography and economic incentives rather than a single trusted operator.

## How it works
A decentralized storage network (DSN) replaces the trusted datacenter with three replacements: **[[content-addressing]]** so any node's bytes are self-verifying; a **durability mechanism** ([[replication]] or [[erasure-coding]] + repair) so data survives node churn; and a **[[proof-of-storage]] + incentive** layer so nodes are paid to keep data and punished (via [[slashing]] or disqualification) if they don't. Clients typically encrypt data first ([[client-side-encryption]]), so nodes store ciphertext they cannot read.

## Why it matters / trade-offs
Versus centralized cloud storage, a DSN removes single points of failure/control and can be cheaper by using spare capacity, but pays for it in coordination cost, retrieval latency, and the hard problem of proving storage without a trusted auditor. [[ipfs]] provides the addressing/transport substrate but *no* persistence guarantee; the incentive layer is what distinguishes a real DSN.

## Related
For the head-to-head comparison of concrete networks, see the topic [[decentralized-storage-networks]].
[[object-storage]], [[proof-of-storage]], [[erasure-coding]], [[content-addressing]], [[storage-incentive-mechanisms]], [[storj]], [[filecoin]], [[arweave]], [[swarm]]

## Sources
- [[object-storage-domain-knowledge]] — the DSN definition and comparative framing
- [[storj-v3]], [[filecoin-spec]], [[arweave-yellow-paper]], [[book-of-swarm]] — concrete instances
