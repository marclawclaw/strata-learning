---
tags: [alpen, concept]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# Alpen Batch

Every ~2 h (1,440 Alpen blocks), the [[Alpen Sequencer]] seals blocks into a **batch** and does three things:

1. **Posts DA**: puts the batch's **aggregated state diff** in a bitcoin transaction.
2. **Proves**: makes one [[SNARK Account Update Proof]].
3. **Submits**: sends a **SNARK Account Update Transaction** to the [[Strata Sequencer]] mempool.

That transaction also carries:
- the withdrawal requests produced in the batch, as outgoing messages to the bridge-gateway system account,
- markers for which inbox deposits were consumed.

Related: [[Alpen Transaction Lifecycle]], [[Strata Checkpoint]]
