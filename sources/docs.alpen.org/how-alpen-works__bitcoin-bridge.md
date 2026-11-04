---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---

# Bitcoin bridge

{% hint style="info" %}
Although the Strata bridge is designed to run on bitcoin mainnet, currently it is only running on the default bitcoin signet. Any references to bitcoin or BTC in this documentation should be read as "bitcoin signet" or "signet BTC", respectively, when describing the current system.
{% endhint %}

Alpen's [orchestration layer](/how-alpen-works/strata-design-rationale.md) Strata has an enshrined bitcoin bridge that enables the transfer of BTC between the bitcoin and Alpen blockchains. The Strata bridge consists of a set of operators who co-sign transactions to move BTC into and out of the bridge. Unlike other multisignature-based bitcoin bridges that require multiple operators (typically a majority) to be functional to preserve withdrawal liveness, the Strata bridge uses a [garbled lock ("glock")-based protocol](https://eprint.iacr.org/2026/812) that requires only a *single* functional operator to guarantee both deposit safety and withdrawal liveness. Strata bridge operators stake a slashable bond that is used to both compensate watchtowers who successfully challenge false operator claims and to disincentivize the operator from deviating from the bridge protocol.

This document gives a high-level overview of the bridge design and operation. A fully-detailed description will be added at a later stage.

## Overview <a href="#overview" id="overview"></a>

At a high level, we can divide the bridge operations into three stages:

1. A setup stage where operators stake the bond and establish garbled circuits with each other operator
2. A deposit stage where operators process a user request to deposit BTC into the Alpen blockchain, and
3. A withdrawal stage where funds are withdrawn from the bridge after a user initiates a request to withdraw BTC from Alpen to the bitcoin blockchain.

{% hint style="info" %}
Deposits and withdrawals are allowed in a fixed denomination of BTC, a value predefined by the bridge consensus rules. For the current testnet, this is set to 2 BTC. The withdrawal fee is set at 0.003 BTC. The values for mainnet will be announced at a later stage.
{% endhint %}

Deposit UTXOs are indexed chronologically by order of arrival, and subsequently withdrawals are processed by the first-in-first-out method. That is, the withdrawal transaction input is the UTXO for the lowest deposit index that the bridge currently possesses.

The deposit flow is composed of a deposit request that is processed by the operator set using a deposit transaction. The user must provide an Alpen address to receive their deposit on Alpen.

Later, when a user wants to withdraw their BTC from Alpen, the user's withdrawal request is assigned to an operator. The operator fulfills the request by sending the user an equivalent amount of BTC from the operator’s own address (i.e. not directly from the bridge), minus a predetermined fulfillment fee. After this fulfillment transaction is sufficiently confirmed on bitcoin, the operator is eligible to request a reimbursement by creating a transaction spending BTC from the bridge address to the operator's own address. This withdrawal reimbursement transaction is subject to the glock-based challenge-response game to ensure correct bridge operations. An operator does not need cooperation from other operators to fulfill or be reimbursed for a withdrawal.

{% hint style="info" %}
Bridge users can optionally assign their withdrawal request to a specific operator. If they do not assign the request to a specific operator, then the request will be randomly assigned to an operator who is part of the signer set on the oldest UTXO (to respect FIFO UTXO processing). If the user assigns their withdrawal to an operator who is not on the signer set of the oldest UTXO, then the withdrawal request will be reassigned to a random operator who is part of the given UTXOs signer set.
{% endhint %}

