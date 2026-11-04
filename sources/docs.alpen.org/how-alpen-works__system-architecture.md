---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/how-alpen-works/system-architecture
---

# System architecture

{% hint style="info" %}
Although Alpen is designed to run on bitcoin mainnet, currently Alpen is running on the default bitcoin signet. Until mainnet launch, any references to bitcoin or BTC in this documentation should be read as "bitcoin signet" or "signet BTC" when describing the current system.
{% endhint %}

## System Overview

<figure><img src="https://3327534418-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FAG4aXq7kxghwzduFy1lP%2Fuploads%2Fr70qced4gcPNVyGuHnAg%2FTN3Arch.png?alt=media&amp;token=7ebcbd6d-57e8-487c-b5f9-3f0d56e768bf" alt=""><figcaption></figcaption></figure>

The diagram above illustrates the overall Alpen system architecture and the flow of data between components and the entities interacting with the system. There are four layers:

1. **Bitcoin** at the base, providing data availability and settlement.
2. **Strata** above it: the orchestration blockchain that anchors state to bitcoin.
3. **Alpen** above that: an EVM-compatible blockchain that executes user transactions and posts proven state commitments to Strata.
4. **Applications** on top: wallets, apps, and tooling.

Arrows indicate the direction of data flow, and arrow labels indicate the data being exchanged. *Centralized services* are operated by Alpen Labs. The other components within the Alpen and Strata layers (bundlers and full nodes) can be run by anyone.

The system consists of **two chains**: a lightweight orchestration blockchain (**Strata**) and an EVM-compatible blockchain (**Alpen**). Alpen executes user transactions and periodically posts a proven state commitment to Strata; Strata aggregates these commitments and checkpoints them to bitcoin.

## Bitcoin

Bitcoin is the base layer and the system's source of truth. It has three roles:

* **Data availability.** The Alpen sequencer posts each Alpen batch's aggregated state diff to bitcoin, so anyone can reconstruct Alpen's state from onchain data. Additionally, the Strata sequencer publishes checkpoints (containing proven Strata state commitments and withdrawal intents) to bitcoin, so anyone can reconstruct Strata's state from onchain data.
* **Consensus.** Alpen and Strata full nodes use the batch and checkpoint data that is published and confirmed on bitcoin to come to consensus on what the current state of the network is. Given equal access to the same full node software and an unpartitioned network connection, full nodes will always arrive at the same current state. This is especially useful when sequencers, bridge operators, and watchtowers need to make proofs referencing the current state of the network.
* **Settlement.** Once the Strata sequencer publishes a checkpoint to bitcoin, and the checkpoint reaches the protocol-defined confirmation maturity depth, then the published state commitment, along with the transactions it covers, cannot be reorganized without reorganizing the bitcoin block that the checkpoint was confirmed in. Operators and watchtowers can then use bitcoin transactions to refer to the confirmed state in challenge-response games for the purpose of arbitrating disputes over the validity of operator and watchtower proofs/counterproofs.

