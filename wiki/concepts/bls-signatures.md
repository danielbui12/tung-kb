---
title: BLS Signatures
type: concept
tags: [bls-signatures, cryptography, aggregation, filecoin, bls12-381]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [filecoin-spec]
---
# BLS Signatures

**Definition:** Boneh–Lynn–Shacham signatures: pairing-based digital signatures over the [[bls12-381]] curve that are deterministic and aggregatable — many signatures over distinct messages compress into a single short signature. Filecoin uses G1 for public keys and G2 for signatures.

## How it works
Signing maps a message to a curve point via hash-to-curve and multiplies it by the secret key; verification checks a bilinear pairing equation `e(sig, g) = e(H(m), pk)`. Because the scheme is deterministic, signing the same message always yields the same signature — no per-signature randomness to leak or mismanage. Aggregation exploits pairing bilinearity: a set of signatures sums into one group element verifiable against the combined public keys. Filecoin pairs BLS with secp256k1 ECDSA for external compatibility, and tags addresses by key type (protocol 3 = BLS, 48-byte public key).

## Why it matters / trade-offs
Aggregation is decisive for a blockchain: blocks, tickets, and messages carry signatures, and collapsing them shrinks block size and verification cost. Determinism also makes BLS a natural base for a [[vrf]], used in [[expected-consensus]] leader election. Trade-offs: signature verification (pairings) is slower per-signature than ECDSA, BLS12-381 is a relatively new curve, and aggregation requires care against rogue-key attacks (mitigated by proof-of-possession or message augmentation).

## Related
[[bls12-381]] [[vrf]] [[expected-consensus]] [[tipset]] [[filecoin]]

## Sources
- [[filecoin-spec]] — BLS12-381 (G1 keys, G2 sigs), deterministic + aggregatable, used for blocks/tickets/messages; address protocol tagging
