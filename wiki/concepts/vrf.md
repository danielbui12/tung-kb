---
title: VRF (Verifiable Random Function)
type: concept
tags: [vrf, randomness, leader-election, filecoin, consensus]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# VRF (Verifiable Random Function)

**Definition:** A verifiable random function takes a secret key and an input and produces a pseudorandom output together with a proof; anyone holding the public key can verify the output was correctly and uniquely derived, but only the key-holder could have produced it. Filecoin uses VRFs for tickets and election proofs in leader election.

## How it works
The key-holder evaluates `(output, proof) = VRF(sk, input)`. The output is deterministic in `(sk, input)` — there is exactly one valid output per input — so the producer cannot grind over multiple results, yet to everyone else the output looks random until revealed. Filecoin builds its VRF on deterministic [[bls-signatures]]: signing a seeded input yields a unique signature whose hash is the random output. In [[expected-consensus]], each miner draws an input from prior tickets plus [[drand]] beacon randomness, computes a VRF, and wins block production via Poisson sortition if the output falls below a threshold scaled by its quality-adjusted power.

## Why it matters / trade-offs
VRFs give Filecoin unbiasable, privately-computed, publicly-verifiable leader election: a miner cannot bias its own draw, cannot prove a win it did not earn, and others cannot predict winners in advance. This underpins Sybil-resistant, grinding-resistant consensus. The cost is reliance on the underlying signature/curve assumptions and on a clean external randomness source ([[drand]]) to seed inputs.

## Related
[[bls-signatures]] [[expected-consensus]] [[drand]] [[tipset]] [[filecoin]]

## Sources
- [[filecoin-spec]] — VRF-based tickets and election proofs seeded by drand, Poisson sortition in Expected Consensus
