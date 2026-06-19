---
title: Storage Power Consensus
type: concept
tags: [storage-power-consensus, filecoin, consensus, power-table]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Storage Power Consensus

**Definition:** Storage Power Consensus (SPC) is the Filecoin subsystem that maintains the Power Table — mapping each miner to its quality-adjusted storage power — and brokers [[expected-consensus]], randomness tickets, and chain selection.

## How it works
SPC is the bridge between provable storage and consensus. It tracks each miner's *quality-adjusted power*: raw-byte power from [[sealing|sealed]] [[sector]]s multiplied by a quality factor (Committed Capacity < regular < verified), recorded in the Power Table held by the Storage Power Actor. SPC supplies the power weighting that [[expected-consensus]] uses for Poisson-sortition leader election, derives and serves the [[vrf]]/[[drand]] randomness tickets each epoch, and runs the heaviest-weight chain-selection rule over tipsets. It is also where power is added (after PoRep) and removed (on faults or sector termination), coordinating with [[slashing]] when [[proof-of-spacetime]] obligations are missed.

## Why it matters / trade-offs
SPC is the accounting core that makes "storage = consensus weight" actually computable and tamper-resistant — without an authoritative, continuously-updated Power Table, EC has nothing to weight elections by. By making power *quality-adjusted* it steers the network toward storing useful and verified-client data rather than empty capacity. The trade-off is that the Power Table is only as honest as the proofs feeding it, so SPC is tightly coupled to PoRep onboarding and ongoing WindowPoSt audits; lapses there must promptly reduce power or consensus security degrades.

## Related
[[expected-consensus]], [[proof-of-spacetime]], [[proof-of-replication]], [[sealing]], [[sector]], [[vrf]], [[drand]], [[slashing]], [[collateral]], [[filecoin]]

## Sources
- [[filecoin-spec]] — SPC role, Power Table, quality-adjusted power, randomness tickets, chain selection
