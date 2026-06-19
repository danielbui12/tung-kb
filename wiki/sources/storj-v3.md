---
title: "Storj: A Decentralized Cloud Storage Network Framework (v3)"
type: source
tags: [decentralized-storage, object-storage, erasure-coding, reed-solomon, s3-compatibility, audits, peer-to-peer]
created: 2026-06-19
author: Storj Labs, Inc.
source_url: "local: raw/storjv3.md"
source_type: paper
ingested: 2026-06-19
---
# Storj: A Decentralized Cloud Storage Network Framework (v3)

**One-line:** Storj v3 is a modular, S3-compatible decentralized cloud object store that erasure-codes client-side-encrypted data across globally distributed untrusted storage nodes, coordinated by trusted-but-blind Satellites, while deliberately avoiding blockchain/Byzantine consensus on the hot path.

## Summary
Storj v3 (whitepaper v3.0 Oct 2018, revised v3.1 Mar 2024) defines both a *framework* for decentralized cloud storage and an initial *concrete implementation*. The framework decomposes the system into eight independent, swappable components: storage nodes, peer-to-peer communication, redundancy, metadata, encryption, audits and reputation, data repair, and payments. The guiding design constraints are security/privacy, decentralization (no single operator, no single point of cost or failure), marketplace economics that reward four parties (end users, storage node operators, demand providers, network operator), Amazon S3 API compatibility, high durability under device failure and node churn, low latency, aggressive bandwidth conservation, optimization for large objects (≥4MB assumed), Byzantine-Altruistic-Rational (BAR) fault tolerance, and—critically—coordination avoidance to reach exabyte scale.

The architecture has three peer classes. *Storage nodes* store pieces and are paid for at-rest storage and egress (not ingress); they register directly with Satellites they trust, advertise space/bandwidth/wallet, and serve get/put/delete keyed by piece ID + Satellite ID. *Uplinks* are lightweight clients (via the libuplink library, an S3 gateway, or a CLI) that perform encryption, erasure coding, and direct peer-to-peer transfer to storage nodes. *Satellites* are the trusted coordination layer: they cache node discovery info, store per-object metadata (pointers), maintain node reputation, run audits, perform repair, manage authorization/accounts, aggregate billing, and pay storage nodes. Users hold accounts on specific Satellites; anyone can run one. The model mirrors GFS/Lustre's split of metadata servers, object storage servers, and clients.

Redundancy uses Reed-Solomon erasure coding rather than replication, decoupling durability from expansion factor and slashing repair bandwidth. Each object uses four parameters k ≤ m ≤ o ≤ n: k = minimum pieces to reconstruct, m = repair threshold, o = optimal/durability target at which excess uploads are canceled (long-tail mitigation), n = total pieces generated. Data is split files → segments (fixed max size; small ones stored "inline" on the Satellite) → stripes (erasure-encoding + audit unit, a few KB) → erasure shares → pieces (same-index shares concatenated; one piece per node). Over-encoding lets uploads/downloads complete from only the fastest nodes, turning variable node performance into a strength.

Encryption is client-side and authenticated (AES-GCM or NaCl Secretbox/Salsa20+Poly1305), applied in ≤4KB encryption blocks with monotonically increasing, segment-deterministic nonces. Each segment/file uses a distinct key so access can be shared granularly. Paths are encrypted hierarchically and deterministically (BIP32-style HMAC chain), so a subtree can be shared without exposing parents. Satellites never see plaintext or keys—only existence, rough size, and access patterns. Authorization uses macaroons (caveatable bearer tokens with expirations and revocation), letting access be delegated to path prefixes and specific operations without central coordination.

Trust in untrusted nodes is enforced by audits and reputation. Rather than pre-generated Merkle-tree proofs of retrievability (vulnerable to nodes that store only expected responses), Satellites audit by requesting a random stripe's erasure shares from all responsible nodes and running Berlekamp-Welch error correction to detect faulty/missing responses without pre-generated challenges. Non-responsive nodes enter "containment mode" (re-challenged on the same audit until they pass or are disqualified). Reputation has four subsystems: proof-of-work identity (trailing-zero node-ID hash, Sybil resistance), a vetting period for new nodes (given small data fractions, with earnings partly held for ~9 months), a filtering/disqualification system (too many failed audits/uptime checks → data moved, held funds forfeited, re-vetting required), and a preference system (Power-of-Two-Choices load balancing toward better nodes).

Data repair detects segments whose live pieces drop below m, downloads just enough pieces to reconstruct, regenerates the missing pieces onto new nodes, and updates the pointer; piece hashes (validated against a hash-of-hashes in the pointer) let repair trust minimal downloads. Payments avoid blockchain hot-path dependency: clients pay Satellites (any method), Satellites pay nodes monthly in the Ethereum ERC-20 STORJ token, with held funds discouraging churn. Bandwidth is metered via Neuman's proxy-based accounting—Satellite-signed "order limits" and Uplink-signed incremental "orders" (restricted bandwidth allocations) that make cheating protocol-impossible and bound either party's loss, inspired by Filecoin's off-chain retrieval market. Garbage collection (Bloom filters) reclaims space from missed deletes. Appendices justify avoiding Byzantine consensus (it conflicts with coordination avoidance and exabyte scale) and analyze attacks (Spartacus, Sybil, Eclipse, Honest Geppetto, Hostage bytes, faithless nodes).

