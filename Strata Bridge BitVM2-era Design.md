---
tags: [bridge, lineage, deprecated]
aliases: [Introducing the Strata bridge]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Strata Bridge BitVM2-era Design

⚠️ This is the original design (2024–25). Alpen's post now says *"Do not use this post as the current source of truth for the bridge"*. The current design is [[Glock]]-based. It is still useful for learning the primitives.

**Flow:** deposit → user burns on Strata → operator fronts → **kick-off** (references stake) → **claim** → optimistic payout after a lock period, *or* a **challenge** (the challenger pays a fee, which deters spam and covers the response) → multi-stage **assertions** → payout *or* **disprove** (burns part of the collateral and rewards the disprover).

**Differences from vanilla [[BitVM2]]:**
- **[[Anchor Blocks]] instead of superblocks.** Superblocks can be pre-mined by a disprover with hash power.
- **No start-time tx.** The operator commits to a start time in the claim, and untimely claims get an invalidation tx.
- **Multi-stage assertions** to fit bitcoin tx and standardness limits.
- **[[Stake Chain]]s** to reuse collateral.

**Proof:** the operator's claim was backed by a **Groth16** SNARK. It proved that the bitcoin headers include the anchor block and meet the difficulty, and that Strata state contains the burn.

**Roles:**
- "Anyone" could challenge or disprove (permissionless challengers).
- In the current design, watchtowers are the other operators ([[Watchtower]]).
