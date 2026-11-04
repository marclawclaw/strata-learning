---
tags: [crypto, primitive]
source: https://eprint.iacr.org/2026/812
---
# Garbled Circuits

Yao's garbled circuits turn a boolean circuit into an encrypted form. Each wire has two random **labels**, one for 0 and one for 1. Someone holding one label per input wire can evaluate the circuit and learn **only the output labels**.

**How they are used on bitcoin ([[Glock]]):**
- The prover (operator) garbles a **verifier circuit**, e.g. a SNARK verifier, that **outputs a secret if the witness/proof is invalid**.
- The prover commits onchain to its witness, which implicitly reveals the matching **input labels**.
- The evaluator (watchtower) runs the circuit offchain. On an invalid proof it gets the secret and posts it as the **fraud proof**.

Bitcoin script never runs the verification, so the onchain cost stays tiny compared with [[BitVM2]]'s chunked onchain verifier.

**Problems:**
- **Malicious garbler**: the prover could garble a wrong circuit. The fix is cut-and-choose, which opens most copies and checks them.
- **Onchain label cost**: naive cut-and-choose multiplies the posted labels by the number of copies. [[Mosaic]] solves this.

Related: [[Adaptor Signatures]], [[DV-SNARK]]