The Strata bridge settles BTC deposits and withdrawals through bitcoin transactions, as described in the [bridge operator](#strata-bridge-operator) section below.

## Strata layer

Strata is a lightweight blockchain that posts its state updates to bitcoin. It is also referred to as the **Orchestration Layer (OL)**.

Strata's state is composed of **ledger accounts** and their inbox message queues. Each account corresponds to either a Strata **system account** (a precompile-like account with protocol-defined behavior) or a SNARK account. A SNARK account holds:

* a state commitment of the account's associated program,
* a verification key for the SNARK proof used to advance that state,
* an inbox message queue for the account program to consume, and
* the account's BTC balance.

A SNARK account program posts a commitment to its state to Strata, along with a validity proof attesting to the transition from the previously committed state to the new one. These commitments are checkpointed to bitcoin. Through its SNARK account, the program also gains access to the Strata bridge. While Strata can technically support multiple SNARK account programs, **Alpen is currently the only one deployed.**

Strata produces blocks roughly every 60 minutes and posts a checkpoint to bitcoin roughly every 10 hours.

### Anchor State Machine (ASM)

The Anchor State Machine (ASM) is a component of Strata that handles Strata's bitcoin onchain protocol and maintains the bitcoin-derived state. It is responsible for:

1. Parsing and validating checkpoints on bitcoin to determine the canonical Strata chain.
2. Tracking deposit and withdrawal requests.
3. Handling protocol updates.

The ASM is used internally by the Strata sequencer, full nodes, and bridge clients.

### Strata sequencer

The Strata sequencer is a centralized component that orders SNARK Account Update Transactions received from SNARK account users (in the context of Alpen, the Alpen sequencer), validates the account updates by verifying their SNARK proofs, and executes the updates against their committed state to produce Strata blocks. Periodically the Strata sequencer posts checkpoints of the Strata chain to bitcoin with a SNARK proof attesting to valid execution of the Strata protocol.

Withdrawal requests arrive alongside these SNARK account updates and are passed to the bridge through the checkpoints posted to bitcoin. Deposits destined for a specific SNARK account are processed during block execution and placed into that account's inbox queue.

On testnet, Alpen Labs operates the Strata sequencer.

### Strata full node

A Strata full node is client-side software that reads Strata checkpoints and other Strata protocol messages from bitcoin (through an ASM component) and reconstructs the canonical state of the Strata blockchain. It exposes an RPC interface that SNARK account users (e.g. Alpen full nodes) can use to read their canonical chain as committed on bitcoin.

### Strata bridge operator

A bridge operator is one of `N` bridge nodes that collectively process user deposits and withdrawals:

* **Deposits.** Each operator monitors bitcoin for a Deposit Request Transaction, in which the user specifies a destination address on Alpen and authorizes the operators to move their BTC deposit into an N-of-N multisig address controlled by the bridge operators within a specified time window. Once the operators observe this transaction, they pre-sign a set of transactions that ensure a functional operator can later fulfill a withdrawal and receive reimbursement. At the end of pre-signing, the operators sign and broadcast a Deposit Transaction that moves the user's funds into the N-of-N bridge address. When that transaction is confirmed and reaches the protocol-specified maturity depth, it gets read by Strata and then Alpen full nodes, and an equivalent amount of BTC is minted to the user's specified Alpen address.
* **Withdrawals.** A user initiates a Withdrawal Request Transaction, in which they specify a destination address on bitcoin and burn BTC on Alpen. Once this transaction is included in an Alpen batch and subsequently referenced in a Strata checkpoint that reaches the maturity depth, the protocol assigns an operator to process the withdrawal on bitcoin. The assigned operator initiates a multi-transaction withdrawal process that spends one bridge UTXO and delivers the user's BTC to their bitcoin address. If an operator processes a withdrawal incorrectly, the other operators act as watchtowers and challenge it.

The Strata bridge is a [glock-based](https://www.alpenlabs.io/blog/glock-verification-on-bitcoin) protocol. A detailed description can be found in the [bitcoin bridge documentation](/how-alpen-works/bitcoin-bridge.md).

## Alpen layer

Alpen is the EVM-compatible rollup blockchain where end-user transactions are executed. It has its own sequencer and full nodes. The Alpen sequencer produces blocks roughly every 5 seconds, and roughly every 2 hours the sequencer settles a batch of those blocks down to bitcoin and then references the batch in an account update on Strata. This is the chain that user-facing applications mainly interact with.

{% hint style="info" %}
Alpen's sequencer and reference full node client are built on [Reth](https://github.com/paradigmxyz/reth).
{% endhint %}

### Alpen sequencer

The Alpen sequencer orders EVM transactions from its mempool using Reth's default algorithm, which prioritizes higher effective gas price while respecting per-account nonce ordering. It builds a block every \~5 seconds, extending the Alpen chain and processing pending deposits and withdrawal requests.

Every \~2 hours the sequencer seals its recent blocks into a batch, then:

1. Publishes the batch's aggregated state diff in a transaction on bitcoin for data availability and consensus.
2. Produces a single SNARK proof (the **SNARK Account Update Proof**) attesting that the batch executed correctly *and* that the batch transaction is included in a valid bitcoin block in the canonical bitcoin blockchain.
3. Submits the proof and account update data to the Strata sequencer's mempool as a **SNARK Account Update Transaction**.

Withdrawal requests are carried as outgoing messages in this update, sent to the Strata system account that acts as the bridge gateway. Deposits from Alpen's inbox queue in Strata are also marked as consumed in this update.

On testnet, Alpen Labs operates the Alpen sequencer.

### Alpen full node

An Alpen full node is client-side software that runs Alpen's core logic and exposes an RPC interface for applications to interact with the Alpen blockchain. Transactions and blocks propagate across the network over a P2P layer: it forwards transactions it receives from the application layer or bundler toward the sequencer, and receives new blocks the same way, executing them against its current state to advance the chain.

A full node determines finality from bitcoin: once it observes (via a Strata full node) that the Strata checkpoint committing the relevant state has been buried to a sufficient depth on Bitcoin, it marks the corresponding Alpen state as finalized.

## Application layer <a href="#application-layer" id="application-layer"></a>

The application layer includes any interface or application that acts as a gateway for users to interact with Alpen. These applications include wallets (e.g. EOA wallets, account abstraction wallets, bitcoin wallets), decentralized applications (e.g. Uniswap), smart contract deployment tools (e.g. Foundry), and command-line interface tools (e.g. Cast). They interact programmatically with an Alpen full node, bundler, and bitcoin through an RPC interface provided by each service, and the RPC interfaces enable applications to interact with Alpen, including deploying smart contracts, sending EOA or account-abstraction transactions, and initiating deposits into Alpen.

### Bundler <a href="#bundler" id="bundler"></a>

The bundler supports account abstraction by accepting `UserOperation` transactions as defined by [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337#useroperation). It bundles these transactions and relays them to a full node via a special entrypoint contract deployed on Alpen.

Alpen Labs provides a bundler that developers can use as a service to send `UserOperation`s. Anyone else can also run their own bundler to connect with the full node network independently.

See [this section](/build-alpen-apps/rpc-endpoints.md#account-abstraction) for more information on the entrypoint contract and the RPC endpoints supported by the bundler.


---

