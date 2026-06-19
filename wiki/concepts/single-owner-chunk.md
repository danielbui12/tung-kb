---
title: Single-Owner Chunk
type: concept
tags: [single-owner-chunk, swarm, content-addressing, feeds, mutability]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [book-of-swarm, swarm-protocol-spec, swarm-whitepaper]
---
# Single-Owner Chunk

**Definition:** A Swarm chunk type whose address is `Keccak256(identifier + owner)` and whose integrity is warranted by the owner's elliptic-curve signature, giving each account a freely assignable, writable subspace of the chunk address space.

## How it works
A single-owner chunk (SOC) packs a 32-byte identifier, a 65-byte (r,s,v) signature, an 8-byte span and up to 4096 bytes of payload. The owner signs the binding of identifier to payload, so storers and retrievers can verify the chunk is authentic for that owner without re-deriving the address from content. The address is computed from the identifier and owner alone, which means the owner can decide *where* in the address space a chunk lands and can later replace the payload at the same address by re-signing — something a content-addressed [[chunk]] (whose address is its [[binary-merkle-tree]] hash) cannot do. This writable address subspace is the foundation for **feeds** (SOCs indexed by topic + sequence, simulating mutable resources and version history) and for **PSS** Trojan-chunk messaging and addressed envelopes, where a target address is mined.

## Why it matters / trade-offs
SOCs add mutability and identity to an otherwise immutable, content-addressed store, without abandoning the uniform [[distributed-hash-table]] placement and [[postage-stamps]] economics. Integrity shifts from "hash of content" to "trust this owner's key," so a SOC is only as honest as its signer; a compromised key can rewrite an address. Each identifier+owner pair is unique, preventing one owner from squatting another's subspace.

## Related
[[chunk]] [[binary-merkle-tree]] [[content-addressing]] [[proximity-order]] [[postage-stamps]] [[swarm]] [[ens]]

## Sources
- [[book-of-swarm]] — SOC format, writable subspace, feeds and PSS built on SOCs
- [[swarm-protocol-spec]] — SOC handling in chunk delivery/validation
- [[swarm-whitepaper]] — SOC as the mutable chunk type, integrity by signature
