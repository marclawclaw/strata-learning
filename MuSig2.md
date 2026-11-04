---
tags: [crypto, bitcoin, schnorr]
source: https://github.com/bitcoin/bips/blob/master/bip-0327.mediawiki
---
# MuSig2

**MuSig2 (BIP-327)** is a two-round Schnorr multisignature scheme. N signers aggregate their keys into **one public key** and produce **one signature**. Onchain it looks like a single-sig Taproot spend.

**In the [[Strata Bridge]]:**
- All deposit-flow multisigs are N-of-N MuSig2: the [[Deposit Request Transaction]] deposit path, bridge UTXOs, and pre-signed graph txs.
- **N-of-N** is what makes [[1-of-N Trust Model|1-of-N]] safety work. A single honest signer can veto.
- The cost: every operator must take part in signing for every new deposit, so liveness of *deposits* needs all N.

Related: [[Emulated Covenants via Pre-signing]]
