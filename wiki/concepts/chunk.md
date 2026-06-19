---
title: Chunk
type: concept
tags: [chunk, swarm, content-addressing, storage-unit, binary-merkle-tree]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec, swarm-whitepaper]
---
# Chunk

**Definition:** Swarm's basic unit of storage and transfer: a fixed-size blob of at most 4096 bytes of payload prefixed by an 8-byte span, whose 256-bit address lives in the same space as node addresses.

## How it works
Because a chunk's address shares the node address space, the [[distributed-hash-table]] places each chunk at the nodes closest to its address by [[proximity-order]]. There are two chunk types. A **content-addressed chunk** is addressed by the [[binary-merkle-tree]] hash of its span and payload, so the address self-certifies integrity and supports compact inclusion proofs. A [[single-owner-chunk]] is addressed by `Keccak256(identifier + owner)` and carries the owner's EC signature, giving an account a writable region of the address space. The 8-byte little-endian span records how many bytes the subtree under this chunk represents, which lets the [[swarm]] hash tree pack chunk references (128 per intermediate chunk, 64 when encrypted) into a balanced tree representing files of any length with random access and logarithmic proofs. Chunks are encrypted by default, making them indistinguishable from random data.

## Why it matters / trade-offs
Fixing the unit at ≤4KB makes the network uniform: every chunk is routed, priced, replicated and proved identically, and identical content deduplicates automatically. Encryption shields node operators from liability for content they did not choose. The cost is overhead for small objects and the need for a hash-tree layer to reassemble large files; mutability must be emulated via single-owner chunks and feeds since content-addressed chunks are immutable.

## Related
[[binary-merkle-tree]] [[single-owner-chunk]] [[content-addressing]] [[proximity-order]] [[distributed-hash-table]] [[postage-stamps]] [[swarm]]

## Sources
- [[book-of-swarm]] — chunk format, two chunk types, hash tree, encryption
- [[swarm-protocol-spec]] — chunk delivery on the wire, storage radius placement
- [[swarm-whitepaper]] — chunk as canonical DISC storage unit
