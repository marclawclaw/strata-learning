---
tags: [bridge, flow, withdrawal]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Bridge Withdrawal Flow

There are two stages, and the second starts only after the first.

**Stage 1: user withdrawal**
1. The user sends the fixed denomination (**2 BTC**) to the bridge precompile `0x54…01` on Alpen ([[Alpen Specifications]]). The BTC is **burned**, and the request names a bitcoin destination.
2. The request flows through [[Alpen Batch]] → Strata → [[Strata Checkpoint]].
3. Once that checkpoint reaches **12-block maturity**, the protocol **assigns an operator**.
4. The operator **fronts** the BTC from its own funds, minus a fee (0.003 BTC on testnet), in a *withdrawal fulfillment* tx ([[Operator Fronts Withdrawals]]).
   - If the operator doesn't pay within **504 blocks (~3.5 days)**, the request is reassigned.

**Stage 2: operator claim**
5. The operator posts a **claim** tx on bitcoin.
6. If nobody challenges it within **1,064 blocks (~1.4 weeks)**, the operator spends the **payout** from the bridge UTXO.
7. If someone challenges, the [[Glock Challenge-Response Game]] runs.

Assignment rules:
- The user can choose an operator.
- Otherwise the request goes to a random member of the **signer set of the oldest bridge UTXO** (FIFO).
- If the chosen operator isn't in that set, the request is reassigned randomly within it. See [[Fixed Denomination and FIFO UTXOs]].

From the user's view: the BTC arrives after checkpoint plus maturity (hours), not after the 1.4-week challenge. The operator carries the challenge-period delay.
