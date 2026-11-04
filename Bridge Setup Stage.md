---
tags: [bridge, flow]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Bridge Setup Stage

The first of three bridge stages (setup → [[Bridge Deposit Flow|deposit]] → [[Bridge Withdrawal Flow|withdrawal]]):

- Each [[Bridge Operator]] **stakes its bond** ([[Operator Stake and Slashing]]).
- Each operator **establishes [[Garbled Circuits]] with each other operator**. This is pairwise, prover ↔ verifier (watchtower), so the circuits exist before any dispute happens.

Malicious security for this exchange (cut-and-choose without blowing up onchain cost) is the subject of [[Mosaic]].

Per deposit there is a further offchain step: generating and signing the **glock transaction graph** ([[Bridge Deposit Flow]]).

Details of the setup are not yet public ("future documentation"). See [[Alpen Open Questions]].
