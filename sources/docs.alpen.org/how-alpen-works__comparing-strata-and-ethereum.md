---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/how-alpen-works/comparing-strata-and-ethereum
---

# Comparing Alpen and Ethereum

## EVM compatibility <a href="#evm-compatibility" id="evm-compatibility"></a>

Alpen is an EVM-compatible blockchain. The execution client used by the Alpen batch producer is based on [Reth](https://github.com/paradigmxyz/reth) and is fully compatible with the EVM spec up to the [Prague](https://github.com/ethereum/execution-specs/tree/forks/osaka?tab=readme-ov-file#ethereum-protocol-releases) release, excluding type 3 transactions and blob-related upgrades (EIP-4844). With these exclusions, Alpen can be considered a [Type 2.5](https://vitalik.eth.limo/general/2022/08/04/zkevm.html) zkEVM.

## Blockchain functionality <a href="#blockchain-functionality" id="blockchain-functionality"></a>

Comparing differences in how Alpen and Ethereum functionally operate:

|                  | Alpen                                                                                | Ethereum       |
| ---------------- | ------------------------------------------------------------------------------------ | -------------- |
| Consensus:       | Proof of Authority                                                                   | Proof of Stake |
| Gas asset:       | BTC                                                                                  | ETH            |
| Block gas limit: | 36,000,000                                                                           | 60,000,000     |
| Block time:      | \~5s                                                                                 | \~12s          |
| Finality:        | Probabilistic; blocks are confirmed in checkpoints on bitcoin signet every 10 hours. | \~15m          |


---

