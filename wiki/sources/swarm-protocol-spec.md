---
title: Swarm Protocol Specification
type: source
tags: [swarm, p2p-protocols, libp2p, decentralized-storage, chunk-syncing, redistribution-game, wire-protocol]
created: 2026-06-19
author: Swarm Research Division
source_url: "local: raw/swarm/swarm_protocol_spec.md"
source_type: docs
ingested: 2026-06-19
---
# Swarm Protocol Specification

**One-line:** The wire-level specification of Swarm's libp2p-based peer-to-peer protocols for handshake, peer discovery, chunk retrieval, push/pull syncing, pricing, and SWAP settlement, plus the parameter constants and probabilistic machinery (price oracle, randomness, density-based reserve sizing) behind the storage-incentive redistribution game.

## Summary
The specification defines Swarm's networking from the underlay up. The underlay is libp2p, satisfying multiaddr addressing, dialing/listening, live full-duplex connections, TLS/secio channel security, mplex multiplexing, and bidirectional-stream delivery guarantees. Swarm fixes the node key type to ECDSA secp256R1 (keys serve both message signing and peer-id derivation), uses Protobuf with varint-delimited messages as the recommended serialization, and supports `dnsaddr` resolution (recursive TXT lookups, e.g. `/dnsaddr/mainnet.ethswarm.org`) to bootstrap underlay addresses. Every stream is identified by the convention `/swarm/{ProtocolName}/{ProtocolVersion}/{StreamName}` and must begin with a `Headers` protobuf exchange (repeated key/value pairs) before any protocol data flows.

On top of the underlay sit several named protocols, each with its own stream IDs and protobuf message schemas. The **Bzz handshake** (`/swarm/handshake/1.0.0/handshake`) is a TCP-style three-way Syn/SynAck/Ack exchange carrying `BzzAddress` (underlay, signature, overlay), `NetworkID`, full-node flag, nonce, and welcome message; mismatched network IDs or repeated handshakes terminate the connection. The **Hive** protocol (`/swarm/hive/1.1.0/peers`) exchanges `BzzAddress` lists (request/response plus broadcast notifications) so nodes maintain saturated Kademlia connectivity per proximity bin, persisting peers in an address book.

The retrieval and syncing protocols move chunk data along the Kademlia topology. **Retrieval** (`/swarm/retrieval/1.4.0/retrieval`) sends a `Request{Addr}` and gets a `Delivery{Data, Stamp, Err}` back, with forwarding to the storer neighborhood and backwarding the response along the same path; peer roles (origin, backwarder, multiplexer) differ in retry behavior. **Push-syncing** (`/swarm/pushsync/1.3.0/pushsync`) forwards a `Delivery{Address, Data, Stamp}` toward the destination neighborhood until a peer's proximity order (PO) to the chunk meets or exceeds its storage radius; the storer returns a signed `Receipt{Address, Signature, Nonce, Err}`. Shallow receipts (storer outside radius) trigger re-pushing; a 5-minute skiplist deduplicates chunk/peer attempts; multiplexers push to at most 2 neighbors in parallel. **Pull-syncing** (`/swarm/pullsync/1.3.0/{pullsync,cursors}`) is a subscription protocol where downstream peers request cursors (epoch-timestamped reserve fingerprint), then `Get{Bin, Start}` ranges, receive an `Offer{Topmost, Chunks}`, reply with a `Want{BitVector}`, and get `Delivery` messages. An interval store tracks synced (start, end) ranges across sessions to fill gaps; topology or epoch changes restart syncing.

Economic protocols round out the wire spec. **Pricing** (`/swarm/pricing/1.0.0/pricing`) announces per-PO `AnnouncePaymentThreshold` values; below-minimum thresholds disconnect the peer. **SWAP/pseudosettle** (`/swarm/pseudosettle/1.0.0/pseudosettle`) exchanges `Payment{Amount}` for `PaymentAck{Amount, Timestamp}` carrying outstanding debt. Blocklisting (temporary for accounting, permanent for invalid chunks) terminates and bans peers. Caching lets nodes retain popular or evicted chunks; garbage collection moves chunks from reserve to cache on stamp expiry or reserve overflow (evicting lowest-value batch first).

The appendices specify the storage-incentive layer's constants and math. Parameter constants include `BZZ_NETWORK_ID=0`, `PHASE_LENGTH=38` blocks, `ROUND_LENGTH=152` (4× phase), `NODE_RESERVE_DEPTH=23`, `SAMPLE_DEPTH=4`, `MAX_SAMPLE_VALUE≈1.2844e72`, `PRICE_2X_ROUNDS=64`, and `NHOOD_PEER_COUNT=4`. The **price oracle** sets unit storage price (PLUR/chunk/block) as an exponential function of supply: `Price(γ) = Price(Prev(γ))·2^(σ·d)` where `d = NHOOD_PEER_COUNT − |Reveals(γ)|` and `σ = 1/PRICE_2X_ROUNDS`. The **source of randomness** is the XOR of revealed obfuscation-key nonces from the redistribution game's commit-reveal scheme, secure as long as one honest party participates; skipped rounds reuse the prior seed but rotate the neighbourhood-selection anchor. **Density-based size estimation** (proof of resources) uses the fact that the kth-smallest of n uniform variates over the 256-bit address space follows an Erlang distribution (shape k, rate n+1); a threshold u on the reserve sample's last element enforces a minimum reserve cardinality, calibrated by minimizing α+β (recall/precision error rates) for sample sizes k = 8, 16, 32.

