---
tags: [alpen, reference, howto]
source: https://docs.alpen.org/build-alpen-apps/troubleshooting
---
# Alpen Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `EVM error: NotActivated` | The contract was compiled for an EVM newer than Alpen supports. Set the compiler to **Prague** ([[Deploying Contracts on Alpen]]) |
| Wallet: `Could Not Fetch chain ID` | The regenesis changed the chain ID from 8150 to **20310**. Update the network ([[Alpen Testnet Network Details]]) |
| CLI: `Failed to sync signet wallet` | Wrong endpoints in `config.toml`. Fix them, then backup → cleanup → restore → `scan` ([[Alpen CLI]]) |
| Full node stuck at 0 peers | Wrong bootnodes or `--trusted-peers`, or firewall on 30303 tcp/udp. Reinstall clean ([[Alpen Full Node]]) |
| Full node "OL tracker errors" | `client_url` or `sequencer_pubkey` don't match the network. Check `strata_getChainStatus` |
