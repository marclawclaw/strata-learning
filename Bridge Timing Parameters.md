---
tags: [bridge, parameters, reference]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Bridge Timing Parameters

Testnet values from the docs:

| Parameter | Value | ≈ Time |
|---|---|---|
| Deposit finality depth (before mint) | 12 blocks | ~2 h |
| DRT reclaim timelock | 36 blocks | ~6 h |
| Checkpoint maturity before withdrawal payout | 12 blocks | ~2 h |
| Operator fulfillment timeout (then reassigned) | 504 blocks | ~3.5 days |
| Claim challenge period | 1,064 blocks | ~1.4 weeks |
| Post-defence payout window | ≈ 1.5 weeks | — |
| Admin update enactment delay | 2,016 blocks | ~2 weeks |
| Security Council delayed sweep | 72 h | — |
| Strata checkpoint cadence | — | ~10 h |
| Alpen batch cadence | 1,440 L2 blocks | ~2 h |

Related: [[Bridge Withdrawal Flow]], [[Bridge Deposit Flow]], [[Protocol Administration]], [[Security Council and Safe Harbor]]
