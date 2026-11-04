---
tags: [governance, proofs]
source: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---
# Verification Key Upgradeability

A SNARK verification key is **deterministically derived from the guest program**. Changing one byte of the program changes the VK.

So upgrading the protocol, whether to fix a bug or add a feature, means **swapping the VK** that checkpoints and [[SNARK Account]] updates are verified against. Assets and state stay in place, with no migration.

- This is dual-use: it is essential for bug fixes, and it is also an unlimited power to change the rules.
- It is controlled by the Alpen and Strata Administrators, with a 2-week cancellable delay ([[Protocol Administration]]).
- Testnet III reportedly **decoupled Alpen and the bridge** so they can be upgraded independently without regenesis.
