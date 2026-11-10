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

## From code (alpenlabs/alpen bc989490, 2026-11-03)

- **Block time** is sequencer config `ol_block_time_ms`. The default is `DEFAULT_OL_BLOCK_TIME_MS = 5_000` (5 s) (`crates/config/src/config.rs:44`). `docker/configs/sequencer.toml` uses the same value.
- **Epoch length** is `[epoch_sealing] policy = "FixedSlot"`, `slots_per_epoch`. The default is `DEFAULT_SLOTS_PER_EPOCH = 64` (`config.rs:308`).
- The sequencer wraps it in `LimitAwareSealing` (`crates/ol/block-assembly/src/epoch_sealing.rs:183`). It seals **early** when the checkpoint's DA/log size hits a soft limit, or on an admitted checkpoint-predicate boundary.
- So with defaults, an epoch (one checkpoint) is ≈ 64 × 5 s ≈ **5.3 min**, or shorter if limits hit. The docs' "60-minute slots" and "~10 h checkpoints" are deployment choices, or outdated. Ask Alpen for the mainnet `ol_block_time_ms` / `slots_per_epoch`.
- Not covered by this: SP1 proving time per checkpoint, and bitcoin inclusion (≥ 1 block ≈ 10 min). These add to the effective cadence.
