---
title: Filecoin VM
type: concept
tags: [filecoin-vm, state-machine, gas, ipld, filecoin]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Filecoin VM

**Definition:** The Filecoin Virtual Machine is a replicated state machine that executes actor code with gas metering, deriving the network's global State Tree from the messages applied at each [[tipset]]. Its output is an IPLD DAG-CBOR state tree identified by a CID.

## How it works
The VM runs the [[actor-model]]: every on-chain operation invokes a method on an actor and incurs a gas cost charged to the sender, who pays the including miner. Applying the messages of a tipset against the prior State Tree produces a new State Tree — a [[merkle-tree]]/DAG of all actors, each at its address. State is stored as IPLD DAG-CBOR (with HAMT and AMT collections) in a content-addressed store, so the entire global state reduces to a single root CID that every node can recompute and agree on. Each executed message yields a `MessageReceipt` with an exit code and gas used; deterministic execution lets all nodes replay messages and converge on the identical state root.

## Why it matters / trade-offs
Gas metering bounds resource use and prices computation, preventing denial-of-service and funding miners. Content-addressed, deterministic state means consensus reduces to agreeing on one CID, and any node can verify a state transition by re-execution. The same determinism constrains the model: execution must be reproducible across implementations (Lotus, Venus, Forest), so gas costs and semantics are tightly specified. Later FEVM work layers EVM compatibility atop this native actor VM.

## Related
[[actor-model]] [[tipset]] [[merkle-tree]] [[content-addressing]] [[collateral]] [[filecoin]]

## Sources
- [[filecoin-spec]] — VM as replicated state machine, gas per operation, State Tree as IPLD DAG-CBOR identified by CID, message receipts
