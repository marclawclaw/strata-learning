---
tags: [alpen, strata, proofs]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# SNARK Account Update Proof

A single SNARK produced per [[Alpen Batch]] that attests to **two facts**:

1. The batch executed correctly (EVM state transition validity).
2. The batch's DA transaction is **included in a valid bitcoin block** on the canonical chain.

Binding the proof to bitcoin inclusion is what makes DA enforceable. A proof can't be accepted unless its data actually landed on bitcoin.

The [[Strata Sequencer]] verifies it against the [[SNARK Account]]'s VK. Strata then proves its own execution in the [[Strata Checkpoint]], so proofs are **layered**: Alpen proof → Strata proof → bitcoin.
