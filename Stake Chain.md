---
tags: [bridge, lineage, economics]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Stake Chain

From the [[Strata Bridge BitVM2-era Design]]: instead of locking collateral **per deposit**, an operator locks it once in a **pre-signed chain of "link" txs**.

- Each link lets the stake back **one claim at a time**, so the same stake can't back concurrent claims.
- A successful disproof **slashes the stake and breaks the chain**.
- **Burn txs** slash the stake if the operator advances the chain while also making a false claim, and they block payouts via [[Connector Outputs]]. Burn txs come **in sets** to resist censorship.
- An operator can run **multiple stake chains** to process claims in parallel.

Purpose: reduce capital requirements, which is the main economic pain of BitVM bridges.

Related: [[Operator Stake and Slashing]]
