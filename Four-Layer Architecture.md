---
tags: [alpen, strata, architecture]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Four-Layer Architecture

The Alpen system has four layers, bottom to top:

1. **Bitcoin**: data availability and settlement ([[Bitcoin Roles in Alpen]])
2. **Strata**: a lightweight orchestration chain that anchors state to bitcoin ([[Strata Orchestration Layer]])
3. **Alpen**: the EVM chain that executes user transactions and posts proven state commitments to Strata ([[Alpen]])
4. **Applications**: wallets, dapps, Foundry/Cast, and ERC-4337 bundlers

There are **two chains** (Strata and Alpen). Alpen batches go to Strata, and Strata checkpoints go to bitcoin.

| Component | Who runs it (testnet) |
|---|---|
| [[Strata Sequencer]] | Alpen Labs (centralized) |
| [[Alpen Sequencer]] | Alpen Labs (centralized) |
| Strata / Alpen full nodes | Anyone |
| Bundler | Alpen Labs provides one; anyone can run one |
| [[Bridge Operator]]s | A permissioned set (partners like Stakely, Chainflow) |

Cadence: Alpen block ~5 s → [[Alpen Batch]] ~2 h → Strata block ~60 min → [[Strata Checkpoint]] ~10 h.

Related: [[Execution and Orchestration Separation]], [[Centralization Points]]
