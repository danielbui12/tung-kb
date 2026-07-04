---
title: Walrus
type: entity
tags: [decentralized-storage, erasure-coding, data-availability, sui]
created: 2026-06-19
updated: 2026-07-04
status: active
sources: [object-storage-domain-knowledge, erasure-coding-algorithms-comparison]
---
# Walrus

**What it is:** A self-healing decentralized storage and data-availability layer on Sui.

## Notes
Walrus is the self-healing-storage / data-availability member of the [[decentralized-storage-networks]]. It uses a two-dimensional "Red Stuff" [[erasure-coding]] scheme ([[reed-solomon]] over GF(2^16), ~4.5x expansion) whose key property is recovery bandwidth of O(|blob|/n) rather than the O(|blob|) of classic 1-D RS — letting the network self-heal cheaply after node loss. (A common misconception, corrected in the source, is that Walrus ships RaptorQ; that was v1-paper only — see [[fountain-codes]] for why RS won out over the rateless option in shipped storage networks.)

A BFT committee of n = 3f+1 nodes (n_shards=1000, quorum 2f+1=667) runs on [[sui]], which anchors the blob lifecycle, committee management, and the on-chain Point of Availability. BlobId = Blake2b256(encoding_type ‖ length ‖ merkle_root), keeping addressing [[content-addressing|content-addressed]]. Proofs are signature- and challenge-based (logarithmic challenges) rather than the heavy zk-SNARKs of [[filecoin]].

Compare [[storj]] (1-D RS, ~2.76x, lightweight [[audits]]) and [[swarm]] (neighborhood replication). Token: WAL. Payment is recurring rent, unlike [[arweave]]'s pay-once endowment.

## Sources
- [[object-storage-domain-knowledge]] — Red Stuff encoding, Sui committee, parameters, RaptorQ correction
