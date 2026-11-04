---
tags: [bridge, deposit, bitcoin-tx]
aliases: [DRT]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Deposit Request Transaction

The user's bitcoin tx that starts a deposit ([[Bridge Deposit Flow]]).

**Outputs:**
1. **`OP_RETURN` (must be the first output)**: magic bytes identifying Strata (`ALPN` on Testnet III, per the [[Alpen CLI]] config), the user's **x-only pubkey**, and the **Alpen destination**.
2. **2 BTC** to a **P2TR** address:
   - **Key path is unspendable**: the internal key is a BIP-341 NUMS point.
   - **Script leaf "deposit path"**: an N-of-N operator [[MuSig2]] aggregate key.
   - **Script leaf "reclaim path"**: a user signature after a **36-block** timelock (~6 h).
3. An optional change output.

Why this works: the user never hands custody to the bridge until the operators have pre-signed the full graph. If they stall, the user's escape hatch is the timelocked reclaim path.

Testnet III bridge aggregate key (x-only, from the CLI config): `1e070c9190e2dcc8671957722aec6f1533f47ce3bf2fcb53cda2961faf309049`.

Related: [[Fixed Denomination and FIFO UTXOs]]
