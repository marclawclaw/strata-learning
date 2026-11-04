---
tags: [bitcoin, design-pattern, bridge]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Emulated Covenants via Pre-signing

Bitcoin has no covenants (no CTV or OP_CAT on mainnet), so a script can't say "these coins may only move to X". BitVM-style bridges **emulate** covenants:

1. Lock funds in an **N-of-N** output ([[MuSig2]]).
2. Before accepting the deposit, all N parties **pre-sign the complete graph** of allowed future transactions: claim, challenge, payout, slash, and so on.
3. Each honest party **deletes its key share** afterwards, or simply never signs anything else.

As long as **one** signer is honest, the pre-signed graph is the only way the funds can ever move. That is the basis of the [[1-of-N Trust Model]].

Building blocks used together with it:
- [[Connector Outputs]] enforce "only one of these paths".
- Timelocks.
- One-time signatures or garbled-circuit secrets that carry data between txs ([[Winternitz One-Time Signatures]], [[Glock]]).

Related: [[Bridge Deposit Flow]] (graph pre-signing step)
