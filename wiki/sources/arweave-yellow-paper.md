---
title: "Arweave: A Protocol for Economically Sustainable Information Permanence"
type: source
tags: [arweave, decentralized-storage, blockchain, mechanism-design, permanent-storage, proof-of-access, permaweb]
created: 2026-06-19
author: Sam Williams, Viktor Diordiiev, Lev Berman, India Raybould, Ivan Uemlianin
source_url: "local: raw/arweave_yellow_paper.md"
source_type: paper
ingested: 2026-06-19
---
# Arweave: A Protocol for Economically Sustainable Information Permanence

**One-line:** Arweave is a mechanism-design-driven protocol that uses a graph-structured "blockweave" and Proof of Access consensus to provide permanent, low-cost, decentralized on-chain data storage funded by a one-time fee and a self-sustaining storage endowment.

## Summary
Arweave is positioned as a blockchain-like protocol whose core function is permanent, scalable on-chain data storage rather than value transfer. Its central data structure is the [[blockweave]]: unlike a singly-linked chain, each block links to two prior blocks — the immediately previous block (as in normal blockchains) and a deterministically-but-unpredictably chosen "recall block" from the weave's history. The recall block height is derived as `BHL[indep_hash mod height]`, so it cannot be predicted before the prior block is mined. This blockweave underpins the [[permaweb]], the array of websites and decentralized applications hosted directly on-chain and served to ordinary web browsers.

Consensus runs on [[proof-of-access]] (PoA), an extension of Proof of Work where the entire recall block's data is folded into the material hashed for the PoW puzzle. To mine or validate a block a node must therefore possess that block's recall block, which directly incentivizes miners to store data. PoA takes a probabilistic, competition-based approach to replication rather than the fixed contract-based replica counts used by networks like [[ipfs]]/Filecoin: the algorithm rewards storing *rare* blocks more (fewer competitors for the same reward) in unhealthy networks, and pushes toward heavy redundancy in healthy ones. A node's probability of winning a block is the product of the fraction of historical blocks it stores and its relative hashing power. The reference miner uses [[randomx]] as its hashing algorithm.

To make the blockweave scale beyond any single node's capacity, Arweave memoises network state into each block ([[memoisation-of-state]]). Each block carries a Block Hash List (BHL) and Wallet List (WL), so new nodes can join by downloading just the current block, and transactions can be validated without holding the block where a wallet last transacted. There is no hard full/light client distinction — only nodes storing more or less of the weave, with higher PoA rewards as an optional upgrade path for storing more. Block propagation uses [[blockshadows]] (building on Graphene and BIP-152 compact blocks): instead of gossiping full blocks, nodes send a few-kilobyte "shadow" of transaction IDs plus WL/BHL hashes, from which peers reconstruct the full block from mempool transactions, decoupling block size from fork risk.

The economic model is the paper's distinctive contribution. Users pay a one-time AR fee to store data forever; most of that fee flows into a [[storage-endowment]] rather than directly to the miner. Perpetual storage cost is modelled as the infinite sum of *declining* future storage costs, relying on the historically observed ~30.57% average annual decline in GB-hour storage cost over ~50 years. A miner's per-block reward combines instant transaction fees, a predetermined inflation reward (decaying as `GAR * 0.2 * ln2 * 2^(-BH/By)`), and — only when fees plus inflation fall short of the cost of maintaining the weave for that period — a draw from the endowment, scaled by recall-block size relative to average block size. Token supply: 55M AR at genesis (8 June 2018) plus 11M minted as rewards, capping at 66M AR; the smallest unit is the Winston (1 AR = 10^12 Winston).

Beyond hard consensus, Arweave layers an off-chain [[adaptive-interacting-incentive-agents]] (AIIA) meta-game for pro-social behaviour. Nodes privately rank peers by generosity and responsiveness and gossip preferentially to high-ranked peers — a generalization of BitTorrent's optimistic tit-for-tat. The reference agent is [[wildfire]]. Crucially, AIIA is *not* consensus-enforced: each node may run a different ranking strategy, allowing the network to adapt to new conditions (the way BitTorrent adapted to the BitTyrant exploit) without protocol-wide upgrades. Limitations: AIIA can only encode behaviours the majority already favours, cannot shift consensus rules, and adapts slowly if nodes under-express pro-social bias.

Storage is not mandatory: [[content-policies]] let each miner decide, via arbitrary computation (substring/hash matching, or e.g. PhotoDNA), which transactions to accept and store, producing a stochastic "voting" process during which content-rejecting and content-accepting nodes race to mine the next block. This makes the network censorship-resistant while letting nodes avoid storing unwanted data. [[gateway-node]]s extend this to indexing and serving, exposing ArQL (a per-node, non-globally-consistent query language over transaction tags) and DNS/TLS integration so permaweb apps can run on custom domains with browser sandboxing. The team frames Arweave as offering *data* permanence, not *network* permanence — expecting the weave's data to eventually be "subsumed" into a successor system, as Gopherspace was archived into the permaweb.

