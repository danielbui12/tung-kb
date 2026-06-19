---
title: Blockweave
type: concept
tags: [blockweave, arweave, proof-of-access, recall-block, data-structure]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [arweave-yellow-paper, arweave-2-9]
---
# Blockweave

**Definition:** Arweave's block data structure: a graph rather than a chain, in which each block links both to the immediately previous block and to an unpredictable historical "recall block" drawn from the weave, making possession of stored data a prerequisite for mining.

## How it works
A normal blockchain is a singly-linked list — each block references only its parent. The blockweave adds a second edge: each block also references a **recall block** whose height is computed as `BHL[indep_hash mod height]`, a function of the previous block's hash. Because that hash is not known until the previous block is mined, the recall block cannot be predicted in advance. Consensus runs [[proof-of-access]] (PoA), an extension of proof-of-work that folds the entire recall block's data into the material being hashed for the PoW puzzle. A miner therefore cannot produce a valid block without holding the recall block's data, so storing the weave is directly required to mine. Win probability is the product of the fraction of historical blocks a miner stores and its relative hashing power, which probabilistically rewards storing *rare* blocks more in an under-replicated network and drives heavy redundancy in a healthy one. State memoisation (block hash list + wallet list per block) lets nodes join by downloading just the current block.

## Why it matters / trade-offs
The blockweave turns replication into a competition-driven, emergent property rather than a fixed contract-based replica count as in [[filecoin]]. It underpins Arweave's [[permanent-storage]] model and the permaweb. The trade-off is that security rests on the recall mechanism staying genuinely unpredictable and on miners finding it cheaper to store than to re-fetch data on demand — an invariant later protocol versions (RandomX packing/activation) work hard to preserve.

## Related
[[proof-of-storage]] [[permanent-storage]] [[storage-endowment]] [[arweave]] [[filecoin]] [[content-addressing]]

## Sources
- [[arweave-yellow-paper]] — blockweave/recall-block construction, PoA, win probability
- [[arweave-2-9]] — packing/activation preserving the store-vs-refetch invariant
