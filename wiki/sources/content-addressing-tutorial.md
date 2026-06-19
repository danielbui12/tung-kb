---
title: Content Addressing on the Decentralized Web
type: source
tags: [content-addressing, location-addressing, cryptographic-hash, content-identifier, ipfs, decentralized-web]
created: 2026-06-19
author: ProtoSchool / Unknown
source_url: local: raw/content-addressing/
source_type: docs
ingested: 2026-06-19
---
# Content Addressing on the Decentralized Web

**One-line:** A tutorial explaining how the decentralized web identifies data by its content (via cryptographic hashes and CIDs) rather than by its storage location, and why that shift enables verifiable, peer-to-peer data sharing.

## Summary
The tutorial contrasts the two fundamental models for identifying and retrieving data on the web. The centralized web uses *location addressing*: URLs point to a location controlled by a specific authority (a domain), where data is assumed to be stored. The contents of a file have no direct relationship to its location-based address, so a URL like `puppies.com/beagle.jpg` cannot itself guarantee what data actually lives there. This dependence on trusted authorities and human-applied labels makes users vulnerable to malicious actors and produces massive redundancy, since identical files are stored under countless different URLs with no way to tell they are the same.

The decentralized web instead uses *content addressing*, where the identifier for a piece of data is derived from the data itself rather than from where it sits. The enabling primitive is cryptographic hashing: a function that takes input data of any size or type and returns a single fixed-size string (a hash) that serves as a unique, content-derived name. Because the same algorithm over the same data always yields the same hash, two peers sharing an identical file will compute identical hashes, allowing byte-for-byte verification without trusting the source. Conversely, any change to the data, however small, produces a completely different hash, making tampering trivially detectable.

This property reframes the trust model. Rather than trusting authorities to label content honestly, peers can independently verify that the bytes they received match the hash they requested. Data retrieval also changes: instead of fetching from one authoritative domain, a peer requests content by its hash from the whole network. Any peer holding a matching file can serve it, so availability no longer hinges on a single host staying online. In this sense a hash functions as a link, not merely a name.

The tutorial then introduces the Content Identifier (CID) as the concrete content-addressing format used on the decentralized web. Originally developed for IPFS, a CID is a single self-describing identifier combining a *codec*, which records how to interpret the underlying data, with a *multihash*, a self-describing hash that records both the hash type (which hashing function was used) and the hash value. Because systems like Git, Bitcoin, and Ethereum already use content addressing but differ in their data interpretation and hashing functions, the CID provides a universal identifier capable of describing data across any of these systems.

## Key takeaways
- Location addressing (URLs) identifies data by where it is stored and which authority hosts it; content addressing identifies data by a fingerprint derived from the data itself.
- A URL's contents have no inherent relationship to its address, forcing reliance on trusted domains and honest labeling, which enables deception and redundancy.
- Cryptographic hashing maps arbitrary data to a fixed-size hash; identical data always hashes identically, and any change yields a different hash.
- Hashes enable verifiable, trustless retrieval: you can confirm received bytes match the requested hash without trusting the sender.
- Content is requested from the whole peer network by hash, so availability survives any single host going offline; a hash acts as a link.
- A CID combines a codec (how to interpret the data) and a multihash (a self-describing hash carrying its hash type and value) into one universal identifier.

## Concepts & entities
Wikilinks: [[content-addressing]], [[location-addressing]], [[cryptographic-hash]], [[content-identifier]], [[multihash]], [[codec]], [[ipfs]], [[decentralized-web]], [[merkle-dag]], [[peer-to-peer]], [[url]].

## Notable claims / data
- A sample CID/hash shown: `bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi`.
- The CID structure is `Codec | Multihash`, where the multihash itself decomposes into `Hash Type | Hash Value`.
- Identical files shared by different peers over the same protocol (e.g. IPFS) produce identical hashes, allowing pixel-for-pixel verification.
- A single-pixel edit (e.g. removing one whisker from an image) produces a completely new hash.
- CID was developed for IPFS but is intended as a universal identifier; Git, Ethereum, and Bitcoin also use content addressing with differing codecs and hash functions.
- Book/ISBN analogy: location addressing is like directions to a specific shelf in one store; content addressing is like an ISBN that retrieves the same book from any source.

## Questions raised
- How does a peer actually discover *which* peers hold the content for a given hash (i.e. the routing/DHT layer)?
- What happens to mutability and updates when identifiers are immutable? (Hints at naming layers like IPNS, not covered here.)
- How are large files or structured data split and linked, given a single hash represents fixed-size output? (Points toward Merkle-DAGs.)
- Which specific hash functions and codecs are supported, and how is collision resistance assured over time?
- How are hashes made more human-friendly given the acknowledged unfriendliness of raw CIDs?
