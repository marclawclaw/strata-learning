---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/welcome/readme
---

# Introduction to Alpen

Alpen gives developers the freedom to program nearly any locking conditions for BTC imaginable, limited only by the Alpen block size and gas limits. This enables developers to create new kinds of applications for BTC with features such as:

* **New signature types**, "provide a valid `P-256` signature to authorize a transfer"
* **Vaults**, "transfers must wait `n` days after initiation to be effectuated, and can be cancelled in the mean time"
* **Subscriptions**, "address `0x123...9a` can withdraw up to `v` BTC per month from this account"
* **Strong privacy**, "transaction details are end-to-end encrypted and verified using a zero-knowledge proof"
* **Economically secured zero-confirmation payments**, "if a double-spend from this sender is reported, the reporter gets to claim the sender's full wallet balance"
* **Collateralized borrowing and lending**, "if enough BTC is locked as collateral to maintain up to `x%` loan-to-value ratio, then up to `y` of this other asset can be borrowed"

... and many more possibilities.

Technically speaking, **Alpen is a work-in-progress EVM-compatible validity rollup on bitcoin**. Let's break down what this means:

* **EVM-compatible**: The Alpen sequencer runs a client that is based on [Reth](https://github.com/paradigmxyz/reth), an Ethereum execution client. Alpen is [fully compatible](/how-alpen-works/comparing-strata-and-ethereum.md#evm-compatibility) with the EVM spec up to the Prague release.
* **Validity rollup**: Every Alpen state transition is proven to be valid using cryptographic validity proofs, which clients can use for fast, low-cost verification.
* **On bitcoin**: Alpen uses bitcoin for consensus and data availability. When an Alpen [checkpoint](/how-alpen-works/transaction-lifecycle.md#batch-checkpoint) gets confirmed on bitcoin, the only way to reorganize this block is to reorganize the bitcoin block that the Alpen checkpoint was confirmed in.

You can find detailed technical information in the [How Alpen works](/how-alpen-works/system-architecture.md) section of this documentation.

{% hint style="info" %}
During the current testnet phase, Alpen is running on the [default bitcoin signet](https://mempool.space/signet). Support for running Alpen on bitcoin mainnet is planned for a future release.
{% endhint %}


---

