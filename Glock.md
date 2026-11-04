---
tags: [crypto, glock, bridge]
aliases: [Garbled Locks, Glocks, Glock25, garbled lock]
source: https://eprint.iacr.org/2025/1485
---
# Glock

**Glock = "Garbled Locks for Bitcoin"**, by Liam Eagen (then Alpen Labs), ePrint **2025/1485**, released 19 Aug 2025. It is a framework for **garbled-circuit-based optimistic smart contracts on bitcoin**, and the verification engine behind the current [[Strata Bridge]].

**Core principle: authenticated conditional disclosure of secrets.**
- One party (the prover/operator) **authenticates the computation's input** onchain.
- If the computation **fails**, e.g. the SNARK proof is invalid, another party (the [[Watchtower]]) can **derive a secret** by evaluating a [[Garbled Circuits|garbled circuit]].
- That secret enables a signature, the **fraud proof**, which unlocks the punishment path ([[Adaptor Signatures]]).
- The heavy computation (verification) runs **offchain**. Only short commitments and signatures go onchain.

**Construction (Glock25):**
- It reduces "verify arbitrary bounded computation" to "**verify a SNARK**".
- It uses a **designated-verifier SNARK**, **DV-Pari**, with small proofs ([[DV-SNARK]]).
- Malicious security comes from **cut-and-choose + verifiable secret sharing + adaptor signatures**.
- It claims to be the first concretely practical construction *without* Rubin's "Grug" technique.
- It is concurrent with **Delbrag** (Rubin) and **BitVM3**. Like Delbrag, the circuit leaks a secret that is used to produce the fraud-proof signature. Alpen says BitVM3's RSA-based construction had flaws, reported by Liam Eagen and **Fairgate Labs**.

**Claimed efficiency** (vendor claims, measured as worst-case onchain data in a dispute):
- SAP variant: >**550×** less than [[BitVM2]].
- QAP variant (the one Alpen pursues): >**430×**, plus better offchain storage.
- Marketing says "up to 1000×" lower onchain cost and "1000× cheaper challenges".
- Glock is also claimed to allow a much smaller operator stake than BitVM2's ~5 BTC ([[Operator Stake and Slashing]]).

Open-source releases: DV-Pari in Rust, a binary circuit generator for DV-Pari, and a generic garbling/evaluation tool.

Note: [[Mosaic]] (2026) instantiates the bridge with a **Groth16** verifier circuit, so the deployed verifier may differ from DV-Pari. The Starknet Foundation gave Alpen a grant for the development **and audit** of a **shared Glock verifier** as a public good. Zelcore's description ("Glock plus a Groth16 verifier") conflicts with Alpen's wording ([[Alpen Doc Discrepancies]]).

Liam Eagen's affiliation moved from Alpen Labs (Glock paper) to Ideal Group ([[Mosaic]] paper).

Related: [[Glock Challenge-Response Game]]
