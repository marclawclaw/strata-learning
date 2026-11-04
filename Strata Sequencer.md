---
tags: [strata, component, centralization]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# Strata Sequencer

The **centralized** block producer of [[Strata Orchestration Layer|Strata]]. Alpen Labs runs it on testnet.

- It receives **SNARK Account Update Transactions**, currently only from the [[Alpen Sequencer]].
- It verifies each [[SNARK Account Update Proof]] and applies the update, advancing the account's state commitment and locked value.
- It places deposits into the destination account's inbox during block execution.
- It produces a Strata block about every **60 min**.
- About every **10 h** it seals blocks into an epoch and posts a [[Strata Checkpoint]] with a SNARK proof of correct Strata execution.

Its public key is hardcoded and checked during checkpoint verification. Rotating it is the job of the [[Strata Sequencer Manager]].

Related: [[Centralization Points]]
