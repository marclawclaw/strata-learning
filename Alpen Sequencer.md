---
tags: [alpen, component, centralization]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Alpen Sequencer

The **centralized** EVM block producer of [[Alpen]]. Alpen Labs runs it on testnet. It is built on Reth.

- It orders its mempool with Reth's default policy: highest effective gas price first, respecting nonce order.
- It builds a block every **~5 s** and processes pending deposits and withdrawal requests in it.
- Blocks at this point are only **sequencer pre-confirmed** ([[Alpen Finality Levels]]).
- Every **1,440 blocks (~2 h)** it seals an [[Alpen Batch]], posts its DA, proves it, and submits it to Strata.

Consensus is listed as **Proof of Authority** ([[Alpen EVM Compatibility]]).

Related: [[Strata Sequencer]], [[Centralization Points]]
