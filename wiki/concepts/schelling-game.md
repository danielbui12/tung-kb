---
title: Schelling Game
type: concept
tags: [mechanism-design, cryptoeconomics, consensus]
created: 2026-06-19
updated: 2026-06-19
status: stub
sources: [book-of-swarm, object-storage-domain-knowledge]
---
# Schelling Game

**Definition:** A coordination game in which staked participants are rewarded for independently converging on the same answer (a Schelling/focal point) — typically the truth — and penalized for deviating, so honest reporting is the rational equilibrium.

## How it works
Participants stake capital and commit-then-reveal an answer. Those who match the majority/consensus answer split a reward; outliers are frozen or slashed. Because each player expects others to report the obvious truth, reporting truth becomes the best individual strategy with no communication needed. [[swarm]] applies this in its [[redistribution-game]]: staked storers converge on a consensual reserve sample to prove they hold their assigned data and to redistribute [[postage-stamps]] rent.

## Why it matters / trade-offs
Schelling games let a network extract honest answers without a trusted oracle, but assume an honest majority and enough stake at risk to deter collusion; they can fail if a bribe or cartel makes a false focal point more profitable.

## Related
[[redistribution-game]], [[mechanism-design]], [[slashing]], [[proof-of-storage]], [[swarm]]

## Sources
- [[book-of-swarm]] — the redistribution game as a staked Schelling game
- [[object-storage-domain-knowledge]] — Schelling/redistribution framing across DSNs
