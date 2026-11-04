---
tags: [bridge, role]
aliases: [challenger, verifier]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Watchtower

In the [[Glock]]-based [[Strata Bridge]], a **watchtower** is the party that challenges an operator's claim. In practice, the **other operators** act as watchtowers.

- It challenges a false claim within the challenge window ([[Bridge Timing Parameters]]).
- It posts a **counterproof** after the operator's bridge proof.
- If it wins, it posts a **watchtower ACK** that blocks the payout. The faulty operator's stake then becomes slashable, and the watchtower claims it as a reward ([[Operator Stake and Slashing]]).

In the older [[Strata Bridge BitVM2-era Design]], *anyone* could challenge or disprove. In the glock design, watchtowers are tied to the garbled-circuit setup between operators ([[Bridge Setup Stage]]).

Known limitation: a counterproof reveals information, so a watchtower can only block a given operator **once**. See [[Payout Administrator]].

Related: [[Glock Challenge-Response Game]]
