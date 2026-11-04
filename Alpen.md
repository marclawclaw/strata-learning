---
tags: [alpen, concept]
aliases: [Alpen Network, Alpen rollup]
source: https://docs.alpen.org/welcome/readme
---
# Alpen

Alpen is a **work-in-progress, EVM-compatible validity rollup on bitcoin**, built by Alpen Labs.

- **EVM-compatible**: the sequencer runs a fork of Reth. See [[Alpen EVM Compatibility]].
- **Validity rollup**: every state transition is proven with a SNARK. See [[SNARK Account Update Proof]]. The docs use the "rollup" label, but see [[Alpen is Closer to a Validium]].
- **On bitcoin**: bitcoin provides consensus and data availability. Once a [[Strata Checkpoint]] is confirmed, the only way to reorg the covered Alpen state is to reorg bitcoin. See [[Bitcoin Roles in Alpen]].

The goal is **programmable BTC**: arbitrary locking conditions such as vaults, subscriptions, P-256 signatures, ZK privacy, and lending, limited only by Alpen's block size and gas.

Alpen is not a standalone chain. It is a [[SNARK Account]] on the [[Strata Orchestration Layer]], and it reaches BTC through the [[Strata Bridge]].

Status: running on bitcoin **signet** (Testnet III). See [[Alpen Project Status]].

Related: [[Four-Layer Architecture]], [[Execution and Orchestration Separation]]
