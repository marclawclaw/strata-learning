---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---

# Protocol administration and safeguards

The Alpen protocol has several system parameters that can and likely will be updated over time, such as the bridge withdrawal denomination and the sequencer public key. To enable these updates, we define an *administration* *system* with designated administrators who have the authority to update these parameters. Currently the updates are unconstrained, with the exception of an activation timelock that applies in some cases (described below). Whether or not to adopt additional parameter update constraints e.g. within a given range or limit, and what those constraints should be, remains an open area of investigation.

Reflecting the early stage of Alpen's development and rollout, the protocol additionally has several safeguards in place to manage and mitigate risk. These safeguards often present a tradeoff where we must decide that one risk is "less bad" than another. Over time, as additional hardening mechanisms are incorporated and the protocol runs for increasingly long periods without critical vulnerabilities discovered, we expect to relax or even remove these safeguards, with the ideal end state being that the protocol is fully secured by cryptographic code rather than humans-in-the-loop. We will keep this page up to date as we progress toward this ideal.

{% hint style="info" %}
On testnet, Alpen Labs controls all of the multisig keys described below. We will publish information about the mainnet keyholders and protocol parameters as we approach the mainnet release date.
{% endhint %}

### The Alpen Administrator

One of the foundational features of the Alpen protocol is *verification key mutability aka "upgradeability"*, which enables the rules of the protocol to be updated "in-place" without requiring manual migration of assets and account state from one version to the next. In proof systems such as the one used to enforce the correctness of Alpen protocol operations, the proof system is split into two distinct programs: a "prover" and a "verifier". One of the parameters within the verifier program is a cryptographic verification key that is deterministically linked to the "guest program" whose correct operation is being proven. Changing even a single byte of the guest program will change the verification key, making mutability useful for both bug fixes as well as adding new features to the guest program, making the mutability feature dual-purpose in nature.

In the event that a verification key update is required to add a new feature or fix a bug in the Alpen protocol, there is a role in the protocol called the Alpen Administrator that has the authority to update the verification key that is used to verify the checkpoints produced by the Alpen sequencer. The Alpen Administrator also has the authority to update the Alpen Administrator public keys.

Operationally, the Alpen Administrator is a M-of-N multisig that submits transactions on bitcoin that embed specially-formatted messages onchain to initiate protocol updates. An Alpen Administrator update must reach a confirmation depth of 2016 blocks (approximately two weeks), then it is automatically enacted by Strata full nodes; at the time of enactment, Alpen full nodes must run full node software that is compatible with the update to remain in consensus. During the 2016-block enactment delay, the Alpen Administrator can submit another bitcoin transaction to cancel the pending update, preventing it from being enacted.

### The Strata Administrator

Similar to the Alpen Administrator's role in the Alpen protocol, Strata has a Strata Administrator that has the authority to update the verification key that is used to verify Strata checkpoint proofs produced by the sequencer. Additionally, the Strata Administrator has the authority to update:

* Bridge and administration rules
* Strata Administrator public keys
* Bridge operator public keys
* Security Council public keys
* Payout Administrator public keys, and
* the Safe Harbor address.

\
(Read the relevant sections below for more details about these roles.)

Similar to the Alpen Administrator, the Strata Administrator is a M-of-N multisig that submits transactions on bitcoin that embed specially-formatted messages onchain to initiate protocol updates. A Strata Administrator update must reach a confirmation depth of 2016 blocks (approximately two weeks), then it is automatically enacted by Strata full nodes; at the time of enactment, Strata full nodes must run full node software that is compatible with the update to remain in consensus. During the 2016-block enactment delay, the Strata Administrator can submit another transaction on bitcoin to cancel the pending update, preventing it from being enacted.

Although the Alpen Administrator and Strata Administrator are defined as two different roles in the protocol, in practice, at least in the early days of the protocol's existence, the signer set for each multisig will be the same to manage operational overhead and simplify protocol administration.

### The Payout Administrator

In the current version of the Strata bridge node software, a watchtower is only able to produce a counterproof that blocks a payout for a given faulty operator once. After that, the given operator is able to use information revealed by the watchtower to nack counterproofs in future challenge-response games, even if the operator's claims are false. Since the bridge relies on a 1-of-N functional watchtower assumption for safety against false operator claims, even a single watchtower being unable to block payouts for a given operator's false claims is a violation of the bridge's safety assumption. We have some ideas about how to overcome this limitation and will share more once we have settled on a specific solution.

The Payout Administrator is a role in the Strata protocol that exists to prevent safety violations in the event that an operator submits multiple false claims to the bridge. The Payout Administrator in an "honest minority" M-of-N multisig, where M < N/2, that has the authority to spend a connector UTXO that the operator needs to provide as an input to the payout transaction for a given claim. By spending this connector UTXO, the Payout Administrator makes it impossible for the operator to receive a payout for the given claim.

The reason for using an "honest minority" is that it requires a malicious bridge operator to collude with a majority of the Payout Administrator multisig signers rather than a minority in order to prevent the multisig from reaching quorum to block a payout. For example, if the Payout Administrator was a 4-of-6 multisig, then a malicious bridge operator would need to collude with only 3 signers to prevent the multisig from reaching quorum. Whereas in the case of a 2-of-6 multisig, a malicious bridge operator needs to collude with 5 signers to prevent the multisig from reaching quorum.

### The Strata Sequencer Manager

The Strata sequencer is the entity given the authority to sequence Strata transactions by producing Strata checkpoints on bitcoin. The public key of the Strata sequencer is hardcoded in the Strata protocol and used as part of the checkpoint verification process, to ensure that random computers on the internet cannot hijack Strata block production. To change the sequencer public key, in case of key compromise or routine key rotation, there is a role in the Strata protocol called the Strata Sequencer Manager that has the authority to update the canonical sequencer public key. The Strata Sequencer Manager also has the authority to update the Sequencer Manager public keys.

Operationally, the Strata Sequencer Manager is an M-of-N multisig that submits transactions on bitcoin to change the sequencer key. Sequencer key update transactions are enacted at the same block height at which they are confirmed to minimize network downtime during a sequencer key update.

### Security Council and Safe Harbor

The enactment delay for Strata Administrator updates provides bridge users an opportunity to exit the protocol if they don't like a given update, but it also introduces a risk: if there is a critical vulnerability that could result in funds being immediately drained from the bridge by an attacker, and a Strata administrator update transaction is used to fix the vulnerability, then there is a 2016 block window during which the attacker could exploit the vulnerability.

To close this gap, there is a role in the Strata protocol called the Security Council, which is an M-of-N multisig that has the authority to confirm a transaction that will signal the Bridge Operators to either immediately, or with 72-hour delay (depending on the signal given), use the N-of-N cooperative spending path to sweep all bridge UTXOs to the Safe Harbor address. The Safe Harbor address is an M-of-N multisig that is hardcoded into the Strata protocol, and, as mentioned above, can only be changed by the Strata Administrator, preventing the Security Council from tricking Bridge Operators into sweeping the bridge UTXOs into an attacker's wallet.

Once the bridge UTXOs are swept into the Safe Harbor address, they should be held there until the vulnerability is fixed using a Strata Administrator update. Once the vulnerability is fixed, the Safe Harbor multisig can transfer the bridge UTXOs back to the bridge, and normal operation of bridge can resume from there.


---

