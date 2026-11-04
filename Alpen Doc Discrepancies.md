---
tags: [alpen, critique, questions]
---
# Alpen Doc Discrepancies

These are places where the sources disagree. Resolve them from code or by asking the team.

1. **Checkpoint cadence**: the architecture and lifecycle docs say a checkpoint every **~10 h**. The RPC docs say **64 slots/epoch × 60 min** ≈ **64 h**. ([[Strata Epochs and Slots]])
2. **Withdrawal fee**: the bridge docs say **0.003 BTC** (300,000 sats). The CLI config has `bridge_fee_sats = 1000`. These may be different fees, e.g. the operator fulfillment fee vs a deposit fee. ([[Fixed Denomination and FIFO UTXOs]], [[Alpen CLI]])
3. **Withdrawal amount**: the bridge docs say withdrawals must be *the* fixed denomination. The CLI accepts **multiples** of 2 BTC, which presumably become multiple 2-BTC withdrawal requests.
4. **Deposit latency**: 12 confirmations (~2 h) before minting per the bridge docs, but the CLI and deploy guides say "up to **15 hours**". The gap is probably the batch and checkpoint cadence. The CLI also has `finality_depth = 0`.
5. **How full nodes learn Strata state**: the architecture docs say full nodes read checkpoints from bitcoin through the ASM. The operator guide configures `[ol] source = "rpc"` pointing at Alpen's Strata RPC. ([[Alpen Full Node]])
6. **Verifier SNARK**: the Glock25 paper uses **DV-Pari**. Mosaic and the audits use **Groth16**. Zelcore says "Glock plus a Groth16 verifier". ([[DV-SNARK]])
7. **Testnet III dates**: press says it went live **2026-07-28**, but the get-started page says the previous testnet (chain 8150) was decommissioned on **2026-08-18** after a regenesis. These fit if both ran in parallel for three weeks.
8. **Domains**: alpenlabs.io and docs.alpenlabs.io now redirect to **alpen.org** and **docs.alpen.org**.

Related: [[Alpen Open Questions]]
