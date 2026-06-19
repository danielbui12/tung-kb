---
title: Storage Node
type: entity
tags: [storj, storage, audits, peer]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3]
---
# Storage Node

**What it is:** A Storj v3 peer that stores erasure-coded pieces for pay.

## Notes
A storage node in [[storj]] v3 stores [[reed-solomon|erasure-coded]] pieces and is paid for at-rest storage and egress (never ingress). Nodes register directly with [[satellite]]s they trust, advertising space, bandwidth, and wallet, and serve get/put/delete keyed by piece ID + Satellite ID. Node IDs use a proof-of-work scheme (trailing-zero hash of the CA public key) for Sybil resistance.

Nodes are subject to [[audits]] run by the Satellite; non-responsive nodes enter "containment mode," and too many failed audits or uptime checks lead to disqualification (data is repaired elsewhere and held funds forfeited). New nodes face a vetting period with a portion of earnings held ~9 months to penalize churn.

This is the untrusted-storage tier, analogous to [[filecoin]]'s storage miners (but without zk-SNARK [[sealing]]) and [[swarm]]'s storer nodes. Receives pieces from [[uplink]]s.

## Sources
- [[storj-v3]] — storage node role, payment model, audits, vetting, disqualification
