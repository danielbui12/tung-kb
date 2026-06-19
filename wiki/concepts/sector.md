---
title: Sector
type: concept
tags: [sector, filecoin, sealing, storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Sector

**Definition:** A sector is Filecoin's fixed-size unit of storage — 32 GiB or 64 GiB, chosen by the miner — that gets [[sealing|sealed]] and continuously proven; it holds either client deal data or committed-capacity data.

## How it works
Storage is committed in sectors rather than per-file. A sector is filled either with client deal data (a "regular" or "verified" sector) or with arbitrary filler (Committed Capacity / CC). The data pipeline is: file → UnixFS DAG → CAR → padded Piece (CommP) → Sector (CommD, unsealed). The miner then [[sealing|seals]] the sector via [[proof-of-replication]], yielding CommR, and thereafter answers [[proof-of-spacetime]] challenges over it. Each sector contributes raw-byte power that is multiplied by a quality factor (CC < regular < verified) to give quality-adjusted power in the [[storage-power-consensus]] Power Table. WindowPoSt audits sectors in partitions of 2349 sectors with 10 challenges each.

## Why it matters / trade-offs
The fixed-size sector is the granularity at which all proving, power accounting, collateral, and slashing happen — it standardizes proofs so SNARK circuits and audit schedules can be fixed-cost regardless of the underlying files. Larger 64 GiB sectors amortize sealing overhead but raise the cost of any single fault. Committed-capacity sectors let miners onboard power before securing deals, which is why power is *quality-adjusted* — to steer capacity toward useful and verified-client data rather than empty filler.

## Related
[[sealing]], [[proof-of-replication]], [[proof-of-spacetime]], [[storage-power-consensus]], [[expected-consensus]], [[collateral]], [[filecoin]]

## Sources
- [[filecoin-spec]] — sector sizes, CC vs regular/verified, data pipeline, partition parameters
