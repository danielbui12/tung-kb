---
title: The Book of Swarm
type: source
tags: [decentralized-storage, swarm, kademlia, content-addressing, storage-incentives, peer-to-peer, web3]
created: 2026-06-19
author: Viktor Trón
source_url: "local: raw/swarm/the-book-of-swarm-2.md"
source_type: paper
ingested: 2026-06-19
---
# The Book of Swarm

**One-line:** A comprehensive design compendium for [[swarm]] — a permissionless, incentivised storage and communication layer for Web3 built on a Kademlia overlay of fixed-size content-addressed chunks with on-chain storage incentives.

## Summary

Swarm is conceived as the "hard disk" of the Ethereum world computer: a peer-to-peer, censorship-resistant, privacy-preserving storage and messaging substrate for a self-sovereign data economy. Its layered architecture stacks (1) an underlay transport (libp2p), (2) an overlay network providing a distributed immutable store of chunks, (3) a high-level data-access layer (files, manifests, feeds, access control), and (4) an application layer. The book's technical core is layers 2 and 3 plus the economic incentive system that makes the network self-sustaining.

Each node has a stable 256-bit *overlay address* derived by hashing its Ethereum public key with the [[bzz-token]] network ID. Nodes organise into a [[kademlia]] topology over *proximity order* (PO — the count of matching leading bits between two addresses, the discrete inverse of XOR distance). Unlike most Kademlia implementations, Swarm uses *forwarding (recursive) Kademlia* rather than iterative: each node relays a message to a connected peer at least one PO closer to the destination, guaranteeing logarithmic hops and connections. The reverse path ("backwarding") returns responses along the same route, which delivers complete sender/requestor anonymity. Nodes maintain saturated Kademlia tables (multiple peers per bin) to survive churn, and denser bins allow multi-order hops.

Storage is the [[distributed-hash-table]]-derived DISC (Distributed Immutable Store for Chunks): rather than storing pointers to seeders, the network stores the [[chunk]] content itself with the nodes closest to the chunk address. Chunks are fixed at max 4KB payload plus an 8-byte span. Two chunk types exist: *content-addressed chunks*, whose address is computed via the [[binary-merkle-tree]] hash (BMT — Keccak256 over 32-byte segments, zero-padded to 4096 bytes, with the span prepended), enabling compact 32-byte-resolution inclusion proofs; and [[single-owner-chunk]]s, whose address is `Keccak256(identifier + owner)` with an EC signature attesting the identifier–payload binding, giving owners a writable subspace of the address space. Chunks are encrypted by default (Keccak256 counter-mode stream cipher) for confidentiality, plausible deniability, and address mining. Redundancy comes first from neighbourhood replication: nearest-neighbour sets (target ~4 nodes) all replicate chunks in their shared area of responsibility, giving "eventual consistency" under churn.

Chunk movement uses three protocols. *Retrieval*: a retrieve request forwards toward the chunk address and the chunk backwards along the same path. *Push-syncing*: an uploaded chunk is forwarded to its closest storer, which returns a signed *statement of custody* receipt. *Pull-syncing*: neighbours continuously stream chunks per PO bin (offer/want round-trips keyed by address) to restore replication and fill spare capacity. There is no explicit "not found"; requests stay open until TTL, enabling opportunistic caching that turns Swarm into an auto-scaling CDN for popular content.

Bandwidth incentives use [[swap-protocol]] (Swarm Accounting Protocol): peers keep a running tit-for-tat balance, and when the debt crosses a payment threshold the debtor sends a *cheque* — an off-chain signed commitment redeemable at the issuer's *chequebook* smart contract, with double-cashing prevented via cumulative totals/serial numbers. *Waivers* offset opposing cheques to save gas; a third party can cash cheques for a node with no Ether, enabling zero-cash entry. Differential pricing by proximity (delivery costs more the farther the peer is from the chunk) prevents free DoS and drives price arbitrage toward uniform, monotonic, marginal-cost pricing.

