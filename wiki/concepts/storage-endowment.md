---
title: Storage Endowment
type: concept
tags: [storage-endowment, arweave, permanent-storage, cryptoeconomics, mechanism-design]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-yellow-paper]
---
# Storage Endowment

**Definition:** Arweave's pooled fund, financed by most of each one-time upload fee, that is drawn down to pay miners only when transaction fees plus inflation fall short of the cost of maintaining the weave — the mechanism that converts a single payment into perpetual storage.

## How it works
When a user uploads data they pay a one-time AR fee. A small portion goes to the mining miner immediately; the bulk flows into the endowment. A miner's per-block reward is the sum of instant transaction fees, a predetermined (decaying) inflation reward, and — *conditionally* — a draw from the endowment, taken only when `Wsize * PGBB > Rinflation + Rfees`, i.e. when fees and inflation do not cover the period's storage cost. The endowment draw is scaled by the recall block's size relative to average block size, evening out storage incentives. The fee is sized as the infinite sum of *declining* future storage costs, relying on the historically observed ~30.57%/yr fall in GB-hour storage cost (later analyses use a far more conservative ~0.5%/yr); the present value of an ever-cheaper perpetual cost is finite, so a bounded up-front payment can in principle fund storage forever.

## Why it matters / trade-offs
The endowment is what makes [[permanent-storage]] economically coherent: it decouples a user's single payment from the open-ended obligation to keep data, smoothing miner revenue across lean periods. The model's soundness hinges on the assumption that storage costs keep declining and that the AR token holds value, since costs are effectively denominated in fiat while the endowment is in a volatile token. If the cost-decline trend stalls or reverses, or the token collapses, the endowment could be exhausted faster than modelled.

## Related
[[permanent-storage]] [[blockweave]] [[mechanism-design]] [[arweave]] [[storage-incentive-mechanisms]]

## Sources
- [[arweave-yellow-paper]] — endowment funding, conditional draw formula, declining-cost model
