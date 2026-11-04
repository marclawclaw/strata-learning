---
tags: [alpen, bitcoin, architecture]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Bitcoin Roles in Alpen

Bitcoin is the source of truth and plays three roles:

1. **Data availability**: the [[Alpen Sequencer]] posts each [[Alpen Batch]]'s aggregated **state diff** to bitcoin, and the [[Strata Sequencer]] posts [[Strata Checkpoint]]s, which carry proven state commitments and withdrawal intents. Anyone can rebuild both chains' state from bitcoin.
2. **Consensus**: full nodes derive the canonical chain from batch and checkpoint data confirmed on bitcoin. Nodes running the same software on the same network always reach the same state. The bridge relies on this when operators and watchtowers make proofs about "the current state".
3. **Settlement**: once a checkpoint reaches maturity depth, the state it covers can't be reorged without reorging bitcoin. The bridge's [[Glock Challenge-Response Game]] refers to this confirmed state.

There is a nuance on DA: the docs say *state diffs* are posted, but older material says full blocks are relayed offchain. See [[Alpen is Closer to a Validium]].
