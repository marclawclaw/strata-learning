---
tags: [bridge, security, trust-model]
aliases: [1-of-N, one honest operator]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# 1-of-N Trust Model

The [[Strata Bridge]] is secure if **at least one of the N operators is functional (honest and online)**.

Where the "1" comes from:
- **Safety of deposits**: funds sit in an N-of-N [[MuSig2]] output. Any spend needs *all* N signatures, so one honest operator refusing to sign a noncompliant transaction blocks theft. The only spends that exist are the ones in the pre-signed graph ([[Emulated Covenants via Pre-signing]]).
- **Liveness of withdrawals**: any single operator can front a withdrawal and claim reimbursement **without cooperation** from the others ([[Operator Fronts Withdrawals]]).
- **Policing**: any single honest [[Watchtower]] can challenge a false claim ([[Glock Challenge-Response Game]]).

Compare: typical multisig bridges are **M-of-N honest-majority**, so liveness is lost if more than N−M operators go offline.

Caveats:
- Setup requires all N operators to sign the graph. A single operator can **stall new deposits** but cannot steal.
- There is a known gap: a watchtower can block a given faulty operator **only once**, patched by the [[Payout Administrator]].
- Governance ([[Protocol Administration]]) can change the rules, so the effective trust also includes admin multisigs.
