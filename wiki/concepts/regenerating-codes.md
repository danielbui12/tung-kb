---
title: Regenerating Codes (MSR/MBR)
type: concept
tags: [regenerating-codes, msr, mbr, network-coding, erasure-coding, repair-bandwidth, ceph]
created: 2026-07-04
updated: 2026-07-04
status: active
sources: [erasure-coding-vs-raid, erasure-coding-algorithms-comparison]
---
# Regenerating Codes (MSR/MBR)

**Definition:** A family of erasure codes, descended from network coding, that minimizes the *bandwidth* required to repair a lost fragment — shrinking bytes-per-helper rather than the number of helpers contacted — split into two operating points: Minimum Storage Regenerating (MSR, MDS-capable) and Minimum Bandwidth Regenerating (MBR, not MDS, trades a little storage for even less repair bandwidth).

## How it works
Classical [[reed-solomon]] repair reads `k` whole fragments to rebuild one. Regenerating codes instead subpacketize each fragment and have each of a larger set of helper nodes send only a small linear combination of its subpackets — more nodes participate, but each ships far less data, cutting total repair *bandwidth* even though the read *count* may not fall. This is the same design space network coding explored for network throughput, retargeted at storage repair. Ceph's Clay codes (Vajha et al., shipped since the Nautilus release, 2019) are a production MSR instance, using a layered/coupled subpacketization to stay MDS while cutting bandwidth.

## Why it matters / trade-offs
Regenerating codes matter when the network, not local disk I/O, is the repair bottleneck — e.g. cross-datacenter or WAN-bound repair. Ceph's Clay reports ~2.9x less repair traffic than plain RS at only 1.25x storage overhead. The trade-off is implementation complexity: subpacketization schemes are mathematically intricate and few mature open-source implementations exist beyond Clay, making regenerating codes less battle-tested than RS or [[local-reconstruction-codes]]. Regenerating codes and LRC both attack repair cost but along genuinely different axes — LRC cuts how many fragments are *read*, regenerating codes cut how many *bytes* are *sent* per helper — so the right choice depends on whether disk read amplification or network bandwidth is the actual constraint.

## Related
[[reed-solomon]] [[mds-code]] [[local-reconstruction-codes]] [[repair-bandwidth]] [[erasure-coding]] [[ceph]]

## Sources
- [[erasure-coding-vs-raid]] — MSR/MBR mechanics, network-coding lineage (Dimakis et al. ~2010), Clay codes in Ceph since Nautilus
- [[erasure-coding-algorithms-comparison]] — MSR vs MBR MDS status, ~2.9x repair-traffic figure at 1.25x overhead
