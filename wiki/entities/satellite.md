---
title: Satellite
type: entity
tags: [storj, coordination, metadata, audits]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3]
---
# Satellite

**What it is:** A Storj v3 trusted-but-blind coordination peer.

## Notes
A Satellite is the coordination layer in [[storj]] v3. It caches node discovery info, stores per-object metadata (pointers), maintains [[storage-node]] reputation, runs [[audits]] (random-stripe retrieval + Berlekamp-Welch correction), performs data repair, manages authorization/accounts (via macaroons), aggregates billing, and pays storage nodes monthly in STORJ. Users hold accounts on a chosen Satellite; anyone may run one, and [[storj-labs]] operates approved ones.

Satellites are "trusted-but-blind": they coordinate but never see plaintext or encryption keys, since [[uplink]]s perform [[client-side-encryption]] before upload. This mirrors the metadata-server role in GFS/Lustre. A known tension: a Satellite must stay online indefinitely or repairs stop and data decays under churn — a single-point-of-availability the design aims eventually to "architect out." This contrasts with [[filecoin]]'s fully on-chain coordination and [[swarm]]'s leaderless [[redistribution-game]].

## Sources
- [[storj-v3]] — Satellite role: metadata, audits, repair, reputation, payments
