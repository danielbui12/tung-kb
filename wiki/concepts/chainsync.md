---
title: ChainSync
type: concept
tags: [filecoin, blockchain-sync, libp2p, consensus]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# ChainSync

**Definition:** ChainSync is Filecoin's blockchain synchronization protocol — a family of sub-protocols (hello, GossipSub, BlockSync, Bitswap/GraphSync) driven by a conceptual state machine that replicates and validates the tipset chain from a trusted checkpoint up to the production fringe.

## How it works

ChainSync is modeled as a 5-state machine. Implementations MAY blur the lines but MUST preserve security.

```
INIT            → verify local data structures, validate local chain (resource-expensive
                  checks MAY be skipped at own risk)
BOOTSTRAP       → discover a "secure enough" peer set; join GossipSub topics
                  /fil/blocks and /fil/msgs
SYNC_CHECKPOINT → start from TrustedCheckpoint (default GenesisCheckpoint; verified by
                  operators, NOT in software); fetch the block it points to, its parents,
                  and the StateTree
CHAIN_CATCHUP   → maintain TargetHeads, pick BestTargetHead, sync toward it via BlockSync,
                  validating blocks and requesting intermediate tipsets; heads churn as new
                  fringe blocks arrive and bad paths fail validation
CHAIN_FOLLOW    → caught up; receive/validate/propagate blocks via GossipSub (Bitswap
                  fallback); finalize tipsets. Revert to CHAIN_CATCHUP if security degrades
```

**hello protocol**: on first connect, two peers exchange chain heads. **BestTargetHead** is selected by comparing (height, weight); ties broken by higher [[expected-consensus]] ChainWeight.

**BlockSync** (`/fil/sync/blk/0.0.1`) — a request/response protocol operating on tipsets (not single blocks). It fetches a chain *segment* by the CID of its highest tipset, so it can fill gaps mid-chain, not just the head.

```go
type BlockSyncRequest struct {
    start         [&Block]   // CIDs of highest tipset in segment
    requestLength UInt       // how many tipsets back to fetch
    options       Options    // Blocks=1, Messages=2, BlocksAndMessages=3
}
type BlockSyncResponse struct { chain [TipSetBundle]; status Status }
type TipSetBundle struct {
    blocks         [Blocks]
    blsMsgs        [Message];        blsMsgIncludes  [[UInt]]
    secpMsgs       [SignedMessage];  secpMsgIncludes [[UInt]]
}
```

Response is in **reverse iteration order** (head → older). `MsgIncludes` arrays map each block to indexes into the dedup'd message arrays. Status: `Success 0`, `PartialResponse 101` (head fetched, not all tipsets — NOT an error), `BlockNotFound 201`, `GoAway 202` (rate limit), `InternalError 203`, `BadRequest 204`. Example gap: height 5000 but missing 4500–4700 → request Head=4700, length=200.

**GossipSub** propagates blocks/messages via two link types: *mesh links* (eager-push, full messages) and *gossip links* (lazy-pull, IHAVE/IWANT of message IDs). All pubsub messages are authenticated and syntactically validated before forwarding. **Bitswap** is the fallback when CHAIN_FOLLOW hears of a block via gossip-IDs it never received in full; GraphSync is a more efficient alternative.

**Progressive Block Validation (BV0–BV8)** prunes bad blocks cheaply before expensive checks:

```
BV0 Syntax           serialization, typing, value ranges
BV1 Plausible cons.  miner/weight/epoch plausible (state at epoch - LookbackParameter)
BV2 Block signature
BV3 Beacon entries   valid random-beacon entries inserted
BV4 ElectionProof    valid election proof
BV5 WinningPoSt      correct PoSt   (see [[proof-of-spacetime]])
BV6 Ancestry/finality links to trusted chain, not prior to finality
BV7 Message sigs
BV8 State tree       parent tipset execution → claimed state root + receipts
```

## State & parameters

- **LastCheckpoint** — last hard social-consensus checkpoint; defines minimum finality and history floor; taken on faith, never reverted past.
- **TargetHeads** — list of BlockCIDs at the production fringe, sorted by likelihood of being the best chain (by ChainWeight).
- **BestTargetHead** — first element of TargetHeads.
- **TrustedCheckpoint** — sync origin (default GenesisCheckpoint).
- **Validation cache** — unvalidated blocks, ideally sorted by chain-membership likelihood; dropped when passed by FinalityTipset or under resource load.

## Why it matters / trade-offs

State replication is security-critical. Progressive validation is a deliberate DoS defense: full validation (BV5/BV8) is costly, so cheap stages prune attacker spam early. Operator-verified checkpoints anchor finality without trusting software. BlockSync's tipset-segment model lets nodes recover from partial outages efficiently.

## Related
[[filecoin]] · [[tipset]] · [[expected-consensus]] · [[storage-power-consensus]] · [[gossipsub]] · [[libp2p]] · [[proof-of-spacetime]] · [[graphsync-data-transfer]] · [[bls-signatures]]

## Sources
- [[filecoin-spec]] — ChainSync state machine, BlockSync wire protocol, GossipSub/hello/Bitswap roles, BV0–BV8 validation stages.
