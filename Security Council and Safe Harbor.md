---
tags: [governance, bridge, emergency]
source: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---
# Security Council and Safe Harbor

**The problem:** a fix shipped via a Strata Administrator update waits **2,016 blocks**. If there is a critical bug that drains the bridge, an attacker has two weeks to exploit it.

**Security Council:** an M-of-N multisig that can confirm a tx on bitcoin signalling [[Bridge Operator]]s to use the **N-of-N cooperative path** to **sweep all bridge UTXOs** into the Safe Harbor, either **immediately** or after a **72 h** delay.

**Safe Harbor address:** an M-of-N multisig **hardcoded** in the protocol. Only the Strata Administrator can change it, so the Council can't redirect a sweep to an attacker. Funds stay there until the fix lands, then return to the bridge.

Note: the sweep uses the cooperative N-of-N path, so it needs **all operators online and cooperating**.

Related: [[Protocol Administration]], [[Centralization Points]]
