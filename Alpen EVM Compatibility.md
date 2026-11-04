---
tags: [alpen, evm]
aliases: [Comparing Alpen and Ethereum]
source: https://docs.alpen.org/how-alpen-works/comparing-strata-and-ethereum
---
# Alpen EVM Compatibility

- Based on Reth. Compatible with the EVM spec **up to Prague**, *except* type-3 (blob) txs / EIP-4844 and the KZG precompile `0x0a`.
- So it is a **Type 2.5 zkEVM** in Vitalik's taxonomy.

| | Alpen | Ethereum |
|---|---|---|
| Consensus | Proof of Authority | PoS |
| Gas asset | **BTC** | ETH |
| Block gas limit | 36M | 60M |
| Block time | ~5 s | ~12 s |
| Finality | Probabilistic, checkpoints every ~10 h | ~15 min |

Related: [[Alpen Specifications]], [[Alpen Finality Levels]]