{% hint style="info" %}
All the multisigs in the deposit flow follow the [MuSig2](https://github.com/bitcoin/bips/blob/master/bip-0327.mediawiki) protocol, where signatures are aggregated into a single signature that validates all `N` operators' partial signatures.
{% endhint %}

## Deposit <a href="#deposit" id="deposit"></a>

The deposit process is initiated by the user with a deposit request transaction. This transaction sends 2[^1] BTC to a P2TR address, where:

1. The key path is unspendable, following [BIP 341](https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki#constructing-and-spending-taproot-outputs) with a "NUMS" point.
2. The script path has two paths:
   1. "Deposit path", an N-of-N multisig path, where N is the number of bridge operators. Note that this uses MuSig2 Schnorr key aggregation, hence using a single aggregate signature that validates all N operators' partial signatures.
   2. "Reclaim path", which allows the user to reclaim their funds if the bridge fails to move funds into the bridge address within 36 blocks i.e. it is time-locked and the user can spend it by providing a signature approximately 6 hours after their initial request.

Apart from the 2 BTC deposit output, the deposit request transaction has some metadata attached to it in the form of an `OP_RETURN` output that must be the **first** output of the transaction in order for the Alpen sequencer and bridge nodes to be able to detect the transaction. The `OP_RETURN` output contains magic bytes that identify the transaction as a Strata transaction plus the user's x-only public key and Alpen destination information, which enable the operators and the protocol, respectively, to process the deposit request.

In addition to the previous outputs, the user can add a third output for change.

The bridge monitors the bitcoin blockchain for incoming transactions, which can be detected by the `OP_RETURN` output. Once the bridge detects the deposit request, it verifies the tapleaf hash, the multisig pubkey path, and the full Taproot commitment.

If the deposit request is valid, the bridge operators fulfill the deposit, via the "deposit path", with a deposit transaction that transfers the BTC to the bridge (that is, to the bridge-owned UTXO set), and the user is minted the same amount of BTC to the Alpen address provided in the deposit request. The BTC is minted after the deposit has sufficiently-many block confirmations on bitcoin. This block confirmation "finality depth" is set to 12 confirmations.

After the deposit request transaction and before the deposit transaction, the glock transaction graph for the deposit is generated and signed by each operator and then distributed (offchain) to all operators. The full details on the transaction graph are out of scope for this doc and will be shared in future documentation. An operator will not sign the deposit transaction before they have the transaction graph signed by all other operators.

The flow of a user deposit is depicted by the following diagram:

<figure><img src="https://3327534418-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FAG4aXq7kxghwzduFy1lP%2Fuploads%2Fg0s4i9wmTjPdOvSc2vVE%2F2.png?alt=media&amp;token=d358ff63-220f-4ce4-b662-0505bea208af" alt=""><figcaption><p>Bridge deposit flow</p></figcaption></figure>

## Withdrawal <a href="#withdrawal" id="withdrawal"></a>

The withdrawal process is initiated by the user with a withdrawal request on Alpen. The full withdrawal process has two stages, where the second can start only after the first is completed:

1. The user requests a withdrawal on Alpen and an operator is assigned to fulfill the request on bitcoin — the operator "fronts" the user BTC out of the operator's own address with a withdrawal fulfillment transaction.
2. The assigned operator submits a claim transaction to the bridge to claim back the BTC corresponding to the withdrawal fulfillment.

### The user withdrawal

1. The user initiates a withdrawal request, which can only be made for the bridge's preconfigured BTC denomination. This is done by sending 2 BTC to the [bridge precompile address](/how-alpen-works/specifications.md#other-notable-addresses-) on Alpen, and as a result the BTC is "burned" on Alpen.
2. After the Strata checkpoint that proves the withdrawal request reaches a maturity depth of 12 blocks, the assigned operator makes a withdrawal fulfillment on bitcoin that sends the user the requested BTC minus an agreed-upon fee. (Read more about Strata checkpoints [here](/how-alpen-works/transaction-lifecycle.md#batch-checkpoint).)

{% hint style="info" %}
If the assigned operator does not fulfill the withdrawal request with an up-front payment (e.g. they do not have sufficient funds), a new operator is assigned after a timeout. The timeout is set to be 504 blocks (approximately 3.5 days).
{% endhint %}

### The operator claim

1. The operator submits the claim transaction to bitcoin.
2. If the claim is not challenged by another operator — fulfilling the role of a "watchtower", in glock parlance — within the challenge period (which lasts 1,064 blocks, approximately 1.4 weeks), then after the challenge period elapses the operator can spend the bridge payout transaction.
3. If the claim is challenged by one of the watchtowers within the challenge period then the challenge-response game starts, which allows the protocol to determine who is at fault. If the challenge is correct i.e. the operator submitted a false claim, the operator will not be able to spend the bridge payout and becomes eligible to lose their stake; if the challenge is incorrect, then the operator will eventually be eligible to spend the payout. After defending their claim, the operator has approximately 1.5 weeks to spend the payout before the protocol assumes the operator's claim was false and the operator becomes eligible to lose their stake.

The withdrawal flow is depicted at a high level by the following diagram:

<figure><img src="https://3327534418-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FAG4aXq7kxghwzduFy1lP%2Fuploads%2F6Ecp92hSntgDOBMmfLQJ%2F3.png?alt=media&amp;token=d7372634-77d9-4011-a33f-659f7d0d28c3" alt=""><figcaption><p>Bridge withdrawal flow. The user makes a withdrawal request on Alpen, which 'burns' the funds. Then, on bitcoin, the operator makes a transfer to the user, then starts the challenge-response game by submitting a claim transaction, claiming funds from the bridge.</p></figcaption></figure>

## Strata's challenge-response game <a href="#bitvm" id="bitvm"></a>

A claim transaction initiates an onchain challenge-response game. In the following diagram we give a high-level depiction of the withdrawal state machine, where challenges are made by the "Watchtower" and the response is given by the “Operator”. The operator needs to make a true claim, otherwise the watchtower can challenge the claim using a challenge transaction. The operator must then respond to the challenge with a bridge proof transaction. A watchtower must respond with a counterproof transaction. The operator has one last chance to defend their claim using a "watchtower NACK" transaction to disprove the counterproof. If the operator's claim was false and the watchtower was correct, then the watchtower can submit a "watchtower ACK" transaction that will block the operator's payout and eventually open up the operator's stake to slashing, which the watchtower can then claim as a reward for protecting the bridge.

A fully-detailed description of how glocks are used in the Strata bridge will be shared in future documentation.

<figure><img src="https://3327534418-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FAG4aXq7kxghwzduFy1lP%2Fuploads%2F5dNmO1ZJz6SUEqkEBgW3%2F4.png?alt=media&amp;token=8169412b-77c0-4252-b43f-1f1ca84fa74f" alt=""><figcaption><p>Glock state-machine (simplified). Yellow boxes indicate bitcoin transactions (or the lack of a transaction), black circles represent start/end points, black diamonds represent a decision point (only one path can be taken), text on arrows states who submits the corresponding transaction.</p></figcaption></figure>

[^1]: In addition to the 2 BTC deposit amount, the user must pay for the bitcoin network fees for both the Deposit Request Transaction and the Deposit Transaction.


---

