---
tags: [bridge, lineage, bitcoin]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Anchor Blocks

The [[Strata Bridge BitVM2-era Design]] used anchor blocks to prove that an operator's claim is on the **canonical bitcoin chain**, not a private fork.

- **Setup:** signers agree on M future checkpoint heights. Each signer i has a key Y_{i,j} per checkpoint j. Group keys are aggregated MuSig2-style: Y_j = Σ c_{i,j} Y_{i,j}.
- The operator's header chain in its proof **must include the anchor block** of the checkpoint it commits to. This stops forks that start too far back and exploit the difficulty adjustment.
- **Chain-disproving tx:** if an operator commits to a stale checkpoint, a disprover reveals a **newer group signing key** and slashes the operator.
- **Key revelation options:**
  - *Lookback*: embed intermediate group keys plus the latest anchor hash in the deposit.
  - *Chained keys*: each signer uses a hash chain, and the deposit embeds N individual keys.

Anchor blocks replaced [[BitVM2]]'s **superblocks**, which a disprover with hash power could pre-mine.
