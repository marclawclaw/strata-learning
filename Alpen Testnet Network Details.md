---
tags: [alpen, reference, testnet]
source: https://docs.alpen.org/welcome/get-started
---
# Alpen Testnet Network Details

Testnet III values:

| | |
|---|---|
| Network name | Alpen Testnet |
| Chain ID | **20310** (`0x4f56`) |
| Currency | Signet BTC, **sBTC** |
| Alpen RPC | `https://alpen.testnet.alpen.org`, or `http://localhost:9545` with your own [[Alpen Full Node]] |
| Strata RPC | `https://strata.testnet.alpen.org` |
| Bundler RPC | `https://bundler.testnet.alpen.org` |
| Explorer (Blockscout) | `https://explorer.testnet.alpen.org` |
| Signet explorer | `https://mempool.space/signet/` |
| Status dashboard | `https://status.testnet.alpen.org` |

- **Regenesis**: the previous testnet (chain ID **8150**) was decommissioned on **2026-08-18**. All state, accounts and contracts were reset.
- Wallets known to work: MetaMask, Enkrypt, Safe (Protofire).
- Underlying L1: bitcoin's default public signet, "Signet Global Test Net VI".
- Getting sBTC:
  - Use the Powcoins faucet (github.com/ajtowns/powcoins), then bridge with the [[Alpen CLI]]. This needs ≥2 BTC and takes up to ~15 h.
  - Or request small amounts via the form at tally.so/r/aQKLZZ.

Related: [[Alpen Specifications]], [[Alpen RPC Methods]], [[Strata RPC Methods]]
