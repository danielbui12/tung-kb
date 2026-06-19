---
title: Poseidon Hash
type: concept
tags: [poseidon-hash, zk-snark, hash-function, filecoin, bls12-381]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, object-storage-domain-knowledge]
---
# Poseidon Hash

**Definition:** A SNARK-friendly sponge hash function defined directly over a large prime field — Filecoin uses the [[bls12-381]] scalar field (p ≈ 2^255) — designed so that hashing is cheap to *prove* inside a zero-knowledge circuit, unlike bit-oriented hashes such as SHA-256.

## How it works
Poseidon operates as a sponge over field elements rather than bytes. Each permutation round applies an algebraic S-box (the power map `x → x^α`, with α = 5 in Filecoin), an MDS matrix mixing layer, and round constants; full rounds are interleaved with cheaper partial rounds to keep the constraint count low while preserving security (M = 128 bits). Because every operation is a native field arithmetic step, it maps to very few R1CS constraints. Filecoin instantiates Poseidon at several widths — t ∈ {3, 5, 9, 12}, corresponding to Merkle-tree arities 2/4/8/11 — and uses the t=3 instance to derive CommR = H(CommC ‖ CommRLast).

## Why it matters / trade-offs
The dominant cost in [[proof-of-replication]] and [[proof-of-spacetime]] is the [[zk-snark]] proving the [[merkle-tree]] openings of a sealed replica. A SHA-256 Merkle proof costs tens of thousands of constraints per node; Poseidon costs orders of magnitude fewer, making sector proofs feasible. The trade-off is youth: arithmetic hashes have a shorter cryptanalytic track record than SHA-2, and parameter selection must be done carefully per field.

## Related
[[zk-snark]] [[merkle-tree]] [[proof-of-replication]] [[bls12-381]] [[cryptographic-hash]] [[filecoin]]

## Sources
- [[filecoin-spec]] — Poseidon parameters (BLS12-381 scalar field, α=5, M=128, widths t), Merkle-tree and CommR usage
- [[object-storage-domain-knowledge]] — SNARK-friendly hashing context
