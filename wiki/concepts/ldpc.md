---
title: LDPC (Low-Density Parity-Check Codes)
type: concept
tags: [ldpc, channel-coding, erasure-coding, belief-propagation]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-algorithms-comparison]
---
# LDPC (Low-Density Parity-Check Codes)

**Definition:** A fixed-rate, sparse-graph code decoded by fast iterative belief-propagation passes, tuned for near-Shannon-capacity throughput on communication channels rather than for storage-style "any k of n" reconstruction.

## How it works
LDPC represents parity constraints as a sparse bipartite graph between data and parity nodes (few edges per node, hence "low density"). Decoding runs iterative message-passing ("belief propagation") over this graph rather than the dense matrix inversion Reed-Solomon uses — which is why LDPC scales to enormous block sizes with fast hardware decoders. Like RS it is fixed-rate (`n` known in advance), but it is not [[mds-code|MDS]]: it does not guarantee exact recovery from any arbitrary `k` of `n` symbols, only near-capacity performance in the statistical sense channel coding cares about.

## Why it matters / trade-offs
LDPC is the workhorse of real-time communication channels — Wi-Fi, 5G, DVB satellite — where the goal is near-Shannon-capacity throughput at huge block sizes with very fast hardware decoding, not the storage system's "any k of n" reconstruction guarantee. For storage, sparse-graph designs make it hard to guarantee exact recovery thresholds the way RS's clean algebraic MDS bound does, so LDPC is essentially absent from disk/object storage despite superficially resembling a fixed-rate erasure code. It sits alongside [[fountain-codes]] as a "transmission-side" pick, contrasted with RS/[[local-reconstruction-codes]]/[[regenerating-codes]] on the "storage-side."

## Related
[[reed-solomon]] [[mds-code]] [[fountain-codes]] [[erasure-coding]]

## Sources
- [[erasure-coding-algorithms-comparison]] — LDPC mechanics, storage-vs-transmission split, Wi-Fi/5G/DVB use cases
