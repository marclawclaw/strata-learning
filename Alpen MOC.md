---
tags: [alpen, strata, moc]
aliases: [Alpen Map, Strata MOC]
created: 2026-11-04
---
# Alpen MOC

Map of content for learning **Alpen** (EVM rollup), **Strata** (orchestration layer) and the **Strata bridge** (BTC ↔ Alpen).

## Big picture
- [[Alpen]]: what the product is
- [[Four-Layer Architecture]]: Bitcoin → Strata → Alpen → Apps
- [[Execution and Orchestration Separation]]: why there are two chains
- [[Bitcoin Roles in Alpen]]: DA, consensus, settlement
- [[Alpen is Closer to a Validium]]: a caveat on the "rollup" label

## Strata (orchestration layer)
- [[Strata Orchestration Layer]]
- [[SNARK Account]]
- [[Anchor State Machine]]
- [[Strata Sequencer]]
- [[Strata Checkpoint]]

## Alpen (execution layer)
- [[Alpen Sequencer]]
- [[Alpen Batch]]
- [[SNARK Account Update Proof]]
- [[Alpen EVM Compatibility]]
- [[Alpen Specifications]]
- [[Alpen Transaction Lifecycle]]
- [[Alpen Finality Levels]]

## Strata bridge
- [[Strata Bridge]]
- [[1-of-N Trust Model]]
- [[Bridge Operator]]
- [[Watchtower]]
- [[Bridge Setup Stage]]
- [[Bridge Deposit Flow]]
- [[Deposit Request Transaction]]
- [[Bridge Withdrawal Flow]]
- [[Operator Fronts Withdrawals]]
- [[Fixed Denomination and FIFO UTXOs]]
- [[Glock Challenge-Response Game]]
- [[Bridge Timing Parameters]]
- [[Operator Stake and Slashing]]

## Cryptography
- [[Glock]]
- [[Garbled Circuits]]
- [[DV-SNARK]]
- [[Mosaic]]
- [[Adaptor Signatures]]
- [[MuSig2]]
- [[Emulated Covenants via Pre-signing]]
- [[Connector Outputs]]

## History / lineage
- [[BitVM2]]
- [[Strata Bridge BitVM2-era Design]]
- [[Stake Chain]]
- [[Anchor Blocks]]
- [[Winternitz One-Time Signatures]]

## Governance and safeguards
- [[Protocol Administration]]
- [[Verification Key Upgradeability]]
- [[Payout Administrator]]
- [[Security Council and Safe Harbor]]
- [[Strata Sequencer Manager]]
- [[Centralization Points]]

## Using Alpen (testnet)
- [[Alpen Testnet Network Details]]
- [[Alpen CLI]]
- [[Deploying Contracts on Alpen]]
- [[Alpen RPC Methods]]
- [[Strata RPC Methods]]
- [[Strata Epochs and Slots]]
- [[Alpen Full Node]]
- [[Alpen Full Node Hardware]]
- [[Alpen Troubleshooting]]

## Integrations
- [[Strata and LEZ Integration]]: BTC into the Logos Execution Zone as a Strata SNARK account
- [[SP1 in Strata]]: how SP1 Groth16 proofs flow through the stack, and the constraints for LEZ
- [[Strata SNARK Account Lifecycle]]: account creation, updates, balances, deposits, upgrades (from code)
- [[Strata Bridge Proof Chain]]: ASM → Moho → bridge proof → counterproof → garbled circuit (from code)
- [[SP1 and RISC Zero Bridging Options]]: cost and trust of each way LEZ could meet Strata's SP1 requirement
- [[LEZ Burn Verifiability]]: what evidence of a LEZ burn exists today and what the roadmap plans

## Meta
- [[Alpen Project Status]]
- [[Alpen Audits]]
- [[Alpen Open Questions]]
- [[Alpen Doc Discrepancies]]
- [[Alpen Funding]]
- [[Alpen Sources]]
- Raw archive: `sources/docs.alpen.org/` (all doc pages) and [[Web Research Log 2026-11-04]]
