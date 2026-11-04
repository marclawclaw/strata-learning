---
tags: [crypto, snark]
aliases: [DV-Pari, designated-verifier SNARK]
source: https://www.alpen.org/blog/glock-is-here
---
# DV-SNARK

A **designated-verifier SNARK**: only a party holding a secret verification key can check proofs. Public verifiability is dropped in exchange for **smaller proofs and a cheaper verifier circuit**.

In [[Glock]] (Glock25), the SNARK is **DV-Pari**, a designated-verifier variant of the **Pari** SNARK.

Why it fits garbled circuits: the [[Garbled Circuits|garbled]] verifier circuit is built for a specific evaluator (the watchtower) anyway. A smaller verifier circuit means fewer gates, which means less offchain garbling and storage. Smaller proofs mean fewer input labels to commit onchain.

Alpen claims it has lower complexity than Groth16 and "orders of magnitude" lower offchain costs. Alpen released a Rust implementation and a circuit generator.

Contrast: the 2026 [[Mosaic]] paper instantiates the bridge with a **Groth16** verifier circuit, and audits list "g16 and CKT" (Groth16 + circuit). So the deployed path may be Groth16-based. See [[Alpen Open Questions]].
