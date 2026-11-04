---
tags: [governance, bridge, limitation]
source: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---
# Payout Administrator

**This role exists because of a known bridge limitation.** In current node software, a [[Watchtower]]'s counterproof can block a given faulty operator's payout **only once**. The counterproof reveals information, and the operator can then use it to **NACK future counterproofs** even when its claims are false.

That breaks the [[1-of-N Trust Model]]: one watchtower is no longer enough against a repeat offender.

**Mitigation:** the Payout Administrator is an **"honest-minority" M-of-N multisig (M < N/2)**. It can spend the [[Connector Outputs|connector UTXO]] that an operator's payout tx needs, which makes that payout impossible.

Why an honest minority: to stop the multisig from reaching quorum, a malicious operator must collude with **N−M+1** signers. For a 2-of-6 multisig that is 5 signers, compared with 3 for a 4-of-6.

Alpen says it has ideas for a cryptographic fix. Until then this role is a **human in the loop**.

Related: [[Glock Challenge-Response Game]], [[Protocol Administration]]
