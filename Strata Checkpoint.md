---
tags: [strata, concept]
aliases: [checkpoint, Strata checkpoints]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# Strata Checkpoint

A **checkpoint** is posted to bitcoin by the [[Strata Sequencer]] about every **10 hours**. It contains:

- the **proven Strata state commitment**, which recursively commits to Alpen's proven state,
- a **SNARK proof** that the epoch's Strata blocks executed correctly,
- the **withdrawal intents** collected that epoch. The [[Bridge Operator]]s pick these up.

It is signed by the sequencer key and validated by the [[Anchor State Machine]].

Checkpoints drive [[Alpen Finality Levels]]:
- Seen on bitcoin → Strata "confirmed", Alpen "safe".
- Buried deep enough → finalized.

Withdrawals need the checkpoint proving the request to reach **12-block** maturity before an operator pays out ([[Bridge Withdrawal Flow]]).

An epoch is 64 slots per the RPC docs, which conflicts with ~10 h ([[Strata Epochs and Slots]], [[Alpen Doc Discrepancies]]). Inspect a checkpoint with `strata_getCheckpointInfo` ([[Strata RPC Methods]]).

A 10 h checkpoint interval plus maturity depth sets a floor on withdrawal latency.
