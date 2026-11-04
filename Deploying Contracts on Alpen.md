---
tags: [alpen, tooling, howto]
source: https://docs.alpen.org/build-alpen-apps/deploying-alpen-smart-contracts
---
# Deploying Contracts on Alpen

This is the standard Foundry flow. The Alpen-specific points:

- **Compile for Prague**: `forge test --evm-version "prague"`. A newer EVM target makes calls fail with `EVM error: NotActivated` ([[Alpen Troubleshooting]]).
- Set `ETH_RPC_URL=https://alpen.testnet.alpen.org`. Check that `eth_chainId` returns `0x4f56` (20310).
- Use a dedicated *deployer* keystore account: `cast wallet new-mnemonic` → `cast wallet import deployer --mnemonic …`. Always pass `--account` so the key stays out of clear text.
- Deploy: `forge create Counter --account deployer --broadcast --gas-price 2gwei --priority-gas-price 1gwei`. Without `--broadcast` it is a dry run.
- Value units: `cast send … --value 0.042ether` sends **0.042 sBTC**. "ether" just means the native 18-decimal unit.
- `cast balance … -e` shows the balance in BTC units.
- Verify on Blockscout: `forge verify-contract --verifier blockscout --verifier-url https://explorer.testnet.alpen.org/api/ …`

Funding: liquidity is scarce because signet BTC isn't controlled by Alpen. Bridge via the [[Alpen CLI]] or ask Alpen Labs ([[Alpen Testnet Network Details]]).

Related: [[Alpen EVM Compatibility]]
