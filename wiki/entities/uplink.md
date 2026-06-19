---
title: Uplink
type: entity
tags: [storj, client, encryption, erasure-coding]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3]
---
# Uplink

**What it is:** The Storj v3 lightweight client.

## Notes
An Uplink is the client in [[storj]] v3, available as the `libuplink` library, an S3 gateway, or a CLI. It performs [[client-side-encryption]] (AES-GCM or NaCl Secretbox in ≤4KB blocks, with hierarchical deterministic BIP32-style path encryption), [[reed-solomon]] [[erasure-coding]] (k,m,o,n parameters), and direct peer-to-peer transfer of pieces to [[storage-node]]s — bypassing the [[satellite]] for bulk data so the Satellite stays blind to plaintext.

Over-encoding lets uploads/downloads complete from only the fastest nodes (canceling after o pieces) to mitigate the long tail. The Uplink also signs incremental bandwidth "orders" against Satellite-signed "order limits" so bandwidth accounting can't be cheated.

The Uplink is where encryption and redundancy live, keeping the rest of the network untrusted — analogous to the client-side responsibility split in [[filecoin]]'s data pipeline, but Storj uniquely hides paths and metadata too.

## Sources
- [[storj-v3]] — Uplink role: encryption, erasure coding, direct P2P transfer, bandwidth orders
