---
title: zk-SNARK
type: concept
tags: [zk-snark, zero-knowledge, groth16, filecoin, proof-systems]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec, object-storage-domain-knowledge]
---
# zk-SNARK

**Definition:** A Zero-Knowledge Succinct Non-interactive ARgument of Knowledge: a cryptographic proof that lets a prover convince a verifier a statement is true while revealing nothing else, with a proof that is tiny and fast to verify regardless of the computation's size. Filecoin uses the Groth16 construction.

## How it works
The statement to prove is compiled into an arithmetic circuit, then into a Rank-1 Constraint System and a Quadratic Arithmetic Program. Groth16 turns satisfying that QAP into checking a small set of pairing equations over an elliptic curve ([[bls12-381]] in Filecoin). The result is a constant-size proof (a few group elements) verified by a handful of pairings, independent of how large the original witness was. Groth16 requires a circuit-specific trusted setup that produces a structured reference string; the setup secret ("toxic waste") must be destroyed.

## Why it matters / trade-offs
Filecoin's [[proof-of-replication]] and [[proof-of-spacetime]] produce huge "vanilla" proofs — Merkle openings over billions of nodes. Submitting those on-chain would be prohibitively expensive. A zk-SNARK compresses each into a few hundred bytes that any node verifies cheaply, making provable storage economically viable on a blockchain. The proving step is itself heavy (this is why [[poseidon-hash]] is used to keep constraint counts low), and the trusted setup is a one-time ceremony risk.

## Related
[[poseidon-hash]] [[proof-of-replication]] [[proof-of-spacetime]] [[bls12-381]] [[filecoin]]

## Sources
- [[filecoin-spec]] — Groth16 used to compress PoRep/PoSt vanilla proofs for on-chain verification
- [[object-storage-domain-knowledge]] — succinct-proof context for storage systems
