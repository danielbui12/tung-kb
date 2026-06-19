---
title: Multiformats
type: entity
tags: [content-addressing, self-describing-formats, content-identifier]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [anatomy-of-a-cid-tutorial, filecoin-spec]
---
# Multiformats

**What it is:** A project of self-describing value formats that make data future-proof and algorithm-agnostic.

## Notes
Multiformats, originated in [[ipfs]], bundles several self-describing prefix formats: **multihash** (a `type-length-value` hash where the type names the algorithm, e.g. `sha2-256` = `0x12`, length = 32 bytes), **multicodec** (an identifier of the data encoding, e.g. `dag-pb` = `0x70`), **multibase** (the string base encoding, e.g. `base32` for CIDv1, `base58btc` for CIDv0), and the **[[content-identifier]]** (CID) that stacks them.

The design lets a value declare its own encoding choices, so formats stay upgradeable when an algorithm is broken (as `sha1` was). A CIDv1's binary layout is `<version><multicodec><multihash>`, with multibase added only in string form.

Multiformats underpins [[content-addressing]] across [[ipfs]], [[ipld]], [[libp2p]] (PeerIDs/multiaddrs), and [[filecoin]] (CIDs throughout its data pipeline and state). Compare [[swarm]], which uses fixed Keccak256 BMT chunk addresses rather than multiformats CIDs.

## Sources
- [[anatomy-of-a-cid-tutorial]] — multihash/multicodec/multibase/CID structure
- [[filecoin-spec]] — multiformats (CIDs) in the Filecoin stack
