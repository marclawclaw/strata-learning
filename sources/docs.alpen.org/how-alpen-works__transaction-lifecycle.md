---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/how-alpen-works/transaction-lifecycle
---

# Transaction lifecycle

This document follows the journey of an Alpen transaction from the moment a user submits it, through the Alpen and Strata chains, to its finalization on bitcoin.

At a high level, a transaction passes through these stages:

1. Submission to an Alpen full node and propagation to the Alpen sequencer.
2. Inclusion in an Alpen block by the Alpen sequencer and submitted to Strata
3. Inclusion in a Strata block by the Strata sequencer and stat update posted to Bitcoin in a Strata checkpoint.
4. Confirmation of the checkpoint by Strata full nodes and, in turn, Alpen full nodes.
5. Finalization, once the Bitcoin block carrying the checkpoint is sufficiently buried.

## Transaction initiation and propagation <a href="#transaction-initiation-and-propagation" id="transaction-initiation-and-propagation"></a>

Alpen supports both Externally Owned Account (EOA) transactions and account abstraction transactions (referred to as `UserOperations`) as defined in ERC-4337. EOA transactions can be initiated by users through any EVM-compatible wallet that is connected to an Alpen full node. For account abstraction, we provide a bundler that accepts `UserOperation`s and relays them to a full node we have deployed. The transactions propagate across a P2P network. A full node forwards the transactions it receives toward the Alpen sequencer, so every user transaction eventually reaches the sequencer's mempool.

A Deposit Transaction is initiated on bitcoin, and more details of its lifecycle are described in the [bitcoin bridge documentation](/how-alpen-works/bitcoin-bridge.md#deposit).

## Alpen Block building <a href="#block-building" id="block-building"></a>

The Alpen sequencer orders transactions from its mempool using Reth's default algorithm, which prioritizes higher effective gas price while respecting per-account nonce ordering. It builds a block roughly every 5 seconds, executing the ordered transactions against the current Alpen state and processing any pending deposits and withdrawal requests in the same block. New blocks propagate back across the P2P network to full nodes.

At this point the transaction is included in an Alpen block that is considered pre-confirmed by the sequencer, not yet finalized on Bitcoin.

## Alpen Batch building and Proving <a href="#batch-building" id="batch-building"></a>

Roughly every 2 hours (every 1,440 blocks), the Alpen sequencer seals a range of blocks into a **batch** and performs three actions:

1. It posts the batch's aggregated state diff to Bitcoin as data availability (DA), so the resulting Alpen state can be reconstructed from Bitcoin on-chain data.
2. It produces a single SNARK proof, the **SNARK Account Update Proof**, that attests to two facts: the batch executed correctly, and the DA transaction is included in a valid Bitcoin block.
3. It submits that proof, with the relevant metadata, to the Strata sequencer's mempool as a **SNARK Account Update transaction**.

Any withdrawal requests produced while executing the batch are submitted with this update as outgoing messages addressed to the Strata system account that acts as the bridge gateway.

## Strata Block Building <a href="#proving" id="proving"></a>

The Strata sequencer receives SNARK Account Update transactions from the Alpen  sequencers (currently only Alpen). For each update it validates the SNARK proof and executes the update against the domain's committed state, advancing that account's state commitment and the value locked in it. The resulting transitions are recorded in a Strata block, which the sequencer produces roughly every 60 minutes.

Deposits flow in the opposite direction during this execution: deposits destined for the Alpen account are placed into that account's inbox queue, to be consumed by Alpen.

## Strata Checkpoint <a href="#proving" id="proving"></a>

Roughly every 10 hours, the Strata sequencer seals a range of Strata blocks into a **batch** and posts a **checkpoint** to bitcoin. The checkpoint carries the proven Strata state commitment, which in turn commits to Alpen's proven state, together with a SNARK proof attesting that the Strata blocks in the epoch executed correctly.

Withdrawal requests that reached Strata in this epoch are passed to the bridge through the checkpoint, where the bridge operators pick them up and settle them on bitcoin.

## Checkpoint Confirmation <a href="#batch-checkpoint" id="batch-checkpoint"></a>

A Strata full node reads checkpoints and other Strata protocol messages directly from bitcoin and reconstructs the canonical Strata chain. This state is marked as "Confirmed" on the Strata full node. An Alpen full node then derives its own canonical chain from the Strata full node's chain. Alpen blocks covered by batches committed by such checkpoints, but not yet finalized, are marked as "safe".&#x20;

## Finalization <a href="#batch-finalization" id="batch-finalization"></a>

Once the Bitcoin block containing a checkpoint is buried under a sufficient depth of subsequent blocks, a Strata full node marks the corresponding Strata checkpoint and its epoch as finalized. An Alpen full node then marks the Alpen blocks covered by the batches committed by this checkpoint, and every transaction in them, as finalized.


---

