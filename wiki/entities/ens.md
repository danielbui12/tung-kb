---
title: ENS
type: entity
tags: [naming, ethereum, name-resolution]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-whitepaper]
---
# ENS

**What it is:** The Ethereum Name Service — maps human-readable names to addresses and content references.

## Notes
ENS resolves human-readable domains (e.g. `swarm.eth`) to on-chain references. In [[swarm]], ENS is the naming layer above the content-addressed store: it resolves a domain to a Swarm root reference (typically a [[swarm]] manifest root), letting websites and apps served over the `bzz` URL scheme have stable, human-friendly names despite the underlying [[content-addressing|content addresses]] being immutable hashes.

This addresses the mutability/naming gap inherent to content addressing (the same gap [[ipfs]] handles with IPNS). ENS lives on [[ethereum]].

## Sources
- [[book-of-swarm]] — ENS name resolution for bzz manifests
- [[swarm-whitepaper]] — ENS resolving domains to Swarm references
