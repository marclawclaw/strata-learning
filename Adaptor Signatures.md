---
tags: [crypto, primitive, schnorr]
source: https://eprint.iacr.org/2026/812
---
# Adaptor Signatures

An **adaptor signature** is a Schnorr pre-signature that becomes valid only when combined with a secret `t`. Publishing the completed signature **reveals `t`** to anyone who holds the pre-signature.

This turns "signing a bitcoin tx" into "revealing a secret", which links offchain cryptography to onchain actions.

**In [[Glock]] / [[Mosaic]]:**
- The operator's onchain commitment to its witness/proof reveals values, such as the missing polynomial share for labels, that the watchtower needs to evaluate the [[Garbled Circuits|garbled circuit]].
- A secret leaked by the circuit on an invalid proof lets the watchtower complete a signature on the punishment/ACK path.

Related: [[MuSig2]], [[Glock Challenge-Response Game]]
