---
title: Anatomy of a CID
type: source
tags: [content-addressing, cid, multiformats, ipfs, ipld, cryptographic-hash, decentralized-web]
created: 2026-06-19
author: ProtoSchool / Unknown
source_url: "local: raw/anatomy-of-a-CID/"
source_type: docs
ingested: 2026-06-19
---
# Anatomy of a CID

**One-line:** A content identifier (CID) is a self-describing, content-addressed label built from stacked Multiformats prefixes (multibase, version, multicodec, multihash) that names data by its cryptographic hash rather than its location.

## Summary
A [[content-identifier]] (CID) is the core identifier used across [[ipfs]], [[ipld]], libp2p, and Filecoin to reference content via [[content-addressing]] rather than location addressing. A CID does not say *where* data lives; it derives an address from the content itself by way of a [[cryptographic-hash]]. Because the identifier depends on the hash digest and not the file size, CIDs are compact and (when the same hash algorithm is used) uniformly sized — most IPFS content is hashed with `sha2-256`, yielding 256-bit / 32-byte digests. A good cryptographic hash algorithm is deterministic, uncorrelated (a one-pixel change yields a wholly different hash), one-way, and collision-resistant ("unique"), which guarantees fetched content matches its requested address.

To remain future-proof against any single algorithm being broken (as happened with `sha1`), CIDs embed a [[multihash]]: a self-describing hash following the `TLV` (type-length-value) pattern. The `type` field is a [[multicodec]] identifier of the hash algorithm (e.g. `sha2-256` is `18` / `0x12`), `length` is the digest length in bytes, and `value` is the raw digest. The original CID format — now [[cidv0]] — was simply a `base58btc`-encoded multihash, recognizable by its leading `Qm...` characters. CIDv0 leaves the data encoding and the string base implicit, which raised concerns about how to know the data's encoding and whether `base58btc` would always be used.

[[cidv1]] resolves these by adding explicit prefixes. A multicodec prefix records the data encoding (e.g. `dag-pb`, `0x70`, an IPLD codec — IPFS data always uses an IPLD format, so an IPFS CID's multicodec is always an IPLD codec). A version prefix distinguishes CIDv0 from CIDv1. The binary CIDv1 layout is therefore `<cid-version><multicodec><multihash>`, where the multihash itself expands to `<algorithm><length><digest>`.

For human-friendly text form, a [[multibase]] prefix records which base encoding converts the binary CID to/from string. This prefix exists only in the string form. CIDv1 string CIDs typically begin with `b` for `base32` (the IPFS default), whereas CIDv0 string hashes are always `base58btc` and begin with `Qm`. [[multicodec]], [[multihash]], and [[multibase]] are all part of the [[multiformats]] project, which originated in IPFS and now serves a broad range of distributed systems.

Both versions can encode the same underlying content: a CIDv0 and its converted CIDv1 share the identical digest (e.g. `C3C4733EC8AFFD06CF9E9FF50FFC6BCD2EC85A6170004BB709669C31DE94391A`). Any CIDv0 converts to CIDv1 (implicit prefixes become explicit), but a CIDv1 converts back to CIDv0 only when `multibase = base58btc`, `multicodec = dag-pb`, `multihash-algorithm = sha2-256`, and `multihash-length = 32`. The CID Inspector tool (cid.ipfs.tech) parses any CID into its component prefixes.

## Key takeaways
- A CID addresses data by its content hash, not its storage location, so fetching by CID guarantees integrity.
- Multiformats stacks self-describing prefixes so a CID declares its own encoding choices, making the format upgradeable and algorithm-agnostic.
- Multihash uses a `type-length-value` layout; `sha2-256` has type id `18` (`0x12`) and produces a 32-byte digest.
- CIDv0 = `base58btc(multihash)` with implicit `dag-pb` codec; identified by the `Qm...` prefix.
- CIDv1 = `<multibase><cid-version><multicodec><multihash>` (multibase only in string form); commonly `base32`, identified by a leading `b`.
- Every CIDv0 upgrades to CIDv1; only CIDv1s matching the v0 defaults (base58btc / dag-pb / sha2-256 / 32 bytes) downgrade back.

## Concepts & entities
Wikilinks: [[content-identifier]], [[content-addressing]], [[cryptographic-hash]], [[multiformats]], [[multihash]], [[multicodec]], [[multibase]], [[cidv0]], [[cidv1]], [[ipfs]], [[ipld]], [[location-addressing]], [[dag-pb]], [[sha2-256]], [[base58btc]], [[base32]], [[cid-inspector]], [[protoschool]].

## Notable claims / data
- `sha2-256` digest = 256 bits = 32 bytes; this is the IPFS default hash.
- Multihash `TLV`: `<type><length><value>`; `sha2-256` type = `18` = `0x12`.
- `dag-pb` multicodec code = `0x70` (an IPLD format).
- CIDv1 binary layout: `<cid-version><multicodec><multihash-algorithm><multihash-length><multihash-hash>`; string form prepends a multibase prefix.
- CIDv0 prefix `Qm` (base58btc); CIDv1 default prefix `b` (base32).
- CIDv1 → CIDv0 conversion requires base58btc + dag-pb + sha2-256 + 32-byte length.
- Shared example digest across versions: `C3C4733EC8AFFD06CF9E9FF50FFC6BCD2EC85A6170004BB709669C31DE94391A`.
- Example aardvark CIDv0: `QmcRD4wkPPi6dig81r5sLj9Zm1gDCL4zgpEj9CfuRrGbzF`.

## Questions raised
- Which non-`sha2-256` algorithms (e.g. `blake2b`, `sha3-256`) see real adoption in CIDs, and how is migration handled in practice?
- Beyond `dag-pb`, which IPLD codecs (e.g. `dag-cbor`) are most common, and how do they affect interoperability?
- How do downstream systems (libp2p, Filecoin) constrain or extend the codec/base choices they accept?
- What are the practical trade-offs (DNS-safety, length, case-sensitivity) driving base32 over base58btc?
