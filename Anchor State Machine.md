---
tags: [strata, component]
aliases: [ASM]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Anchor State Machine

The **ASM** is the Strata component that implements Strata's **bitcoin onchain protocol** and keeps bitcoin-derived state. It:

1. parses and validates [[Strata Checkpoint]]s on bitcoin to decide the canonical Strata chain,
2. tracks deposit and withdrawal requests ([[Deposit Request Transaction]], [[Bridge Withdrawal Flow]]),
3. handles protocol updates, i.e. the admin messages from [[Protocol Administration]].

It is shared by the [[Strata Sequencer]], Strata full nodes, and bridge clients ([[Bridge Operator]]). They all read bitcoin the same way, which is what lets the bridge games refer to an agreed state.
