---
title: drand
type: entity
tags: [randomness-beacon, distributed-randomness, consensus]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# drand

**What it is:** A distributed, publicly-verifiable, unbiasable randomness beacon.

## Notes
drand produces public randomness that no single party can bias, by combining contributions from a threshold of participants. [[filecoin]] consumes drand beacon entries (combined with domain-separation tags) as the unbiasable randomness seed for its Expected Consensus leader election — each miner runs a VRF-based election seeded by drand — and for deriving proof challenges (e.g. SealRandomness drawn from the VRF chain prevents long-range/replay attacks in [[proof-of-replication]] [[sealing]]).

In the [[proof-of-storage-systems]] context, drand is the trusted source of fresh, unpredictable challenges that make Filecoin's [[proof-of-spacetime]] audits meaningful. Compare [[swarm]]'s [[redistribution-game]], which derives its randomness internally from XORed commit-reveal nonces rather than an external beacon.

## Sources
- [[filecoin-spec]] — drand as the randomness source for EC leader election and proof challenges
