---
title: Ethereum
type: entity
tags: [blockchain, smart-contracts, web3]
created: 2026-06-19
updated: 2026-06-19
status: active
sources: [swarm-whitepaper, book-of-swarm]
---
# Ethereum

**What it is:** A smart-contract blockchain hosting Swarm's incentive contracts and the Ethereum Name Service.

## Notes
Ethereum is the settlement and contract layer that makes [[swarm]] economically self-sustaining: Swarm is conceived as the "hard disk" of the Ethereum world computer, with its bandwidth ([[swap-protocol]] chequebooks) and storage ([[postage-stamps]], [[redistribution-game]]) incentives enforced by smart contracts, and naming via [[ens]]. Swarm node overlay addresses are derived from Ethereum public keys.

More broadly, Ethereum is the canonical reference point for the on-chain incentive mechanisms across [[storage-incentive-mechanisms]] — [[storj]] settles its STORJ token as an ERC-20 on Ethereum. In practice Swarm's production storage/redistribution contracts and BZZ settlement run on [[gnosis-chain]] (an EVM chain) rather than Ethereum mainnet.

## Sources
- [[swarm-whitepaper]] — Ethereum smart contracts + BZZ powering Swarm incentives
- [[book-of-swarm]] — overlay addresses from Ethereum keys, contract suite, ENS