Storage incentives center on [[postage-stamps]]: uploaders buy a *postage batch* on-chain (paying BZZ that is spent as per-block storage rent via a price oracle), then attach stamps (batch ID, storage slot in a power-of-2 bucket grid, timestamp, owner signature) to chunks. Batch *depth* sets issuance volume; *uniformity depth* arranges slots into buckets so over-issuance and targeted-DoS skew are locally detectable and exponentially costly. Storer nodes keep a fixed-size *reserve* governed by the "rules of the reserve" (upward-closed by PO and by per-chunk balance). Three depths — reserve depth (potential demand), storage depth (effective demand), neighbourhood depth (effective supply) — characterise network health. The [[redistribution-game]] redistributes accumulated rent: a [[schelling-game]] run by four contracts (postage, game, staking, price oracle) over commit/reveal/claim phases. A random neighbourhood-selection anchor picks a winning neighbourhood; staked nodes commit to a *reserve sample* (first k chunks by salted modified BMT hash), and an honest-majority Schelling consensus, weighted by stake, selects truth, pays the winner the whole pot probabilistically, and freezes/slashes liars and saboteurs. The "eight rules of entitlement" (replication, redundancy, responsibility, relevance, retention, recency, retrievability, resources) are enforced via inclusion proofs and a proof-of-density. The number of honest revealers serves as a price signal feeding the oracle (`p_{t+1} = 2^{σ(4−r)} p_t`). A separate *negative-incentive* insurance layer — "swap, swear, and swindle" — lets staked insurers issue secured storage receipts and be litigated (deposit largely burned) for losing insured chunks.

Higher-level structures build on the DISC. The *Swarm hash tree* packs chunk references into intermediate chunks (branching factor 128 unencrypted / 64 encrypted) to represent arbitrary-length files with random access, append, length-preserving edits, and logarithmic inclusion proofs. *Manifests* (compacted tries) map paths to files, implementing websites, directories, and key–value stores, served via the `bzz` URL scheme with ENS name resolution. Access control uses chunk-level encryption plus an access-control trie (ACT) that grants multiple parties access via session-key-derived lookup/access keys, supporting passphrase or ECDH credentials and hierarchical roles. *Feeds* give mutability over the immutable store using single-owner chunks indexed by topic+index (simple, epoch-based for sporadic updates, or double-ratchet nonces), implementing persisted pull-messaging/pub-sub, authoritative version history, and real-time fork-finality integrity checks.

[[pss]] is direct push messaging: a *Trojan chunk* disguises an asymmetrically-encrypted message as ordinary chunk traffic by mining a nonce so its content address lands in the recipient's neighbourhood, giving zero-leak (undetectable) messaging plus automatic mailboxing for offline recipients. *Addressed envelopes* (mined single-owner-chunk addresses with pre-paid postage) enable push notifications, free contact vouchers, and targeted chunk delivery. For persistence, Swarm adds [[erasure-coding]] (Reed–Solomon, systematic, m-of-n per packed-address-chunk, with retrieval strategies NONE/DATA/PROX/RACE), *dispersed replicas* (single-owner-chunk copies of singleton root chunks scattered across address space), local/global *pinning* with the `prod` recovery protocol, and the *dream* construct — a network-computed one-time-pad scheme giving revocable/deletable content over the immutable store.

## Key takeaways
- Swarm stores *content itself* at the closest nodes (DISC), not seeder pointers like classic DHTs/IPFS.
- Forwarding (recursive) Kademlia + backwarding gives logarithmic routing AND built-in sender/requestor anonymity.
- Two chunk primitives: content-addressed (BMT hash) and single-owner (signed, writable subspace) — feeds and PSS are built on single-owner chunks.
- Bandwidth is paid instantly via SWAP cheques/chequebook (off-chain commitments, on-chain settlement); storage is paid up-front via postage stamps and redistributed by an on-chain Schelling game.
- Differential pricing by proximity is the load-bearing mechanism preventing free DoS and aligning forwarding incentives.
- Three depths (reserve / storage / neighbourhood) jointly express demand vs. supply and drive automatic price discovery.
- Persistence is layered: neighbourhood replication → erasure coding/dispersed replicas → pinning + recovery → optional staked insurance.
- Everything is encrypted by default, enabling confidentiality, plausible deniability, and address mining.

