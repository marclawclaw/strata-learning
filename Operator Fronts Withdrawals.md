---
tags: [bridge, design-pattern]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Operator Fronts Withdrawals

In the [[Strata Bridge]] (and [[BitVM2]]-style bridges in general), the withdrawing user is paid **from the operator's own BTC**, not directly from the bridge UTXO. The operator later **claims reimbursement** from the bridge.

**Why:** deposit outputs and their pre-signed graph are fixed at deposit time. Bitcoin has no covenants, so the graph can't know who will eventually withdraw. It can only say "pay *an operator* who proves they paid *someone* entitled to it". So the operator pays the user first, then proves it in the claim.

**Consequences:**
- Users get fast exits, and the operator carries the challenge period (~1.4 weeks) of **capital lockup**.
- Operators need **BTC liquidity** in addition to their bond.
- Liveness needs only one operator with funds ([[1-of-N Trust Model]]). A non-paying operator is replaced after 504 blocks.
- Operators charge a fulfillment fee (0.003 BTC on testnet).

Related: [[Bridge Withdrawal Flow]], [[Emulated Covenants via Pre-signing]]
