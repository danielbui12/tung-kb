---
title: FIL Token
type: entity
tags: [token, filecoin, cryptoeconomics]
created: 2026-06-19
updated: 2026-06-19
status: stub
sources: [filecoin-spec]
---
# FIL Token

**What it is:** The native currency (⨎) of [[filecoin]], used for deal payments, gas, collateral, and block rewards.

## Notes
FIL is unusual in fusing payment with security: miners must lock FIL as [[collateral]] (initial pledge + vesting rewards + deal collateral), and storage power — not raw token holdings — determines consensus weight via [[expected-consensus]]. Block rewards are minted on a mixed schedule (≈30% simple exponential decay + 70% storage-baseline). Faults trigger [[slashing]] of locked FIL. Contrast [[storj-token]] and [[bzz-token]], which are payment tokens with no direct consensus role.

## Sources
- [[filecoin-spec]] — FIL's role in deals, gas, collateral, minting, and slashing
