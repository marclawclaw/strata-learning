---
tags: [bridge, parameters]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Fixed Denomination and FIFO UTXOs

- Deposits and withdrawals use a **single fixed denomination**: **2 BTC** on testnet. Mainnet values are TBA.
- The withdrawal fee is **0.003 BTC** on testnet.
- Bridge UTXOs are **indexed by arrival order**, and withdrawals consume the **lowest-index (oldest)** UTXO first: **FIFO**.

**Why fixed denominations:** each deposit UTXO has its own pre-signed graph with one payout. Variable amounts would need splitting and merging inside a covenant-less graph. A fixed size makes every claim "1 deposit ↔ 1 withdrawal". It is also common in BitVM-family designs.

**Why FIFO:** the operator signer set can change over time, so different UTXOs may have different signer sets. Assignment uses the oldest UTXO's signer set. FIFO drains old UTXOs (and old signer sets) first.

The [[Alpen CLI]] lets you withdraw **positive multiples** of 2 BTC (`bridge_denomination_sats = 200000000`). Its config also has `bridge_fee_sats = 1000`, which conflicts with the 0.003 BTC fee. See [[Alpen Doc Discrepancies]].

Side effect: small holders can't withdraw less than 2 BTC on L1 directly. They need a market maker or a liquidity layer.

Related: [[Bridge Withdrawal Flow]], [[Deposit Request Transaction]]
