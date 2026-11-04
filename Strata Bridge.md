---
tags: [bridge, strata, concept]
aliases: [Alpen bridge, Strata BTC bridge]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Strata Bridge

The **enshrined BTC bridge** of the [[Strata Orchestration Layer]]. It moves BTC between bitcoin and [[Alpen]], with 1 BTC on Alpen backed by 1 BTC locked on bitcoin.

**Core idea:** a set of N [[Bridge Operator]]s co-sign (via [[MuSig2]]) the transactions that move BTC in and out. Unlike classic federations, which need an honest majority, it needs **only one functional operator** for both deposit safety and withdrawal liveness ([[1-of-N Trust Model]]).

How it gets there:
- **Pre-signed transaction graphs** emulate covenants ([[Emulated Covenants via Pre-signing]]).
- **Operators front withdrawals** and claim reimbursement later ([[Operator Fronts Withdrawals]]).
- Claims are policed by a **[[Glock]]-based [[Glock Challenge-Response Game]]**. Earlier versions were [[BitVM2]]-based ([[Strata Bridge BitVM2-era Design]]).
- A slashable **bond** rewards correct challengers ([[Operator Stake and Slashing]]).

Three stages: [[Bridge Setup Stage]] → [[Bridge Deposit Flow]] → [[Bridge Withdrawal Flow]].

Parameters: [[Fixed Denomination and FIFO UTXOs]], [[Bridge Timing Parameters]].

Caveats: [[Payout Administrator]] (a known repeat-claim limitation), [[Security Council and Safe Harbor]], [[Centralization Points]].
