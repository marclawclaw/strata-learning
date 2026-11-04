---
tags: [alpen, flow]
source: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---
# Alpen Transaction Lifecycle

1. **Submit**: an EOA tx via any EVM wallet, or an ERC-4337 `UserOperation` via a bundler. It goes to an Alpen full node and propagates over P2P to the [[Alpen Sequencer]].
2. **Alpen block** (~5 s): the tx is included and **pre-confirmed** by the sequencer.
3. **[[Alpen Batch]]** (~2 h): state diff to bitcoin, [[SNARK Account Update Proof]], update tx to Strata.
4. **Strata block** (~60 min): the [[Strata Sequencer]] verifies the proof and applies it.
5. **[[Strata Checkpoint]]** (~10 h): proof plus state commitment posted to bitcoin.
6. **Confirmation**: Strata full nodes read the checkpoint and mark it "confirmed". Alpen nodes mark covered blocks "safe".
7. **Finalization**: the checkpoint's bitcoin block is buried deep enough, so it is "finalized".

End to end, finality takes on the order of **hours** (up to ~12 h plus burial depth). See [[Alpen Finality Levels]].

Deposits start on bitcoin instead. See [[Bridge Deposit Flow]].
