---
tags: [bridge, dispute, glock]
aliases: [Strata challenge-response game, withdrawal state machine]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Glock Challenge-Response Game

A claim tx from an operator starts an onchain game against [[Watchtower]]s. Simplified state machine:

1. **Claim** (operator)
   - With no challenge for 1,064 blocks, the operator takes the **payout** ✅.
2. **Challenge** (watchtower)
3. **Bridge proof** (operator): the operator's proof that the claim is valid.
4. **Counterproof** (watchtower): disputes the bridge proof.
5. **Watchtower NACK** (operator): the operator's last chance to *disprove the counterproof*.
6. **Watchtower ACK** (watchtower), if the operator was wrong: **blocks the payout** and later opens the operator's **stake to slashing**, claimable by the watchtower.

If the operator wins, it can still pay out, but it has **~1.5 weeks after defending** to do so. Otherwise the protocol assumes the claim was false and the stake becomes slashable.

The garbled-circuit mechanics behind "counterproof" and "NACK" come from [[Glock]]. Evaluating the circuit on a bad proof leaks a secret, and that secret signs the fraud-proof tx ([[Adaptor Signatures]]). Full details are "future documentation".

Related: [[Bridge Timing Parameters]], [[Payout Administrator]], [[Operator Stake and Slashing]]

> [!warning] Code differs from the docs
> Current `strata-bridge`/`asm` code differs on several parameters and steps. These include operator assignment timing, denomination, timelocks, and which proof the garbled circuit checks. See [[Strata Bridge Proof Chain]].
