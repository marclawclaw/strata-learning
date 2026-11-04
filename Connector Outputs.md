---
tags: [bitcoin, design-pattern]
aliases: [connector UTXO]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Connector Outputs

A **connector** is a UTXO, often dust or zero-value, created by one tx and required as an input by several mutually exclusive txs. Bitcoin forbids double-spends, so **only one** of those paths can ever confirm. Connectors are used as mutexes and signals inside pre-signed graphs ([[Emulated Covenants via Pre-signing]]).

Uses in the [[Strata Bridge]]:
- An operator's payout must spend a connector, so a watchtower ACK or a burn tx can **block the payout** by spending that connector first.
- The [[Payout Administrator]] blocks repeat false claims by spending the connector that the payout tx needs.
- In the [[Stake Chain]], connectors link stake to claims.
