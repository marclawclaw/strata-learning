---
tags: [alpen, rpc, reference]
source: https://docs.alpen.org/build-alpen-apps/rpc-endpoints
---
# Alpen RPC Methods

Alpen full nodes expose the **standard Reth JSON-RPC** plus one Alpen-specific method.

## `alpen_getBlockStatus(block_hash)`
Returns the finalization status of an Alpen (EE) block, abstracting over Strata checkpoint progress.
- `status`:
  - `pending`: not covered by a confirmed or finalized checkpoint.
  - `confirmed`: covered by a confirmed [[Strata Checkpoint]].
  - `finalized`: covered by a finalized checkpoint, so it can't be reorged without reorging bitcoin.
- `checkpoint_epoch`: the Strata epoch (uint32). It is empty while pending, and can be used with [[Strata RPC Methods]] to inspect the checkpoint.
- Unknown or non-canonical hash → `invalid params` / `block not found`. Genesis (block 0) is always `finalized`.

This is the programmatic view of [[Alpen Finality Levels]].

## Account abstraction (ERC-4337)
EntryPoint addresses:

| Version | Address |
|---|---|
| v0.6 | `0x5ff137d4b0fdcd49dca30c7cf57e578a026d2789` |
| v0.7 | `0x0000000071727De22E5E9d8BAf0edAc6f37da032` |
| v0.8 | `0x4337084D9E255Ff0702461CF8895CE9E3b5Ff108` |
| v0.9 | `0x433709009B8330FDa32311DF1C2AFA402eD8D009` |

The bundler is **Rundler-based** (Alchemy, v0.11.0) at `https://bundler.testnet.alpen.org`. It follows **EIP-7769**: `eth_sendUserOperation`, `eth_estimateUserOperationGas`, `eth_getUserOperationByHash`, `eth_getUserOperationReceipt`, `eth_supportedEntryPoints`.
