---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/build-alpen-apps/troubleshooting
---

# Troubleshooting

## Common issues and solutions <a href="#common-issues-and-solutions" id="common-issues-and-solutions"></a>

### 1. Contract calls fail with `EVM error: NotActivated` <a href="#id-1-contract-calls-fail-with-evm-error-notactivated" id="id-1-contract-calls-fail-with-evm-error-notactivated"></a>

The contract has been compiled at a newer EVM version than supported by Alpen. Ensure the EVM version of the compiler is set to `Prague`.

### 2. I get `Could Not Fetch chain ID` error on my wallet <a href="#id-1-contract-calls-fail-with-evm-error-notactivated" id="id-1-contract-calls-fail-with-evm-error-notactivated"></a>

Alpen Testnet did a network regenesis with a different chain ID. Check that your wallet is using the current parameters in the [Get started](/welcome/get-started.md#network-details) page.

### 3. Alpen CLI returns `Failed to sync signet wallet` error <a href="#id-1-contract-calls-fail-with-evm-error-notactivated" id="id-1-contract-calls-fail-with-evm-error-notactivated"></a>

Check that your Alpen CLI configuration file `~/.config/alpen/config.toml` uses the correct endpoints from [Using the Alpen CLI](/welcome/using-the-alpen-cli.md#user-content-configuration), then follow the instructions to backup, clean up, restore, and scan again your CLI wallet balances.

### 4. My full node is stuck at 0 peers <a href="#id-1-contract-calls-fail-with-evm-error-notactivated" id="id-1-contract-calls-fail-with-evm-error-notactivated"></a>

Verify that your configuration file in `alpen/docker/.env` is using the correct Alpen bootnodes and other parameters, as indicated in the [Running an Alpen Full Node](/running-an-alpen-node/alpen-full-node-operator-guide.md) documentation page. If your file had wrong parameters, make sure to reinstall following the instructions at [Running an Alpen Full Node](/running-an-alpen-node/alpen-full-node-operator-guide.md#cleaning-up)


---