## Key takeaways
- Swarm's underlay is libp2p with mandated ECDSA secp256R1 keys, varint-delimited Protobuf serialization, and `/swarm/{Protocol}/{Version}/{Stream}` stream-ID naming; every stream opens with a `Headers` exchange.
- Six core wire protocols are specified with versioned stream IDs and protobuf schemas: handshake (1.0.0), hive (1.1.0), retrieval (1.4.0), pushsync (1.3.0), pullsync (1.3.0), pricing (1.0.0), pseudosettle (1.0.0).
- Chunk placement is governed by proximity order (PO) vs. storage radius: a chunk is stored when a peer's PO to the chunk address meets or exceeds its storage radius; shallow receipts trigger re-pushing.
- Forwarding (pushsync toward storer) and backwarding (response along the same path) are the two directional strategies over the keep-alive Kademlia network; peer roles (origin, forwarder/backwarder, storer, multiplexer) define distinct retry semantics.
- Pull-sync uses an offer/want bitvector handshake to save bandwidth and an interval store to fill gaps between syncing sessions; epoch timestamps fingerprint a peer's reserve so a wipe forces a fresh sync.
- The incentive layer rests on a redistribution game whose price oracle doubles price every `PRICE_2X_ROUNDS` per unit undersupply, draws randomness from XORed reveal nonces, and verifies reserve volume via an Erlang-distribution-based density estimate.

## Concepts & entities
Wikilinks: [[swarm]], [[libp2p]], [[kademlia]], [[content-addressing]], [[chunk]], [[proximity-order]], [[storage-radius]], [[push-sync]], [[pull-sync]], [[retrieval-protocol]], [[hive-protocol]], [[bzz-handshake]], [[postage-stamps]], [[swap-protocol]], [[redistribution-game]], [[price-oracle]], [[reserve-sampling]], [[density-based-size-estimation]], [[proof-of-resources]], [[bzz-address]], [[overlay-address]], [[underlay-address]], [[decentralized-storage]], [[binary-merkle-tree]], [[single-owner-chunk]], [[pss]], [[erasure-coding]].

## Notable claims / data
- Stream ID convention: `/swarm/{ProtocolName}/{ProtocolVersion}/{StreamName}`; all prefixed `/swarm`; versions are semver.
- Handshake messages: `Syn{ObservedUnderlay}`, `Ack{Address(BzzAddress), NetworkID, FullNode, Nonce, WelcomeMessage=99}`, `SynAck{Syn, Ack}`, `BzzAddress{Underlay, Signature, Overlay}`; mismatched network IDs terminate the connection.
- Hive `BzzAddress` additionally carries `Nonce=4`; peer underlays sent in fixed-size rate-limited batches.
- Retrieval: `Request{Addr=1}` → `Delivery{Data=1, Stamp=2, Err=3}`; non-empty Err triggers next-candidate retry.
- Pushsync: `Delivery{Address, Data, Stamp}` → `Receipt{Address, Signature, Nonce, Err}`; multiplexer pushes to max 2 peers in parallel; skiplist timeout 5 min; invalid chunk in response → blocklist.
- Pullsync messages: `Syn{}`, `Ack{Cursors[], Epoch}`, `Get{Bin, Start}`, `Chunk{Address, BatchID}`, `Offer{Topmost, Chunks[]}`, `Want{BitVector}`, `Delivery{Address, Data, Stamp}`; sync from storage radius up to bin 31; non-neighbours sync only the equivalent bin; pullsync auto-restarts after 5-minute inactivity.
- Pricing: `AnnouncePaymentThreshold{PaymentThreshold}`; below-minimum threshold disconnects the peer.
- SWAP/pseudosettle: `Payment{Amount}` → `PaymentAck{Amount, Timestamp}`.
- Parameter constants: `BZZ_NETWORK_ID=0`, `PHASE_LENGTH=38`, `ROUND_LENGTH=152`, `NODE_RESERVE_DEPTH=23`, `SAMPLE_DEPTH=4`, `MAX_SAMPLE_VALUE≈1.2844e72`, `MINIMUM_STAKE=10`, `MIN_STAKE_AGE=228`, `PRICE_2X_ROUNDS=64`, `NHOOD_PEER_COUNT=4`.
- Price function: `Price(γ) = Price(Prev(γ))·2^(σ·d)`, `d = NHOOD_PEER_COUNT − |Reveals(γ)|`, `σ = 1/PRICE_2X_ROUNDS`; unit is PLUR/chunk/block.
- Random seed: XOR of all revealed obfuscation-key nonces; skipped rounds keep prior seed but rotate the neighbourhood-selection anchor by rounds-since-last-reveal.
- Density estimation: kth-smallest of n uniform variates ~ Erlang(k, n+1); calibration table gives k=8/16/32 → u·2^256 ≈ 6.42e71 / 1.28e72 / 2.57e72 (α+β minimized at n=10^6, m=2·10^6).

## Questions raised
- The spec mentions a "light node capability" flag and "FullNode" but does not fully detail light-node behavioral differences across protocols.
- Kademlia peer-selection "choice strategy" within a single proximity bin is acknowledged as under-specified ("not well documented") — secondary selection / multiplexing rules among equidistant peers remain ambiguous.
- The postage-stamp format and validation rules are referenced (Stamp fields, multi-stamping) but the stamp data structure itself is not specified in this document.
- The relationship between `NODE_RESERVE_DEPTH`, `SAMPLE_DEPTH`, and `MAX_SAMPLE_VALUE` in the live proof-of-resources check could be made more concrete with a worked example.
- How exactly the price oracle smart-contract input (revealer counts, skipped rounds) is fed on-chain is described conceptually but not at the contract-interface level.