## Concepts & entities
Wikilinks: [[swarm]], [[kademlia]], [[distributed-hash-table]], [[content-addressing]], [[chunk]], [[binary-merkle-tree]], [[single-owner-chunk]], [[postage-stamps]], [[swap-protocol]], [[redistribution-game]], [[schelling-game]], [[pss]], [[erasure-coding]], [[decentralized-storage]], [[proof-of-storage]], [[bzz-token]], [[gnosis-chain]], [[proximity-order]], [[forwarding-kademlia]], [[swarm-feeds]], [[swarm-manifest]], [[chequebook-contract]], [[trojan-chunk]], [[dispersed-replicas]], [[ens]].

## Notable claims / data
- Overlay address = 256-bit Keccak256 of EC public key + bzz network ID; proximity order ranges 0–256.
- Content-addressed chunk: max 4096-byte payload + 8-byte little-endian span; BMT over 32-byte segments, Keccak256 base hash, zero-padded to 4KB.
- Single-owner chunk content: 32-byte identifier + 65-byte (r,s,v) EC signature + 8-byte span + ≤4096-byte payload; address = Keccak256(identifier + owner).
- Swarm hash tree branching factor: 128 (unencrypted, 32-byte refs) / 64 (encrypted, 64-byte refs = hash + key).
- Target neighbourhood redundancy r ≈ 4 nodes; neighbourhood depth = highest PO whose prefix range holds ≥3 other peers.
- Postage batch fields: batch ID, batch depth (log2 issuance), owner, per-chunk balance, mutability flag, uniformity depth (log2 buckets); bucket index 0..2^u−1, within-bucket 0..2^(d−u)−1.
- Storage incentive contract suite: postage, game, staking, price oracle. Redistribution round phases commit/reveal/claim (¼, ¼, ½ of round); only the winner submits the expensive claim.
- Price update rule: `p_{t+1} = 2^{σ(4−r)} · p_t`; honest revealers r capped at r_max = 8; price floored at p_min; optimum 4 nodes per neighbourhood.
- Reserve sample = first k chunks ordered by Keccak256 BMT hash salted with round nonce, read big-endian.
- Insurance ("swap, swear, swindle"): ≥95% of a guilty insurer's deposit is burned to close the collude-and-exit attack.
- Reed–Solomon: m + k = b (128 or 64); any m of m+k recovers; systematic encoding keeps original data; retrieval strategies NONE/DATA/PROX/RACE.
- Dispersed replicas signed by fixed private key 0x0100…00 (address 0xdc5b20847f43d67928f49cd4f85d696b5a7617b5); SOC ID matches payload hash on first 31 bytes.
- PSS Trojan chunk: 8-byte span + 32-byte nonce + 4064-byte asymmetrically encrypted message (2-byte length + 32-byte obfuscated topic + message + random pad); mining cost ≈ network size; retrieve-request TTL ≈ 30s doubles as subscription.
- Dream reference: 4 segments — parity address, batch ID, initial generator g, decryption key; revocation by uploading chunks that change the bucket's closest chunk χ on any honest neighbourhood of the dream path.

## Questions raised
- The book describes the design intent; how closely does the deployed Bee client / Gnosis-Chain mainnet match (e.g., is the full insurance "swear/swindle" layer actually live)?
- Proof-of-density and reserve-sampling assume honest-majority neighbourhoods — what is the empirically observed neighbourhood infiltration rate, and how is the security parameter σ/step-count calibrated in practice?
- Differential proximity pricing assumes liquid peer competition; how does pricing behave in sparse early-network or geographically clustered conditions?
- The "dream" deletable-content construct depends on at least one honest neighbourhood per dream path and on batch under-utilisation — how robust is this against a well-funded archiving adversary at scale?
- How does Swarm's per-chunk economic model compare quantitatively (cost, latency, durability) against Filecoin/Arweave/Storj for real workloads?
