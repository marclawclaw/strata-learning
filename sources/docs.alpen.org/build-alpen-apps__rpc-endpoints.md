---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/build-alpen-apps/rpc-endpoints
---

# RPC endpoints

Our RPC endpoints adhere to the [JSON-RPC 2.0 specification](https://www.jsonrpc.org/specification). Public full nodes are located at the following addresses:

* Alpen RPC: `https://alpen.testnet.alpen.org`&#x20;
* Strata RPC: `https://strata.testnet.alpen.org`&#x20;

For example, to call the `strata_getChainStatus` method, you can use `curl`:

```sh
curl --location 'https://strata.testnet.alpen.org' \
--header 'Content-Type: application/json' \
--data '{
  "jsonrpc":"2.0",
  "method":"strata_getChainStatus",
  "params":[],
  "id": 1
}'
```

## Alpen client methods <a href="#client-methods" id="client-methods"></a>

In addition to the [standard Reth API methods](https://reth.rs/jsonrpc/intro), Alpen full nodes make available the following Alpen-related method:

| RPC method             | Description                               | Link                                           |
| ---------------------- | ----------------------------------------- | ---------------------------------------------- |
| `alpen_getBlockStatus` | Finalization status of a given block hash | [alpen\_getBlockStatus](#alpen_getblockstatus) |

### `alpen_getBlockStatus` <a href="#alpen_getblockstatus" id="alpen_getblockstatus"></a>

Returns the finalization status of a given block hash, abstracting Strata's checkpoint progress.

Availability: Public Alpen RPC, and Alpen RPC on Alpen full nodes.

Parameters:

* `block_hash`: `ee-block-hash`&#x20;

Request:

```
{
  "jsonrpc": "2.0",
  "method": "alpen_getBlockStatus",
  "params": [
    "0x<ee-block-hash>"
  ],
  "id": 1
}
```

Returns:

* `status` one of:
  * `pending`
  * `confirmed`
  * `finalized`&#x20;
* `checkpoint_epoch` either:
  * `uint32` Strata checkpoint epoch number
  * empty if the status is still `pending`

Status meanings:

* `pending`: the Alpen block is not yet covered by any confirmed or finalized by a Strata checkpoint
* `confirmed`: the Alpen block is covered by a confirmed Strata checkpoint
* `finalized`: the Alpen block is covered by a finalized Strata checkpoint

Notes:

* *Finalized* indicates that an Alpen block cannot be reorganized without reorging the bitcoin block it was confirmed in. Find more information in the [transaction lifecycle](/how-alpen-works/transaction-lifecycle.md) docs page.
* The block hash must resolve to a canonical Alpen block on the serving node.
* Unknown or non-canonical block hashes return an `invalid params` error with message `block not found`.
* Genesis block `0` is always reported as `finalized`.
* The `checkpoint_epoch` can be used to query the Strata RPC and inspect the checkpoint on Strata.

## Strata client methods <a href="#client-methods" id="client-methods"></a>

Strata RPC exposes `strata_*` methods at the dedicated endpoint `https://strata.testnet.alpen.org` to query the [Strata orchestration layer](/how-alpen-works/strata-design-rationale.md).

Endpoints marked as available in a *checkpoint-sync node* are exposed by any [Alpen full node](/running-an-alpen-node/alpen-full-node-operator-guide.md). Other methods are only available on the public RPC.

Strata client methods can be used to inspect Strata's state, snark account activity, checkpoints, and block data, independently verifying transaction finalization status and bridge transactions.

{% hint style="info" %}
Alpen is represented in Strata as a snark account with account ID `0101010101010101010101010101010101010101010101010101010101010101`
{% endhint %}

<table><thead><tr><th width="281.13671875">RPC method</th><th>Description</th><th>Link</th></tr></thead><tbody><tr><td><code>strata_getChainStatus</code></td><td>Returns the current Strata chain status, including tip, confirmed, finalized, and latest epoch frontiers.</td><td><a href="#strata_getchainstatus">strata_getChainStatus</a></td></tr><tr><td><code>strata_getCheckpointInfo</code></td><td>Returns checkpoint coverage and confirmation status for a given epoch.</td><td><a href="#strata_getcheckpointinfo">strata_getCheckpointInfo</a></td></tr><tr><td><code>strata_getAccountEpochSummary</code></td><td>Returns a snark account’s epoch summary, including final balance, final state root, epoch commitments, and update inputs.</td><td><a href="#strata_getaccountepochsummary">strata_getAccountEpochSummary</a></td></tr><tr><td><code>strata_getAccountGenesisEpochCommitment</code></td><td>Returns the Strata epoch commitment for the epoch in which an account was created.</td><td><a href="#strata_getaccountgenesisepochcommitment">strata_getAccountGenesisEpochCommitment</a></td></tr><tr><td><code>strata_getAsmManifestCommitment</code></td><td>Returns the canonical ASM manifest commitment for a given L1 block height.</td><td><a href="#strata_getasmmanifestcommitment">strata_getAsmManifestCommitment</a></td></tr><tr><td><code>strata_getBlocksSummaries</code></td><td>Returns account-specific summaries across the requested Strata block range.</td><td><a href="#strata_getblockssummaries">strata_getBlocksSummaries</a></td></tr><tr><td><code>strata_getSnarkAcctInboxMsgRange</code></td><td>Returns indexed inbox messages for a snark account over the requested range.</td><td><a href="#strata_getsnarkacctinboxmsgrange">strata_getSnarkAcctInboxMsgRange</a></td></tr><tr><td><code>strata_getSnarkAccountStateByTag</code></td><td>Returns snark account state at the requested finalization tag.</td><td><a href="#strata_getsnarkaccountstatebytag">strata_getSnarkAccountStateByTag</a></td></tr><tr><td><code>strata_getSnarkAcctUpdateManifest</code></td><td>Returns published manifest metadata for a specific snark account update.</td><td><a href="#strata_getsnarkacctupdatemanifest">strata_getSnarkAcctUpdateManifest</a></td></tr></tbody></table>

### `strata_getChainStatus`

Returns tip, confirmed, finalized, and latest completed epoch state.

Availability: Public Strata RPC, and local checkpoint-sync node.

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getChainStatus",
  "params": [],
  "id": 1
}
```

Returns:

* `tip`: current Strata tip block info with `blkid`, `slot`, `epoch`, and `is_terminal`
* `confirmed`: confirmed epoch commitment
* `finalized`: finalized epoch commitment
* `latest`: latest epoch terminal processed by fork choice

Use this when you need a current read point for follow-up calls such as `strata_getBlocksSummaries` or `strata_getSnarkAccountStateByTag`.

Notes:

* Strata `slot`s can be considered equivalent to blocks in general blockchain terminology.
* On Testnet III, each `epoch` contains `64` slots, each `slot` contains one block, and Strata's blocktime is `60 minutes`. Find more information in the [transaction lifecycle](/how-alpen-works/transaction-lifecycle.md) docs page.

### `strata_getCheckpointInfo`

Returns checkpoint coverage and confirmation status at the requested epoch.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `epoch`: `uint32`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getCheckpointInfo",
  "params": [42],
  "id": 1
}
```

Returns `null` if the epoch does not exist. Otherwise returns:

* `idx`: checkpoint index
* `l1_range`: inclusive L1 (bitcoin) block commitment range covered by the checkpoint
* `l2_range`: inclusive L2 (Strata) block commitment range covered by the checkpoint
* `confirmation_status`: one of:
  * `pending`
  * `confirmed`
  * `finalized`

### `strata_getAccountEpochSummary`

Returns a snark account’s summary at the requested epoch.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId` string in hex format, without `0x`
* `epoch`: `uint32`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getAccountEpochSummary",
  "params": [
    "<account-id-hex>",
    42
  ],
  "id": 1
}
```

Returns:

* `epoch_commitment`
* `prev_epoch_commitment`
* `final_balance`
* `final_state_root`
* `update_inputs`

Notes:

* This call is intended to query a specific `<account-id-hex>` snark account, use Alpen's [parameter above](#client-methods-1) for reference.
* `update_inputs` is an array returning sequence numbers, consumed messages, extra data, and state roots when available. If no inputs are available, it returns an empty array.

### `strata_getAccountGenesisEpochCommitment`

Returns the Strata epoch commitment for the epoch in which an account was created. Provides a canonical starting point for replaying a snark account's state history.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getAccountGenesisEpochCommitment",
  "params": [
    "<account-id-hex>"
  ],
  "id": 1
}
```

Returns:

* `epoch`
* `last_slot`
* `last_blkid`

### `strata_getAsmManifestCommitment`

Returns the canonical ASM manifest commitment for an L1 block height, if the node has indexed ASM manifest data for that height.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `l1_height`: `uint32`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getAsmManifestCommitment",
  "params": [13428],
  "id": 1
}
```

Returns:

* `null` if the node has not indexed a manifest for that height yet
* otherwise a `HexBytes32` manifest tree-hash root

### `strata_getBlocksSummaries`

Tracks snark account balance, sequence number, inbox position, updates, and new inbox messages over time.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId`
* `start_slot`: `uint64`
* `end_slot`: `uint64`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getBlocksSummaries",
  "params": [
    "<account-id-hex>",
    100,
    110
  ],
  "id": 1
}
```

Returns an array of objects with:

* `account`
* `block_commitment`
* `balance`
* `next_seq_no`
* `updates`
* `new_inbox_messages`
* `next_inbox_msg_idx`

Notes:

* The range is inclusive: `[start_slot, end_slot]`.
* Results are returned in ascending slot order.
* Only canonical-chain blocks are included.
* `next_seq_no` and `next_inbox_msg_idx` are zero for non-snark accounts.
* The requested range is capped by the node’s `max_headers_range` setting.

### `strata_getSnarkAcctInboxMsgRange`

Returns a snark account's indexed inbox messages within the requested range.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId`
* `start`: `uint64`
* `end`: `uint64`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getSnarkAcctInboxMsgRange",
  "params": [
    "<account-id-hex>",
    0,
    25
  ],
  "id": 1
}
```

Returns an array of:

* `index`
* `value`

Each `value` contains:

* `source`
* `incl_epoch`
* `payload`

Notes:

* The range is half-open: `[start, end)`.
* Empty ranges return `[]`.
* Maximum interval is 1000 messages, larger calls are rejected.
* It can be used to independently inspect protocol-generated messages (such as deposits messages) or to generate proofs that Alpen consumed all the incoming messages in the right order.

### `strata_getSnarkAccountStateByTag`

Returns a specified snark account state at the requested Strata finalization tag.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId`
* `tag`: `latest`, `confirmed`, or `finalized`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getSnarkAccountStateByTag",
  "params": [
    "<account-id-hex>",
    "latest"
  ],
  "id": 1
}
```

Returns `null` if the account does not exist or is not a snark account. Otherwise returns:

* `seq_no`
* `inner_state`
* `next_inbox_msg_idx`
* `update_vk`

Notes:

* This method does not return account balance.
* Pair with `strata_getBlocksSummaries` method if you need the balance at a specific block slot.

### `strata_getSnarkAcctUpdateManifest`

Returns manifest metadata for a published snark account update.

Availability: Public Strata RPC, and local checkpoint-sync node.

Parameters:

* `account_id`: `AccountId`
* `seq_no`: `uint64`

Request:

```
{
  "jsonrpc": "2.0",
  "method": "strata_getSnarkAcctUpdateManifest",
  "params": [
    "<account-id-hex>",
    7
  ],
  "id": 1
}
```

Returns:

* `seq_no`
* `prev_next_msg_idx`
* `new_next_msg_idx`
* `new_inner_state_root` when stored by the serving node
* `extra_data` when stored by the serving node

Notes:

* The consumed inbox range is `[prev_next_msg_idx, new_next_msg_idx)`.
* If `seq_no` has not been published yet, the method returns not found.
* `new_inner_state_root` and `extra_data` may be `null` if the serving node does not retain update metadata for that sequence number.

## Account abstraction <a href="#account-abstraction" id="account-abstraction"></a>

Alpen supports account abstraction (AA) following the [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) standard. Testnet EntryPoint contracts and a public Bundler RPC are available below.

**EntryPoint Contract**

Developers can access it using the following addresses:

| Version | Contract Address                           |
| ------- | ------------------------------------------ |
| v0.6    | 0x5ff137d4b0fdcd49dca30c7cf57e578a026d2789 |
| v0.7    | 0x0000000071727De22E5E9d8BAf0edAc6f37da032 |
| v0.8    | 0x4337084D9E255Ff0702461CF8895CE9E3b5Ff108 |
| v0.9    | 0x433709009B8330FDa32311DF1C2AFA402eD8D009 |

**Bundler**

Alpen provides a [Rundler-based](https://github.com/alchemyplatform/rundler/releases/tag/v0.11.0) public bundler RPC at `https://bundler.testnet.alpen.org`.

