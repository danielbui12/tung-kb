---
title: Cryptographic Hash
type: concept
tags: [cryptographic-hash, content-addressing, integrity, security]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [content-addressing-tutorial, anatomy-of-a-cid-tutorial, merkle-dags-tutorial]
---
# Cryptographic Hash

**Definition:** A function that maps input data of any size to a fixed-size digest in a way that is deterministic, one-way, collision-resistant, and uncorrelated with the input.

## How it works
The function takes arbitrary bytes and returns a fixed-length string (e.g. `sha2-256` yields a 256-bit / 32-byte digest). Four properties make it suitable for addressing and integrity: **deterministic** — identical input always yields the identical digest; **one-way** — the input cannot be recovered from the digest; **collision-resistant** — it is computationally infeasible to find two inputs with the same digest, so a digest is effectively unique; and **uncorrelated** — a one-bit (e.g. one-pixel) change produces a completely different, unpredictable digest. This guarantees that fetched content matching a requested digest is byte-for-byte authentic.

## Why it matters / trade-offs
Cryptographic hashing is the primitive underlying [[content-addressing]], [[content-identifier]]s, [[merkle-tree]]s, and [[merkle-dag]]s. It lets mutually distrusting peers verify data without a trusted authority. The main risk is algorithm obsolescence: `sha1` was broken, so systems wrap digests in self-describing [[multiformats]] (multihash) to allow migrating to stronger functions like `sha2-256`, `sha3`, or `blake2b` without changing the format.

## Related
[[content-addressing]], [[content-identifier]], [[multiformats]], [[merkle-tree]], [[merkle-dag]], [[poseidon-hash]]

## Sources
- [[content-addressing-tutorial]] — hashing as the enabling primitive for content addressing
- [[anatomy-of-a-cid-tutorial]] — the four hash properties and `sha2-256` defaults
- [[merkle-dags-tutorial]] — hashes as fingerprints and structural links
