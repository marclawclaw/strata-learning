---
tags: [crypto, bitcoin, lineage]
aliases: [WOTS]
source: https://www.alpen.org/blog/introducing-the-strata-bridge
---
# Winternitz One-Time Signatures

Winternitz signatures are **hash-based one-time signatures** that can be verified in plain bitcoin script (hash opcodes only). [[BitVM2]] and the [[Strata Bridge BitVM2-era Design]] use them to **carry verifiable state between transactions**. The operator "signs" intermediate values, so a later tx (the disprove) can check what was asserted.

- Each key is used **exactly once**. Signing two different values reveals enough to forge, which doubles as **equivocation evidence**.
- Fresh keys are generated per deposit–withdrawal instance.
- They are large onchain, which is a big part of why BitVM2 assertions are expensive. [[Glock]] replaces this data-passing with garbled-circuit labels and [[Adaptor Signatures]].
