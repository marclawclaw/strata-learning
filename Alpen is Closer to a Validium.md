---
tags: [alpen, critique, rollup-taxonomy]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Alpen is Closer to a Validium

Alpen calls itself a "validity rollup" (originally a "sovereign validity rollup"). Its own bridge post admits that sequencers post validity proofs, withdrawal data, and block commitments to bitcoin while **relaying full blocks offchain**, so it "could more accurately be categorized as a validium".

Current docs say each [[Alpen Batch]]'s **aggregated state diff** is posted to bitcoin for DA. That is enough to reconstruct *state*, but not the full transaction history. State-diff DA is a legitimate rollup DA style (zkSync and Starknet use it). Whether Alpen counts as a rollup depends on whether the diffs are complete and enforced by the proof.

Takeaway: be precise about which one you mean. Proofs are on bitcoin. Full block data is not.

Related: [[Bitcoin Roles in Alpen]], [[Alpen Open Questions]]
