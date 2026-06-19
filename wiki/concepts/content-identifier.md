---
title: Content Identifier
type: concept
tags: [content-identifier, content-addressing, multiformats, cryptographic-hash]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [anatomy-of-a-cid-tutorial, content-addressing-tutorial, merkle-dags-tutorial, filecoin-spec]
---
# Content Identifier

**Definition:** A CID is a self-describing, content-addressed label that names data by its [[cryptographic-hash]] together with metadata describing how the hash and data are encoded.

## How it works
A CID stacks [[multiformats]] prefixes so the identifier declares its own encoding choices. **CIDv0** is simply a `base58btc`-encoded multihash, recognizable by its leading `Qm...`; it leaves the data codec (implicitly `dag-pb`) and string base implicit. **CIDv1** makes everything explicit with the binary layout `<cid-version><multicodec><multihash>`, where the multihash itself expands to `<algorithm><length><digest>`; in string form a multibase prefix is prepended (commonly `base32`, leading `b`). The multicodec records how to interpret the data (for IPFS content, always an IPLD codec). Because a CID depends on the hash digest and not the file's size, CIDs are compact and uniformly sized for a given hash algorithm. Any CIDv0 upgrades to CIDv1; a CIDv1 downgrades only when it matches the v0 defaults (`base58btc` + `dag-pb` + `sha2-256` + 32 bytes).

## Why it matters / trade-offs
The CID is the universal address used across [[ipfs]], [[ipld]], [[libp2p]], and [[filecoin]], and is the link primitive composing [[merkle-dag]]s. Self-description makes it algorithm-agnostic and upgradeable, unlike fixed formats in Git, Bitcoin, or Ethereum. The cost is opacity: raw CIDs are not human-friendly and require tooling (e.g. cid.ipfs.tech) to inspect.

## Related
[[content-addressing]], [[multiformats]], [[cryptographic-hash]], [[merkle-dag]], [[ipfs]], [[ipld]], [[filecoin]]

## Sources
- [[anatomy-of-a-cid-tutorial]] — CIDv0/v1 structure, multiformats layering, conversion rules
- [[content-addressing-tutorial]] — CID as universal content-addressed identifier
- [[merkle-dags-tutorial]] — CIDs as the links that compose DAGs
- [[filecoin-spec]] — CIDs addressing Filecoin state and piece data
