---
tags: [alpen, reference]
source: https://docs.alpen.org/how-alpen-works/specifications
---
# Alpen Specifications

Developer-facing parameters, current testnet values:

- **EVM Chain ID**: `20310`
- **Block time**: 5 s
- **Block gas limit**: 36,000,000

Notable addresses:
- `0x5400000000000000000000000000000000000001`: **bridge withdrawal precompile**. Send the fixed denomination here to burn and withdraw ([[Bridge Withdrawal Flow]]).
- `0x5400000000000000000000000000000000000002`: **Schnorr signature verification** precompile.
- `0xA2903243F20eEdE1839Cb07Ee5ce6b2Ba5820000`: WETH9-style wrapper for BTC.

All upstream Prague precompiles are supported except `0x0a` (KZG).

Related: [[Alpen EVM Compatibility]]
