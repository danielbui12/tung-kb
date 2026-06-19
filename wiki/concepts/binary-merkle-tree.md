---
title: Binary Merkle Tree
type: concept
tags: [binary-merkle-tree, content-addressing, keccak256, swarm, inclusion-proof]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec, swarm-whitepaper]
---
# Binary Merkle Tree

**Definition:** Swarm's BMT is a fixed-depth binary hash tree built with Keccak256 over the 32-byte segments of a chunk's payload, producing the chunk's content address and supporting compact 32-byte-resolution inclusion proofs.

## How it works
A content-addressed [[chunk]]'s payload (≤4096 bytes) is split into 128 segments of 32 bytes, zero-padded if short. These segments are the leaves; each internal node is the Keccak256 hash of its two children, halving the count at every level until a single 32-byte BMT root remains. The chunk's span (an 8-byte length prefix) is then hashed together with that root to give the final chunk address. Because the tree is balanced and fixed at 4KB, proving that a particular 32-byte segment belongs to the chunk requires only the sibling hash at each level — a logarithmic, constant-shape proof — which is exactly what [[swarm]]'s file hash tree and the [[redistribution-game]]'s reserve sampling use to verify possession without transferring whole chunks.

## Why it matters / trade-offs
The BMT makes a chunk's address self-certifying: anyone can recompute it and detect tampering, the basis of [[content-addressing]] in Swarm. Segment-level proofs allow random access and range verification inside a 4KB chunk and feed efficient storage proofs. Fixing the structure to 4KB keeps proofs uniform and cheap but means the BMT alone only addresses one chunk; arbitrary files need the higher Swarm hash tree stacking BMT-addressed chunks. Zero-padding wastes space for sub-4KB payloads.

## Related
[[chunk]] [[content-addressing]] [[single-owner-chunk]] [[proof-of-storage]] [[redistribution-game]] [[swarm]]

## Sources
- [[book-of-swarm]] — BMT over 32-byte segments, span prepend, inclusion proofs
- [[swarm-protocol-spec]] — BMT addressing referenced in chunk validation
- [[swarm-whitepaper]] — BMT hash as content-chunk address and file checksum