For a quick test, call `eth_supportedEntryPoints` method using `curl`:

```sh
curl --location 'https://bundler.testnet.alpen.org' \
--header 'Content-Type: application/json' \
--data '{
  "jsonrpc":"2.0",
  "method":"eth_supportedEntryPoints",
  "params":[],
  "id": 1
}'
```

Bundler's RPC endpoints follow the [EIP-7769](https://eips.ethereum.org/EIPS/eip-7769) specification:

| Endpoint                                                                                               | Description                                            |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| [`eth_sendUserOperation`](https://eips.ethereum.org/EIPS/eip-7769#eth_senduseroperation)               | Sends a `UserOperation` to the bundler for inclusion.  |
| [`eth_estimateUserOperationGas`](https://eips.ethereum.org/EIPS/eip-7769#eth_estimateuseroperationgas) | Estimates gas usage for a given `UserOperation`.       |
| [`eth_getUserOperationByHash`](https://eips.ethereum.org/EIPS/eip-7769#eth_getuseroperationbyhash)     | Fetches a `UserOperation` by its `userOpHash`.         |
| [`eth_getUserOperationReceipt`](https://eips.ethereum.org/EIPS/eip-7769#eth_getuseroperationreceipt)   | Retrieves the receipt for a submitted `UserOperation`. |
| [`eth_supportedEntryPoints`](https://eips.ethereum.org/EIPS/eip-7769#eth_supportedentrypoints)         | Lists supported `EntryPoint` addresses.                |


---