## Key takeaways
- The blockweave links each block to both the previous block and an unpredictable historical "recall block"; possessing the recall block is required to mine, making storage a prerequisite for mining rewards.
- Proof of Access folds recall-block data into the PoW hash, turning consensus into a probabilistic, competition-driven replication mechanism instead of fixed contract-based replica counts.
- Storage is paid once: most of the fee funds a perpetual storage endowment, justified by a modelled infinite sum of historically-declining (~30.57%/yr) storage costs.
- State memoisation (Block Hash List + Wallet List per block) lets nodes join cheaply and store only part of the weave, while blockshadows decouple block size from fork probability.
- AIIA / wildfire is an off-chain, non-consensus tit-for-tat meta-game rewarding responsiveness and pro-sociality; nodes may each run different strategies, giving the network adaptability.
- Decentralized content policies + gateways give per-node and per-user control over what is stored and shown, yielding censorship resistance without forced storage.
- AR supply is capped at 66M (55M genesis + 11M rewards); 1 AR = 10^12 Winston; mining uses RandomX.

## Concepts & entities
Wikilinks: [[arweave]], [[blockweave]], [[proof-of-access]], [[permaweb]], [[storage-endowment]], [[recall-block]], [[memoisation-of-state]], [[blockshadows]], [[adaptive-interacting-incentive-agents]], [[wildfire]], [[content-policies]], [[gateway-node]], [[arql]], [[decentralized-storage]], [[blockchain]], [[mechanism-design]], [[permanent-storage]], [[content-addressing]], [[merkle-tree]], [[endowment]], [[proof-of-work]], [[randomx]], [[ipfs]], [[nakamoto-consensus]], [[dsic]].

## Notable claims / data
- Token economy: 55M AR at genesis (launch 8 June 2018) + 11M AR (20%) as gradual mining rewards = 66M AR max supply; 1 AR = 1,000,000,000,000 Winston; wallet creation fee of 25 AR acts as a spam filter for active wallets.
- Cost model: `PGBH = HDDprice / (HDDsz * HDDmtbf)`; perpetual price = infinite sum of `Datasize * PGBH[i]`; GB-hour storage cost has declined ~30.57%/yr average over ~50 years.
- Inflation reward per block: `GAR * 0.2 * ln2 * 2^(-BH) / Bny`, with GAR = 55,000,000 and By = 262,800 blocks/year; total miner reward = fees + inflation + (conditional) endowment draw.
- Endowment is only drawn when `Wsize * PGBB > Rinflation + Rfees`; the draw is scaled by recall-block size / average block size to even out storage incentives.
- Win probability: `P(win) = (Blocks_local / B_height) * (HP_local / HP_net)` — storage share times relative hashing power.
- Recall block selection: `BHrecall = BHL[B_indep_hash mod B_height]`, deterministic but unpredictable until the prior block is mined; precludes the current block being its own recall block.
- Fork resolution uses a `cumulative_difficulty` field (Nakamoto-style heaviest-work selection); transactions from rejected forks are re-pooled unless present in the 3 most recent blocks.
- Scale figures: ~97% replication rate; ~750 replications per piece of data; ~310M typical-size data items storable per year (as of block 223,280); permaweb growing ~2,500 apps/pages per day, ~500 days live at writing.
- Block uses SHA-384 (Block Data Segment, Hash List Merkle Root, Deep Hash) and SHA-256 (wallet addresses, transaction IDs); transactions signed RSA-SHA256; max transaction data 10,485,760 bytes (~10 MB).
- Future work: Succinct Proofs of Access (store only Merkle roots, trading data-availability guarantees for capacity), append-only wallet logs (reducing O(blocks*wallets) to O(txs+blocks)), and Fast Find routing agents.

## Questions raised
- Does the assumption of perpetually declining storage cost (~30%/yr) hold over the centuries the endowment must fund, and what happens economically if the trend stalls or reverses?
- How robust is the endowment to sustained AR price collapse, given rewards are denominated in a volatile token but costs are in fiat?
- With AIIA being non-consensus and majority-driven, how vulnerable is pro-social behaviour to coordinated selfish or Sybil agents, and how is "majority" measured off-chain?
- Succinct Proofs of Access remove proof-of-distribution guarantees — how is real data availability assured for large datasets that may never be fully seeded?
- Content policies provide censorship resistance via miner choice, but how is durable availability of unpopular-but-legal content guaranteed if most miners reject it?
- How does ArQL's lack of global consistency and inability to guarantee complete results affect applications that depend on it as a primary index?
