---
tags: [strata, parameters]
source: https://docs.alpen.org/build-alpen-apps/rpc-endpoints
---
# Strata Epochs and Slots

- A **slot** is a Strata block: one block per slot, with a **60-minute** block time.
- An **epoch** is the range of slots sealed into one [[Strata Checkpoint]]. Its last block is "terminal" (`is_terminal`).
- The RPC docs say each epoch on Testnet III contains **64 slots**.

⚠️ 64 slots × 60 min ≈ **64 h** per epoch, which contradicts the "checkpoint roughly every 10 hours" in the architecture docs. See [[Alpen Doc Discrepancies]].

Related: [[Strata RPC Methods]], [[Strata Sequencer]]
