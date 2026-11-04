---
tags: [alpen, strata, design-rationale]
aliases: [Alpen design rationale]
source: https://docs.alpen.org/how-alpen-works/strata-design-rationale
---
# Execution and Orchestration Separation

Alpen copies Ethereum's post-Merge split between an **execution client** and a **consensus client**:

| Ethereum | Alpen system |
|---|---|
| Execution client (EVM) | **Alpen**: a Reth fork, EVM execution |
| Consensus client (PoS) | **Strata**: the orchestration layer (bitcoin interaction, bridge, checkpoints) |
| Engine API | A modified Engine API: Strata full node → Alpen full node messages, plus Alpen sequencer calls into its [[SNARK Account]] |

**Why split them:**
- Modularity: each layer can be iterated independently.
- Multiple execution environments: future WASM or privacy chains could become further [[SNARK Account]]s on Strata and interoperate trustlessly.

**Why Alpen built it custom instead of using Sovereign SDK, Rollkit or OP Stack:**
- Alpen started on a third-party SDK, then restarted from scratch after several months.
- Alpen wanted control over its roadmap, including preconfirmations, slashing, interop, and volition-style hybrid DA.
- Waiting on SDK changes caused delays.
- No existing SDK natively handles **bitcoin settlement**, meaning the UTXO model, limited script, and probabilistic finality, or an enshrined validating BTC bridge.

Related: [[Strata Orchestration Layer]], [[Four-Layer Architecture]]
