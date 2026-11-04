---
tags: [strata, concept]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# SNARK Account

A **SNARK account** is how an execution chain lives inside [[Strata Orchestration Layer|Strata]]. It holds:

- a **state commitment** of its program,
- a **verification key** for proofs that advance that state,
- an **inbox message queue**, e.g. deposits waiting to be consumed,
- its **BTC balance**.

To advance, the program submits a new state commitment plus a validity proof of the transition. Strata verifies it and updates the account. These updates are later rolled into a [[Strata Checkpoint]].

Through its SNARK account, a program gets access to the [[Strata Bridge]]:
- Deposits arrive as inbox messages.
- Withdrawals leave as outgoing messages to the bridge-gateway **system account**.

[[Alpen]] is currently the only SNARK account, with account ID `0101…01` (32 bytes of `0x01`). Inspect it via [[Strata RPC Methods]]. The design leaves room for other VMs, such as WASM or privacy chains, to interoperate via Strata.

The VK is mutable through governance. See [[Verification Key Upgradeability]].

Related: [[SNARK Account Update Proof]]
