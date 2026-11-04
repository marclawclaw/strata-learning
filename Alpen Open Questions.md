---
tags: [alpen, questions, todo]
---
# Alpen Open Questions

Things the public docs leave unclear. These are worth answering from code at `github.com/alpenlabs` (strata, alpen, strata-bridge, glock repos) or from the papers.

- [ ] **What is N?** How many bridge operators will mainnet have, and how is the set chosen or rotated?
- [ ] **Bond size** per operator under glock ([[Operator Stake and Slashing]]).
- [ ] **Deployed SNARK verifier**: is it DV-Pari ([[DV-SNARK]]) or Groth16 ([[Mosaic]]; the "g16" audit)? Which proving system is used for the Alpen and Strata proofs (SP1? RISC Zero?)?
- [ ] **Exact transaction graph** of the glock bridge, including what the bridge proof proves: the header chain, the checkpoint, and inclusion of the withdrawal fulfillment ([[Glock Challenge-Response Game]]).
- [ ] **Setup cost**: the size of the garbled circuits exchanged pairwise among N operators, and how it scales with N² ([[Bridge Setup Stage]]).
- [ ] **One-shot watchtower fix**: the planned replacement for the [[Payout Administrator]].
- [ ] **Do DA state diffs make Alpen a real rollup?** Can a user force inclusion or exit if the sequencer censors? ([[Alpen is Closer to a Validium]], [[Centralization Points]])
- [ ] **Mainnet denomination and fees** ([[Fixed Denomination and FIFO UTXOs]]).
- [ ] Resolve everything in [[Alpen Doc Discrepancies]].
- [ ] Read the full Glock and Mosaic PDFs. Only the abstracts were read.
- [ ] **Liquidity**: how do sub-2-BTC users exit to L1?
