---
tags: [alpen, tooling, bridge]
source: https://docs.alpen.org/welcome/using-the-alpen-cli
---
# Alpen CLI

`alpen` is a Rust CLI wallet that holds a **signet wallet** and an **Alpen EVM wallet**, and drives the [[Strata Bridge]].

Install:
`cargo +nightly install --git https://github.com/alpenlabs/alpen alpen-cli --rev 5643210d… --locked --force`

**Testnet III config** (`~/.config/alpen/config.toml`). These values reveal bridge parameters:

```toml
esplora = "https://blockstream.info/signet/api/"
alpen_endpoint = "https://alpen.testnet.alpen.org"
mempool_endpoint = "https://mempool.space/signet"
blockscout_endpoint = "https://explorer.testnet.alpen.org"
bridge_pubkey = "1e070c9190e2dcc8671957722aec6f1533f47ce3bf2fcb53cda2961faf309049"  # N-of-N MuSig2 aggregate (x-only)
bridge_alpen_address = "0x5400000000000000000000000000000000000001"
bridge_fee_sats = 1000
finality_depth = 0
network = "signet"
magic_bytes = "ALPN"                  # OP_RETURN tag in the DRT
bridge_denomination_sats = 200000000  # 2 BTC
recovery_delay = 36                   # DRT reclaim timelock (blocks)
max_withdrawal_descriptor_len = 81
```

**Commands:** `deposit`, `withdraw [--amount] [--operator <idx>]`, `balance signet|alpen`, `send`, `receive`, `drain`, `recover`, `scan`, `backup` (BIP-39 mnemonic), `change-password`, `reset`, `config`, `debug`.

- **Deposit**: sends 2 BTC plus fees. It can take **up to 15 h** to appear on Alpen. If it hasn't arrived after that, run `recover`. Recovery is possible **36 blocks (~6 h)** after the deposit confirms, via the reclaim path ([[Deposit Request Transaction]]).
- **Withdraw**: the amount must be a **positive multiple** of 2 BTC. `--operator` picks the operator index ([[Bridge Withdrawal Flow]]).
- Addresses:
  - Signet receive addresses are **BIP-86 P2TR**.
  - The Alpen EOA uses path `m/44'/60'/0'/0/0` and is always the same address.
- ⚠️ **Never `reset` with a pending deposit.** The recovery DB is separate from the seed, so funds may become unrecoverable.
- The secret is stored in the OS keychain on macOS and Windows, and in a file under home on Linux.

Related: [[Fixed Denomination and FIFO UTXOs]], [[Alpen Doc Discrepancies]]
