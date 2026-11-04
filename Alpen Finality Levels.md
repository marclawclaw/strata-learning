---
tags: [alpen, finality]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# Alpen Finality Levels

Alpen reuses Ethereum-like block tags:

| Level | Meaning | Trust |
|---|---|---|
| **Pre-confirmed / latest** | Included in an Alpen block by the [[Alpen Sequencer]] | The sequencer only |
| **Safe** | Covered by a batch committed in a [[Strata Checkpoint]] that is on bitcoin but not buried | Bitcoin, but reorgable |
| **Finalized** | The checkpoint's bitcoin block is buried to protocol depth | Bitcoin PoW |

The docs describe finality as "probabilistic", with checkpoints every ~10 h, compared with ~15 min on Ethereum ([[Alpen EVM Compatibility]]).

RPC: `alpen_getBlockStatus` returns `pending` / `confirmed` / `finalized` plus the `checkpoint_epoch` ([[Alpen RPC Methods]]). Strata's own view: `strata_getChainStatus` ([[Strata RPC Methods]]).

Related: [[Alpen Transaction Lifecycle]]
