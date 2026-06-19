---
title: Permanent Storage
type: concept
tags: [permanent-storage, arweave, data-durability, storage-incentive-mechanisms, decentralized-storage-networks]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-yellow-paper, object-storage-domain-knowledge]
---
# Permanent Storage

**Definition:** A storage model in which a user pays once and the data is kept forever, as opposed to contract- or rent-based models where storage persists only while recurring payment continues. Arweave is the canonical example.

## How it works
Arweave realises permanence by routing most of a one-time fee into a [[storage-endowment]] that subsidises miners whenever fees and inflation fall short of storage cost, and by making storage a prerequisite for mining through the [[blockweave]] and [[proof-of-access]]. This contrasts with the rent model of [[filecoin]], [[storj]] and [[swarm]]: Filecoin and Storj store data for the life of a paid deal, and Swarm's [[postage-stamps]] decay like rent so that under-funded chunks are deliberately garbage-collected and forgotten. The domain framing distinguishes pay-once-store-forever (Arweave) from recurring-rent and from Swarm's intentional forgetting. A subtle but important distinction Arweave itself draws is **data permanence** versus **network permanence**: the protocol promises the data survives, expecting the weave to eventually be "subsumed" into a successor system, much as Gopherspace was archived into the permaweb — not that this particular network runs unchanged forever.

## Why it matters / trade-offs
Permanence suits archival, provenance, and public-record use cases where deletion is undesirable and a single payment is operationally simpler than perpetual rent. The cost is economic fragility (the endowment must outlast centuries of cost assumptions) and the irreversibility problem: data cannot be removed, raising censorship-resistance benefits but also legal and right-to-erasure tensions. Rent-based forgetting (Swarm) trades guaranteed durability for lower long-run cost and the ability to let unwanted data lapse.

## Related
[[storage-endowment]] [[blockweave]] [[proof-of-storage]] [[postage-stamps]] [[arweave]] [[filecoin]] [[storj]] [[swarm]] [[decentralized-storage-networks]]

## Sources
- [[arweave-yellow-paper]] — pay-once permanence, endowment, data-vs-network permanence
- [[object-storage-domain-knowledge]] — permanence vs rent vs deliberate forgetting across DSNs
