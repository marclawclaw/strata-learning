---
tags: [bridge, flow, deposit]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Bridge Deposit Flow

1. The user broadcasts a **[[Deposit Request Transaction]]** (DRT) on bitcoin: 2 BTC to a special P2TR address plus an `OP_RETURN` with the Alpen destination.
2. Operators detect it via the `OP_RETURN` magic bytes and **verify** the tapleaf hash, the multisig pubkey path, and the full Taproot commitment.
3. **Graph pre-signing**: each operator generates and signs the **glock transaction graph** for this deposit and shares it offchain. This graph is what later lets any functional operator fulfil a withdrawal and get reimbursed. **No operator signs the deposit tx until it holds the graph signed by all others.**
4. Operators co-sign (via [[MuSig2]]) and broadcast the **Deposit Transaction (DT)**. It spends the DRT via the N-of-N "deposit path" into the bridge UTXO set.
5. After **12 confirmations**, Strata reads it (via the [[Anchor State Machine]]) and queues it in Alpen's inbox ([[SNARK Account]]). The [[Alpen Sequencer]] then **mints 2 BTC** to the user's Alpen address.

If operators don't act within **36 blocks (~6 h)**, the user takes the DRT's **reclaim path** back.

The user pays bitcoin fees for both the DRT and the DT on top of the 2 BTC.

Related: [[Fixed Denomination and FIFO UTXOs]], [[Emulated Covenants via Pre-signing]]
