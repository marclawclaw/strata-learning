---
tags: [bitcoin, bridge, lineage]
aliases: [BitVM]
source: https://bitvm.org/bitvm_bridge.pdf
---
# BitVM2

BitVM2 (Robin Linus et al.) is an **optimistic** scheme for verifying arbitrary computation, in practice a SNARK verifier, on bitcoin **without a soft fork**. It enables bridges with a **1-of-N** honesty assumption.

**Mechanism:**
- The SNARK verifier is split into **chunks**, each small enough to be a bitcoin script.
- The operator **asserts** intermediate values, committed with [[Winternitz One-Time Signatures]].
- A challenger who finds one wrong chunk executes **only that chunk's script** in a Taproot leaf to **disprove**, which slashes the operator.
- Funds move via [[Emulated Covenants via Pre-signing]] and [[Connector Outputs]].

**Pain points:**
- Assertion and disprove txs are **large** (MBs of onchain data in the worst case).
- Design complexity.
- High operator collateral, "on the order of 5 BTC" per Alpen.

**Relation to Alpen:**
- The first [[Strata Bridge]] was BitVM2-inspired with optimizations ([[Strata Bridge BitVM2-era Design]]).
- The current bridge replaces onchain chunked verification with **offchain garbled-circuit evaluation** ([[Glock]]), which Alpen claims reduces dispute data by ~430–550×.