## Key takeaways
- Decentralized S3-compatible object storage built as eight swappable framework components plus a concrete implementation.
- Three peer classes: storage nodes (store pieces), Uplinks (clients doing encryption + erasure coding), Satellites (trusted-but-blind metadata/coordination/payment layer).
- Reed-Solomon erasure coding (k,m,o,n) replaces replication—durability decoupled from expansion factor, low repair bandwidth, long-tail elimination by canceling slow transfers after o pieces.
- Client-side authenticated encryption + hierarchical deterministic (BIP32-style) path encryption; Satellites never hold plaintext or keys.
- Audits use random-stripe retrieval + Berlekamp-Welch correction (no pre-generated challenges), plus containment mode and a four-part reputation system with proof-of-work IDs and a vetting period.
- Coordination avoidance is a first-class goal: no blockchain on the hot path, no Byzantine consensus; metadata is partitioned per-user/per-Satellite.
- Bandwidth accounting via signed, incrementally-restricted allocations (order limits/orders) makes cheating protocol-impossible.
- Payments in ERC-20 STORJ token, monthly, with held funds (~half a year) to penalize churn; nodes paid for at-rest + egress, never ingress.

## Concepts & entities
Wikilinks: [[storj]], [[decentralized-storage]], [[object-storage]], [[erasure-coding]], [[reed-solomon]], [[data-durability]], [[redundancy]], [[proof-of-storage]], [[audits]], [[reputation-system]], [[s3-api]], [[metadata-management]], [[content-addressing]], [[coordination-avoidance]], [[byzantine-fault-tolerance]], [[bar-model]], [[client-side-encryption]], [[hierarchical-deterministic-encryption]], [[macaroons]], [[proof-of-work]], [[sybil-attack]], [[long-tail-mitigation]], [[berlekamp-welch]], [[bandwidth-allocation]], [[garbage-collection]], [[satellite]], [[storage-node]], [[uplink]], [[storj-token]], [[filecoin]].

## Notable claims / data
- A (k=20, n=40) RS code gives ~99.99999980% monthly durability at 10% node churn vs. replication needing n=16 for similar—erasure coding raises durability without raising expansion factor.
- Durability modeled via Poisson CDF P(D)=e^(-λ)·Σ λ^i/i! with λ=pn over n-k; (k=20,n=40) spreads risk better than (k=10,n=20) at identical 2x expansion.
- Repair-cost framework (from "be lazy" P2P guide): loss rate increases exponentially with churn α, requiring node lifetimes of several months for acceptable durability.
- Decision tables: e.g. MTTF 6 months, (k=20,n=40,m=30) → repair bandwidth ratio 0.87, ~17 nines; larger n lets m sit closer to k, lowering rebuilds/month and repair bandwidth.
- Default typical setup cited: 20/40 RS spread across 40 drives in a network of 100,000+ nodes (v2 had 150,000+); files encrypted with 256-bit AES, keys held only by user.
- Node ID = hash of CA public key; proof-of-work measured by trailing zero bits to deter Sybil attacks. Leaf certs are timestamped/revocable.
- Audit confidence via Bayesian beta-posterior estimator; ~200 successful audits → ~0.995 (uniform prior) / ~0.998 (Jeffrey's prior) success estimate.
- Example metadata cost: 1 EB of 50MB objects (n=40) ≈ 20B objects ≈ 55TB metadata; heavily partitionable per user (100TB → 5.5GB).
- Bandwidth caps (e.g. Comcast 1TB/month ≈ 385 KB/s sustained) constrain usable space: paid + repair bandwidth must stay under the node's cap.
- Transport: custom DRPC (gRPC-like) over TLS or Noise (IK mode) over TCP or QUIC; Noise reduces handshake round-trips when data is already encrypted.
- Assumes objects ≥4MB; small files supported but costlier (recommend packing). Vetting holds a portion of earnings for first ~9 months.

## Questions raised
- The Satellite must stay online indefinitely or repairs stop and data decays from churn (Kademlia-like republishing dependency)—how is this single-point-of-availability reconciled with full decentralization claims?
- Initial implementation does not detect/mitigate a misbehaving Satellite's file removal or rollback (relies on trust); how robust is this against malicious Satellite operators?
- Reputation is per-Satellite at launch ("global over time")—what is the path and mechanism to shared/global reputation without reintroducing coordination?
- The long-term goal to "architect the Satellite out" depends on a fast, scalable Byzantine-fault-tolerant consensus algorithm "should one arise"—what concrete candidate or timeline?
- How well do the erasure/durability models hold under correlated failures (Honest Geppetto, related operators) that violate the independence assumption underpinning availability claims?
- Move operations and precise garbage collection are deferred; what are the real-world performance/consistency impacts of conservative GC and atomic pointer batch updates?
