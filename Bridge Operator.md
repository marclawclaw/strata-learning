---
tags: [bridge, role]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Bridge Operator

One of **N** permissioned nodes that run the [[Strata Bridge]]. Each operator:

- stakes a **slashable bond** ([[Operator Stake and Slashing]]),
- sets up [[Garbled Circuits]] with every other operator ([[Bridge Setup Stage]]),
- watches bitcoin for [[Deposit Request Transaction]]s, co-signs the deposit's transaction graph, and then co-signs the Deposit Transaction,
- when assigned a withdrawal, **pays the user from its own BTC** and then claims reimbursement from the bridge ([[Operator Fronts Withdrawals]]),
- also acts as a **[[Watchtower]]** against the other operators' claims.

Operators' public keys are set by the Strata Administrator ([[Protocol Administration]]).

Testnet III partners include Stakely and Chainflow, with P2P.org expected (per press). See [[Alpen Project Status]].
