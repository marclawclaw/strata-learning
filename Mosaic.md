---
tags: [crypto, glock, paper]
source: https://eprint.iacr.org/2026/812
---
# Mosaic

**"Mosaic: Practical Malicious Security for Garbled Circuits on Bitcoin"**, ePrint **2026/812**. Received Apr 2026, revised Jul 2026. Authors: N. Khambhati et al. (Alpen Labs) and Liam Eagen (Ideal Group). The Alpen bridge docs link to it as the bridge's glock protocol.

**Problem:** malicious-garbler security through cut-and-choose garbles **N copies** of the circuit. The evaluator needs the prover's input labels for the unopened copies, and posting labels onchain costs permanent block space × N.

**Idea:**
- Make labels across all N copies **evaluations of a degree-t polynomial**.
- Cut-and-choose opens **t** shares, which is **one short** of reconstruction.
- The prover's **onchain witness commitment** (via [[Adaptor Signatures]]) reveals the **missing share** as a byproduct.
- The evaluator then **interpolates** labels for every unopened copy.

The **onchain footprint is independent of N.**

**Instantiation:** a trust-minimized bitcoin bridge with a **Groth16 verifier circuit**, a full spec, and a Rust implementation. This is effectively the [[Strata Bridge]].

Related: [[Glock]], [[Garbled Circuits]], [[Bridge Setup Stage]]
