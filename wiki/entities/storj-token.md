---
title: STORJ Token
type: entity
tags: [token, storj, cryptoeconomics]
created: 2026-06-19
updated: 2026-06-19
status: stub
sources: [storj-v3, storj-v2]
---
# STORJ Token

**What it is:** An Ethereum ERC-20 token used as the default payment from [[satellite]]s to [[storage-node]]s on the [[storj]] network.

## Notes
STORJ is purely a payment instrument — it has **no** consensus or staking role, consistent with Storj's [[coordination-avoidance]] design (no blockchain on the hot path). Satellites pay nodes for at-rest storage and egress, settled periodically off-chain rather than per-operation on-chain. v3 dropped payment for ingress after Storj v2's [[storj-v2|Storjcoin]] micropayment-per-audit model invited delete-and-rehoard abuse. Contrast [[fil-token]] (also a consensus stake) and [[bzz-token]] (storage rent + bandwidth).

## Sources
- [[storj-v3]] — STORJ as off-chain settlement to nodes
- [[storj-v2]] — the earlier Storjcoin micropayment model
