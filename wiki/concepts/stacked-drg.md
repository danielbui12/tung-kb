---
title: Stacked DRG
type: concept
tags: [stacked-drg, filecoin, sealing, proof-of-replication, cryptography]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# Stacked DRG

**Definition:** Stacked Depth-Robust Graph (SDR) is the [[proof-of-replication]] sealing construction Filecoin launched with: a stack of depth-robust graphs joined by bipartite expanders that enforces slow, sequential, non-parallelizable, regeneration-resistant encoding of a [[sector]].

## How it works
SDR stacks `LAYERS` = 10 depth-robust graphs, each of `GRAPH_SIZE` = 2^31 nodes, with adjacent layers connected by a bipartite expander graph. Within a layer, DRG parents are chosen by Bucket Sampling (base degree 6); expander parents come from a Feistel-network pseudorandom permutation (expansion degree 8, 13 parents total per node). Each node is labeled by hashing its parents' labels with SHA254; because a node depends on earlier-labeled parents, labeling must proceed sequentially. The final layer's labels form a key that encodes the data into the replica. Commitments: CommC over the column labels, CommRLast over the replica, and CommR = H(CommC ‖ CommRLast). Merkle trees use the [[poseidon-hash]] over the BLS12-381 scalar field.

## Why it matters / trade-offs
Depth-robustness guarantees that any attempt to discard labels and recompute them on challenge still requires a long sequential computation — so a replica cannot be cheaply regenerated, which is exactly what makes PoRep meaningful. The deliberate slowness is the security property but also the main cost: [[sealing]] is CPU- and time-heavy, motivating the planned Narrow Stacked Expander (NSE) successor for lower sealing cost and retrieval latency.

## Related
[[proof-of-replication]], [[sealing]], [[sector]], [[poseidon-hash]], [[zk-snark]], [[merkle-tree]], [[filecoin]]

## Sources
- [[filecoin-spec]] — SDR layers, DRG/expander parents, labeling hash, CommC/CommR, parameters
