---
title: Location Addressing
type: concept
tags: [location-addressing, content-addressing, decentralized-web]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [content-addressing-tutorial, anatomy-of-a-cid-tutorial]
---
# Location Addressing

**Definition:** Identifying data by *where* it is stored — a host, path, or URL controlled by an authority — rather than by the data itself.

## How it works
A location address such as `puppies.com/beagle.jpg` names a position in a namespace owned by a domain, and assumes the desired bytes live there. The contents of a file have no inherent relationship to its address: the same URL can return different bytes over time, and there is no way to verify, from the address alone, what was actually served. Retrieval therefore depends on a specific host being online and on trusting the authority to label content honestly.

## Why it matters / trade-offs
Location addressing is simple, human-readable, and supports easy mutation (update the file behind a stable URL). But it creates single points of failure, enables silent tampering or substitution, and produces massive redundancy because identical files under different URLs cannot be recognized as the same. These weaknesses motivate [[content-addressing]], which derives a self-certifying address from the bytes themselves. The classic analogy: a location address is directions to a shelf in one store; a content address is an ISBN that fetches the same book from any source.

## Related
[[content-addressing]], [[content-identifier]], [[cryptographic-hash]]

## Sources
- [[content-addressing-tutorial]] — the centralized-web location model and its trust weaknesses
- [[anatomy-of-a-cid-tutorial]] — contrast of location vs. content addressing
