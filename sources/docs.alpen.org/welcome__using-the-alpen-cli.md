---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/welcome/using-the-alpen-cli
---

# Using the Alpen CLI

The Alpen CLI is a Rust command line interface that allows you to interact with Alpen, the [bitcoin bridge](/how-alpen-works/bitcoin-bridge.md), and an associated signet bitcoin network.

This guide assumes that you are doing a fresh install of Alpen CLI. If you are upgrading from 0.2.0, follow the [#cleaning-up](#cleaning-up "mention") instructions in the section below.

### Installation <a href="#user-content-installation" id="user-content-installation"></a>

{% hint style="info" %}
If you are running Linux, you need a C/C++ compiler along with `libssl` and development libraries and tools required for compiling software.\
\
In Debian/Ubuntu flavored distributions you can install them with:\
\
`sudo apt install build-essential libssl-dev pkg-config`
{% endhint %}

Ensure you have Cargo installed. If you don't have it, we recommend you install it using [Rustup](https://rustup.rs/). Then run:

{% code overflow="wrap" %}

```
cargo +nightly install --git https://github.com/alpenlabs/alpen alpen-cli --rev 5643210dc7648fcf3820d13e29d9a3fb63d50d58 --locked --force
```

{% endcode %}

This will compile and install the binary into your system path. It may take a few minutes depending on how powerful your hardware is.

### Configuration <a href="#user-content-installation" id="user-content-installation"></a>

The CLI uses a TOML file for configuration. Changing the parameters is for *advanced users only*.

To find your config file location, run `alpen config`. You can then open this file in an editor. You may only change the keys listed in the file if you want to use alternatives to the defaults provided.

```
~/.config/alpen/config.toml # Linux
~/Library/Application\ Support/io.alpenlabs.alpen/config.toml # MacOS
C:\Users\<USER>\AppData\Roaming\alpen\config.toml # Windows
```

Use these parameters to run a CLI on Testnet III:

{% code overflow="wrap" %}

```
## Alpen Testnet III 0.3 CLI configuration params
esplora = "https://blockstream.info/signet/api/"
alpen_endpoint = "https://alpen.testnet.alpen.org"
mempool_endpoint = "https://mempool.space/signet"
blockscout_endpoint = "https://explorer.testnet.alpen.org"

bridge_pubkey = "1e070c9190e2dcc8671957722aec6f1533f47ce3bf2fcb53cda2961faf309049"
bridge_alpen_address = "0x5400000000000000000000000000000000000001"
bridge_fee_sats = 1000
finality_depth = 0

network = "signet"
magic_bytes = "ALPN"
bridge_denomination_sats = 200000000
recovery_delay = 36
max_withdrawal_descriptor_len = 81

```

{% endcode %}

Check in the [#user-content-configure-the-cli](#user-content-configure-the-cli "mention") section what these parameters are for.

### How to use <a href="#user-content-how-to-use" id="user-content-how-to-use"></a>

The CLI is your entrypoint for interacting with Alpen.

Run `alpen --help` to display the help message.

```
$ alpen --help
Usage: alpen <command> [<args>]

A CLI for interacting with Alpen and the underlying bitcoin (signet) network

Options:
  --help, help      display usage information

Commands:
  recover           Attempts a recovery of old deposit transactions
  drain             Drains the internal wallet to the provided signet or Alpen
                    address
  balance           Prints the wallet's current balance(s)
  backup            Prints a BIP39 mnemonic encoding the internal wallet's seed
                    bytes
  deposit           Deposits BTC from signet into Alpen
  withdraw          Withdraws BTC from Alpen to signet
  send              Sends BTC from the internal wallet
  receive           Prints a new address for the internal wallet
  change-password   Changes the seed's encryption password
  reset             DANGER: resets the CLI completely, destroying all keys and
                    databases. Keeps config.
  scan              Performs a full scan of the signet wallet
  config            Prints the location of the CLI's TOML config file
  debug             Various debug utilities

```

The CLI is composed of multiple commands. You can append `--help` to any command to get more details about what the command does and what arguments it supports.

These commands control a bitcoin signet wallet and an EVM-compatible Alpen wallet, allowing you to manage Alpen funds without any other tools. For example, the CLI can help you deposit into and withdraw BTC from Alpen via the `deposit` and `withdraw` commands, respectively.

The first time you make a command in the CLI, you will be asked to create a new password to encrypt the wallet's secret. If you're using Windows or macOS, `alpen` will store the encrypted secret in the operating system's keychain. On Linux, it is stored in a file in the user's home directory.

#### Backup your wallet <a href="#user-content-backup-your-wallet" id="user-content-backup-your-wallet"></a>

Before you start, you should make sure to backup your wallet.

```
Usage: alpen backup [--language <language>]

Prints a BIP39 mnemonic encoding the internal wallet's seed bytes

Options:
  --language        select a language for the BIP39 mnemonic. defaults to
                    English. options: en, cn, cn-trad, cz, fr, it, jp, kr or es
  --help, help      display usage information
```

Keep this backup safe; your funds will be lost if you lose it.

{% hint style="warning" %}
Never reset the CLI when an `alpen deposit` transaction hasn't been either received on Alpen or successfully recovered, as the database storing recovery information is separate to your seed, and funds may be impossible to recover.
{% endhint %}

When you restore your wallet, make sure to run the command [`alpen scan`](/welcome/using-the-alpen-cli.md#user-content-scan-your-wallet) so the CLI can find all your previous funds. This process can take some time, depending on how many UTXOs you have.

#### Add BTC to your CLI wallet <a href="#user-content-request-btc" id="user-content-request-btc"></a>

Alpen Testnet is using bridged liquidity from Bitcoin's default public signet (Signet Global Test Net VI). Signet maintainers offer a *Powcoins Faucet* utility on GitHub at [https://github.com/ajtowns/powcoins/](https://github.com/ajtowns/powcoins/tree/master). You'll need at least 2 BTC plus a few thousand sats for mining fees in the CLI signet wallet to request a Strata bridge deposit with the `alpen deposit` command below, and wait up to 15 hours for the transaction to be visible on Alpen.

Alternatively, Alpen Labs can provide a few test coins, already bridged into Alpen, if you submit the request form at <https://tally.so/r/aQKLZZ>.

#### Check your balance <a href="#user-content-check-your-balance" id="user-content-check-your-balance"></a>

```
Usage: alpen balance <network_type>

Prints the wallet's current balance(s)

Positional Arguments:
  network_type      either "signet" or "alpen"

Options:
  --help, help      display usage information
```

#### Deposit BTC into Alpen <a href="#user-content-deposit-btc-into-alpen" id="user-content-deposit-btc-into-alpen"></a>

If you have at least 2 BTC in your CLI signet wallet, plus a few thousand extra sats for mining fees for the deposit request and deposit transactions, then you can make a deposit into Alpen.

{% code overflow="wrap" %}

```
Usage: alpen deposit [--fee-rate <fee-rate>] [--] [<alpen_address>]

Deposits BTC from signet into Alpen

Positional Arguments:
  alpen_address     the Alpen address to deposit the funds into. defaults to the
                    wallet's internal address.

Options:
  --fee-rate        override signet fee rate in sat/vbyte; the effective rate is
                    at least 1
  --help, help      display usage information
```

{% endcode %}

To deposit 2 BTC from the internal signet wallet into the internal Alpen wallet, run this command.

```
alpen deposit
```

If the funds are not yet available in your CLI wallet on Alpen after 15 hours (check using `alpen balance alpen`) then use the [#user-content-recover](#user-content-recover "mention") command to scan the network and perform an autorecovery.

#### Withdraw BTC to signet <a href="#user-content-withdraw-btc-back-to-signet" id="user-content-withdraw-btc-back-to-signet"></a>

Use this command to withdraw/bridge BTC from your Alpen wallet to the provided signet address. If an address isn't provided, the CLI will generate a new unused signet address from its own wallet. The withdrawal amount must be 2 BTC or exact multiples.

{% code overflow="wrap" %}

```
Usage: alpen withdraw [--amount <amount>] [--operator <operator>] [--] [<address>]

Withdraws BTC from Alpen to signet

Positional Arguments:
  address           the signet address to send funds to. defaults to a new
                    internal wallet address

Options:
  --amount          amount to withdraw in sats (must be a positive multiple of
                    the denomination). defaults to one denomination unit.
  --operator        selected operator index for withdrawal assignment
  --help, help      display usage information

```

{% endcode %}

#### Send BTC <a href="#user-content-send-btc" id="user-content-send-btc"></a>

You can send BTC from the internal signet or Alpen wallet via the `alpen send` command. On signet, you can use the `fee-rate` option to override the automatically determined fee rate.

```
Usage: alpen send <network_type> <amount> <address> [--fee-rate <fee-rate>]

Sends BTC from the internal wallet.

Positional Arguments:
  network_type      either "signet" or "alpen"
  amount            amount to send in sats
  address           address to send to

Options:
  --fee-rate        override signet fee rate in sat/vbyte. must be >=1
  --help, help      display usage information
```

{% hint style="info" %}
Use instead the [drain command](#user-content-drain-the-wallet) if you want to send all funds to a different wallet.
{% endhint %}

#### **Receive BTC**

`alpen receive signet` generates a new address for receiving BTC on signet network. This is the first step to bridge funds to Alpen. The signet wallet will print a different address every time, which is a [BIP-86](https://github.com/bitcoin/bips/blob/master/bip-0086.mediawiki) P2TR address.

On Alpen network, the command `alpen receive alpen` creates an EVM-compatible externally owned account (EOA) with the following BIP44 derivation path: `m/44'/60'/0'/0/0`. Once generated, the CLI will always return the same Alpen EOA address.

```
Usage: alpen receive <network_type>

Prints a new address for the internal wallet

Positional Arguments:
  network_type      either "signet" or "alpen"

Options:
  --help, help      display usage information
```

#### Change wallet password <a href="#user-content-change-wallet-password" id="user-content-change-wallet-password"></a>

The `change-password` command will load your existing seed, decrypt it, and then prompt you for a new password, encrypt the seed with the new password, and overwrite the old encrypted seed.

```
Usage: alpen change-password

Changes the seed's encryption password

Options:
  --help, help      display usage information
```

#### Drain the wallet <a href="#user-content-drain-the-wallet" id="user-content-drain-the-wallet"></a>

Run the `alpen drain` command if you want to send all of your funds to a new single address.

Signet funds can only be drained to another signet address, and Alpen funds can only be drained to another Alpen address. This command will not implicitly deposit/withdraw funds between the two wallets.

```
Usage: alpen drain [-s <signet-address>] [-r <alpen-address>] [--fee-rate <fee-rate>]

Drains the internal wallet to the provided signet and Alpen addresses

Options:
  -s, --signet-address
                    a signet address for signet funds to be drained to
  -r, --alpen-address
                    an Alpen address for Alpen funds to be drained to
  --fee-rate        override signet fee rate in sat/vbyte. must be >=1
  --help, help      display usage information
```

#### Recover <a href="#user-content-recover" id="user-content-recover"></a>

To recover any failed/expired deposit transactions, use the `recover` command.

Recovery of failed deposit transactions can be made 36 signet blocks after the deposit transaction was confirmed. With an expected 10 minute block time, that's about 6 hours.

Once recovery is complete, run `alpen scan` to make the funds available in your spendable balance.

```
Usage: alpen recover [--fee-rate <fee-rate>]

Attempts a recovery of old deposit transactions

Options:
  --fee-rate        override signet fee rate in sat/vbyte. must be >=1
  --help, help      display usage information
```

#### Scan your wallet <a href="#user-content-scan-your-wallet" id="user-content-scan-your-wallet"></a>

The `scan` command scans the signet for coins belonging to the internal wallet. This should be used after restoring Alpen CLI from a BIP-39 mnemonic.

```
Usage: alpen scan

Performs a full scan of the signet wallet

Options:
  --help, help      display usage information
```

### Cleaning Up

If you need to completely remove Alpen CLI, perform the following tasks:

1. Do an [`alpen backup`](#user-content-backup-your-wallet) to preserve your seed passphrase.
2. Run `alpen reset` to remove all local data.
3. Find the configuration files as indicated by `alpen config` and remove the `config.toml` file.
4. Run the command `cargo uninstall alpen-cli` to remove the binary.
5. If running a shell, reload or exit your session to clean up the command path and temporary env variables.

Once done, you can re-install the CLI without stale/outdated parameters.


---

