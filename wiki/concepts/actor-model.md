---
title: Actor Model (Filecoin)
type: concept
tags: [actor-model, filecoin-vm, smart-contracts, filecoin, state-machine]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Actor Model (Filecoin)

**Definition:** Filecoin's on-chain smart-contract model, in which every stateful entity is an *actor* — the equivalent of a smart contract — carrying a FIL balance, a `state` pointer, a `code` CID identifying its type, and a `nonce`. Actors are invoked exclusively through messages on the [[filecoin-vm]].

## How it works
There are eleven built-in system actors (Account, Init, Cron, Reward, Multisig, Storage Power, Storage Market, Storage Miner, Verified Registry, Payment Channel, and the System actor). State transitions are driven by `messages`: a message bundles a value transfer plus an optional method invocation with parameters. External users submit `SignedMessage`s and pay gas to the including miner; an actor may synchronously call another actor only as a side-effect of a top-level user message. Each message undergoes syntactic validation (well-formed addresses, non-negative value/gas) and semantic validation (signature verifies against the sender's key), producing a `MessageReceipt` with an exit code and gas consumed. An actor's `state` is a pointer into the IPLD store; collections use HAMT and AMT structures.

## Why it matters / trade-offs
The actor model gives Filecoin a clean, message-passing concurrency discipline: no shared mutable memory, all interaction mediated by validated messages, every effect metered by gas. This makes the protocol's economic machinery — miners, markets, [[collateral]], rewards — expressible as ordinary actors enforced by the chain. The restriction that cross-actor calls happen only synchronously within one top-level message keeps execution deterministic and replayable, at the cost of less flexible composition than free-form contract calls.

## Related
[[filecoin-vm]] [[collateral]] [[tipset]] [[slashing]] [[filecoin]]

## Sources
- [[filecoin-spec]] — actor definition (balance, state, code CID, nonce), 11 system actors, message validation and receipts, synchronous cross-actor calls
