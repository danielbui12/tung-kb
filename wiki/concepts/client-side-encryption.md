---
title: Client-Side Encryption
type: concept
tags: [client-side-encryption, privacy, key-management, storj, decentralized-storage]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [storj-v3, storj-v2]
---
# Client-Side Encryption

**Definition:** Encrypting data on the owner's own device before it is uploaded, so the storage provider and every relay node only ever handle ciphertext. [[storj]] applies authenticated encryption (AES-256-GCM or NaCl Secretbox/Salsa20+Poly1305) at the Uplink client, and Satellites and storage nodes never see plaintext or keys.

## How it works
The Uplink encrypts data in small (≤4 KB) blocks with monotonically increasing, segment-deterministic nonces, using a distinct key per segment/file so access can be delegated granularly. Keys are derived hierarchically and deterministically via a BIP32-style HMAC chain, so a single root secret derives the whole tree but any subtree key can be shared in isolation. Object paths are themselves encrypted hierarchically and deterministically, letting a user share a path prefix without exposing parent or sibling names. Only after encryption does the client erasure-code (see [[erasure-coding]]) and transfer pieces directly to nodes.

## Why it matters / trade-offs
Client-side encryption makes the storage layer trustless: a compromised or curious Satellite/node learns only existence, rough size, and access patterns — not content. Combined with macaroon authorization, it enables fine-grained, coordination-free sharing, fitting Storj's [[coordination-avoidance]] design. The trade-offs are familiar key-management burdens: lose the key and the data is unrecoverable, and the provider cannot offer server-side features (dedup across users, plaintext search, recovery).

## Related
[[storj]] [[erasure-coding]] [[coordination-avoidance]] [[content-addressing]] [[decentralized-storage-networks]]

## Sources
- [[storj-v3]] — AES-GCM/NaCl encryption blocks, per-segment keys, BIP32-style hierarchical deterministic key and path encryption, blind Satellites
- [[storj-v2]] — earlier client-side encryption with user-held 256-bit AES keys
