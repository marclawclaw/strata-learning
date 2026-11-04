---
tags: [governance, security]
aliases: [Alpen Administrator, Strata Administrator]
source: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---
# Protocol Administration

Alpen has **upgradeable parameters**, governed by **M-of-N multisigs** that post specially formatted messages **on bitcoin**. These are parsed by the [[Anchor State Machine]]. On testnet, Alpen Labs holds **all** keys.

| Role | Can update | Delay |
|---|---|---|
| **Alpen Administrator** | Alpen checkpoint VK, its own keys | 2,016 blocks (~2 wk), cancellable |
| **Strata Administrator** | Strata checkpoint VK, bridge and admin rules, operator keys, Security Council keys, Payout Admin keys, Safe Harbor address, its own keys | 2,016 blocks, cancellable |
| [[Strata Sequencer Manager]] | Sequencer pubkey | Immediate |
| [[Payout Administrator]] | Can block a specific payout | — |
| [[Security Council and Safe Harbor\|Security Council]] | Trigger an emergency bridge sweep | Immediate or 72 h |

- After the delay, updates are **auto-enacted** by full nodes, which must upgrade to stay in consensus.
- The 2-week delay gives users an **exit window** before rule changes.
- Early on, the Alpen and Strata admin signer sets will be **the same**.
- Updates are currently **unconstrained**, e.g. no bounds on parameter ranges.

**Trust implication:** the admin multisig can change the bridge rules and the operator set. Security is therefore really "1-of-N operators **and** the admin not acting maliciously, or users exiting in time".

Related: [[Verification Key Upgradeability]], [[Centralization Points]]
