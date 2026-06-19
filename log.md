# Log — tung-kb

Append-only, chronological. Newest at the bottom. Each entry starts with `## [YYYY-MM-DD] <op> | <title>`
so the timeline is greppable: `grep "^## \[" log.md | tail -5`.

## [2026-06-13] init
Initialized the LLM Wiki. Created schema (AGENTS.md), index.md, log.md, and folder structure
(raw/, raw/assets/, wiki/{sources,concepts,entities,topics,questions}). Domain: research, deep technical
topics, and software engineering.

## [2026-06-19] ingest | Bulk ingest of raw/ — decentralized storage & content-addressing
Ingested the entire `raw/` corpus (14 logical sources) covering decentralized storage networks and their
content-addressing primitives. Multi-file tutorials/specs were treated as single logical sources
(ProtoSchool tutorials; the Filecoin spec's ~200 files → one source page).

Sources created (14): [[what-is-raid]], [[erasure-coding-vs-raid]], [[object-storage-domain-knowledge]],
[[storj-v2]], [[storj-v3]], [[arweave-yellow-paper]], [[arweave-2-9]], [[book-of-swarm]],
[[swarm-whitepaper]], [[swarm-protocol-spec]], [[content-addressing-tutorial]], [[merkle-dags-tutorial]],
[[anatomy-of-a-cid-tutorial]], [[filecoin-spec]].

Wiki graph built: 56 concept pages, 24 entity pages, 5 topic syntheses
([[decentralized-storage-networks]], [[content-addressed-data-structures]],
[[storage-redundancy-and-durability]], [[proof-of-storage-systems]], [[storage-incentive-mechanisms]]).
index.md repopulated.

Notes / decisions: resolved two filename collisions (Obsidian resolves wikilinks by basename) — kept
[[multiformats]] as an entity (dropped the duplicate concept), and renamed the RAID *article* source to
[[what-is-raid]] to free the `raid` slug for the [[raid]] concept. Contradictions surfaced and recorded on
topic pages: erasure-coding-vs-unique-replication for durability (Storj vs Filecoin), credibility of
Arweave's pay-once endowment, and coordination-avoidance vs full decentralization (Storj satellites).
Dangling links left intentionally (e.g. `[[proof-of-work]]`, `[[walrus]]`-detail, `[[retrieval-market]]`,
`[[payment-channels]]`, `[[scrubbing]]`) — promote to pages when they earn one.

## [2026-06-19] ingest | Deepen sync protocols (Swarm / Filecoin / Storj)
Filled the depth gaps in sync coverage flagged during a query thread. Deep-read previously-unmined raw
files and wrote 6 spec-level concept pages + 1 topic synthesis.

Concepts created: [[pull-sync]], [[push-sync]], [[retrieval-protocol]] (from [[swarm-protocol-spec]] +
[[book-of-swarm]] — exact protobuf schemas, interval store, offer/want, peer roles, epoch fingerprint);
[[chainsync]], [[graphsync-data-transfer]] (from filecoin-spec block_sync/gossip_sub/data_transfer —
BlockSync wire protocol, BV0–BV8 validation, GraphSync vs Bitswap, voucher-bound transfer);
[[storage-repair]] (from [[storj-v3]]/[[storj-v2]] — repair threshold ≈ m in k≤m≤o≤n, Poisson durability
model, satellite-orchestrated reconstruct→redistribute pipeline).

Topic created: [[replica-sync-and-data-transfer]] — synthesizes the three distinct meanings of "sync"
and contrasts Swarm peer anti-entropy vs Storj orchestrated repair vs Filecoin no-auto-repair.

Key correction surfaced and recorded: "the network heals lost replicas" is true for Swarm/Storj but
FALSE for Filecoin (each replica is an independently sealed sector; durability = incentives + client-side
redundancy). New dangling link `[[forwarding-kademlia]]` introduced (Swarm's recursive routing variant) —
a reasonable future page. index.md updated with a new "Sync, transfer & repair" concept section.
