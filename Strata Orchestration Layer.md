---
tags: [strata, concept]
aliases: [Strata, Orchestration Layer, OL]
source: https://docs.alpen.org/how-alpen-works/system-architecture
---
# Strata Orchestration Layer

**Strata** (also called the **Orchestration Layer**, OL) is a lightweight blockchain between bitcoin and execution chains like [[Alpen]]. It is Alpen's analogue of Ethereum's consensus layer ([[Execution and Orchestration Separation]]).

- State = **ledger accounts** plus their **inbox message queues**.
- Each account is either a **system account**, which is precompile-like with protocol-defined behaviour (e.g. the bridge gateway), or a [[SNARK Account]].
- Strata includes an enshrined BTC bridge ([[Strata Bridge]]).
- Its bitcoin-facing logic lives in the [[Anchor State Machine]].
- Blocks come every ~60 min, made by the [[Strata Sequencer]]. Checkpoints go to bitcoin every ~10 h ([[Strata Checkpoint]]).
- Strata supports multiple SNARK accounts, but **Alpen is the only one deployed today**.

A naming quirk: "Strata" used to be the name of the whole rollup. It now names only the orchestration layer, and the EVM chain is "Alpen". Older docs and blogs use "Strata" for the rollup.
