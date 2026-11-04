---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/build-alpen-apps/deploying-alpen-smart-contracts
---

# Deploying Alpen smart contracts

In this walkthrough we will go through creating a very simple Counter contract, deploying it, and doing some simple contract calls.

## Deploying a counter contract

{% hint style="info" %}
This guide assumes that you have your local [git configured](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration).
{% endhint %}

1. Follow the [official steps](https://book.getfoundry.sh/getting-started/installation) to install Foundry.

#### 1. Follow the official steps to install Foundry.

Foundry is a modular toolkit for Ethereum development written in Rust. It includes:

* Forge: Build, test, and deploy smart contracts
* Cast: Interact with the chain from the command line
* Anvil: Local Ethereum node for development
* Chisel: Interactive Solidity REPL

In this guide we'll use Forge and Cast.

#### 2. Initialize your project

Use `init` to create a new smart contract project

```sh
forge init Counter
cd Counter
```

Forge will automatically add its standard counter at  `src/Counter.sol` with the code below

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Counter {
    uint256 public number;

    function setNumber(uint256 newNumber) public {
        number = newNumber;
    }

    function increment() public {
        number++;
    }
}
```

#### 3. Test and compile the contract

Test and build the Counter using Forge

```sh
forge test --evm-version "prague" ## runs the tests under Prague specs
forge build
```

Make sure to select Alpen's current EVM version.

#### 4. Setup the environment

Before we deploy the contract, we need to make sure we can connect to the Alpen public RPC.

Add the following line to a new `.env` file in your Counter project folder:

{% code overflow="wrap" %}

```sh
export ETH_RPC_URL=https://alpen.testnet.alpen.org ## by declaring it as ETH_RPC_URL we don't need to add --rpc-url to all subsequent forge and cast commands
```

{% endcode %}

Run `source .env` to load the parameters in the active shell session and query the endpoint to verify we're on the correct network, and call the RPC endpoint

```sh
curl -X POST $ETH_RPC_URL \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

The expected result is 20310 in hex format: `{"jsonrpc":"2.0","id":1,"result":"0x4f56"}`

{% hint style="info" %}
You can also run a full node! See the [Alpen full node operator guide](/running-an-alpen-node/alpen-full-node-operator-guide.md) to host your own Alpen RPC endpoints.
{% endhint %}

#### 5. Generate a *deployer* testnet wallet

{% hint style="info" %}
Foundry keeps previously imported wallets in the common directory at `~/.foundry/keystores`. If you already created and funded your *deployer* account, you can skip step 6.
{% endhint %}

From now on, you'll need a wallet with an active balance to deploy your contracts. Creating a dedicated *deployer* is a best practice to keep its private key separated from a standard *user* account.

Create a new mnemonic with

{% code overflow="wrap" %}

```sh
cast wallet new-mnemonic ## Backup or transcribe the Phrase and save the Address for later
```

{% endcode %}

You'll get an output similar to the following:

```
Generating mnemonic from provided entropy...
Successfully generated a new mnemonic.
Phrase:
pull pipe diagram aerobic horror child orange height blast true source tray

Accounts:
- Account 0:
Address:     0x2eAB7b2Cf9950621C05329Da3dE34c35B1c69108
Private key: 0x590fe557738fd3611527eb8303591aefd022a7842dc19f5209acd3cec493fee2
```

Backup or transcribe the 12-word `Phrase`, and save the `Address` for later.

Use  `import` command with your mnemonic passphrase to add *deployer* to the Foundry keystore

{% code overflow="wrap" %}

```sh
cast wallet import deployer --mnemonic "pull pipe diagram ..." ## Copy/paste the 12-word passphrase you generated in the previous step inside the double quotes
```

{% endcode %}

Setup a wallet password (or keep it blank) to proceed. See the official [Foundry cast documentation](https://getfoundry.sh/cast/reference/wallet#cast-wallet) for more information about cast.

#### 6. Add balance to your *deployer*

Send funds to the deployer address generated above to pay for Alpen fees.

Testnet III runs on the default bitcoin signet ("Signet Global Test Net VI"). Because signet BTC is not issued or controlled by Alpen Labs, we cannot provide large amounts of test liquidity on demand. Developers that need significant testnet liquidity should plan to source it independently.

The following options are available:

* **Use the Strata Bridge** [**through Alpen CLI**](/welcome/using-the-alpen-cli.md)**.** The Strata bridge supports fixed deposits of 2 sBTC and may take up to 15 hours to complete the lock-and-mint flow. This is the recommended path when an application needs to test with larger balances or simulate higher-liquidity flows.&#x20;
* **Request support from Alpen Labs.** Developers can complete the request form at <https://tally.so/r/aQKLZZ>. Alpen Labs may send a small amount of sBTC to the provided address, and a team member may follow up if additional coordination is required. This is the slowest option and is intended for later-stage testing or cases where the other liquidity paths are not sufficient.

When done, check your deployer balance with:

{% code overflow="wrap" %}

```bash
cast balance "$(cast wallet address --account deployer)" -e ## The -e parameter shows the balance in BTC
```

{% endcode %}

#### 7. Deploy the contract

(Optional) Test the Counter contract using Alpen testnet RPC

```sh
forge test --fork-url $ETH_RPC_URL
```

Perform a dry run of the contract deployment with `forge create`

{% code overflow="wrap" %}

```sh
forge create Counter --account deployer ## the --account parameter instructs forge to take the private key from the keystore instead of adding it in clear-text
```

{% endcode %}

{% hint style="info" %}
Always use forge with the `--account` parameter if you need to keep the private key confidential.&#x20;
{% endhint %}

In the result, check that `"chainId": "0x4f56"` is pointing at the right network. Issue the command `echo $((0x4f56))` to verify that the resulting chain ID number is `20310` :

{% code overflow="wrap" %}

```sh
Warning: Dry run enabled, not broadcasting transaction

Contract: Counter
Transaction: {
  "from": "0xef0a426c941722b9da46f6c54e78664a8ab154f8",
  "to": null,
  "maxFeePerGas": "0xf",
  "maxPriorityFeePerGas": "0x1",
  "gas": "0x26942",
  "input": "0x6080604052348015600e575f5ffd5b506101e18061001c5f395ff3fe608060405234801561000f575f5ffd5b506004361061003f575f3560e01c80633fb5c1cb146100435780638381f58a1461005f578063d09de08a1461007d575b5f5ffd5b61005d600480360381019061005891906100e4565b610087565b005b610067610090565b604051610074919061011e565b60405180910390f35b610085610095565b005b805f8190555050565b5f5481565b5f5f8154809291906100a690610164565b9190505550565b5f5ffd5b5f819050919050565b6100c3816100b1565b81146100cd575f5ffd5b50565b5f813590506100de816100ba565b92915050565b5f602082840312156100f9576100f86100ad565b5b5f610106848285016100d0565b91505092915050565b610118816100b1565b82525050565b5f6020820190506101315f83018461010f565b92915050565b7f4e487b71000000000000000000000000000000000000000000000000000000005f52601160045260245ffd5b5f61016e826100b1565b91507fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff82036101a05761019f610137565b5b60018201905091905056fea2646970667358221220df4a76c68e7f061f542d26155d08ef6ac6937da6830d06a5a5c3e20d2c78b21164736f6c634300081e0033",
  "nonce": "0x0",
  "chainId": "0x4f56"
}
ABI: [
  {
    "type": "function",
    "name": "increment",
    "inputs": [],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "number",
    "inputs": [],
    "outputs": [
      {
        "name": "",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "stateMutability": "view"
  },
  {
    "type": "function",
    "name": "setNumber",
    "inputs": [
      {
        "name": "newNumber",
        "type": "uint256",
        "internalType": "uint256"
      }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  }
]

Warning: To broadcast this transaction, add --broadcast to the previous command. See forge create --help for more.
```

{% endcode %}

Finally, broadcast the transaction with the command

{% code overflow="wrap" %}

```sh
forge create Counter --account deployer \
    --broadcast \
    --gas-price 2gwei \
    --priority-gas-price 1gwei
```

{% endcode %}

#### 8. Test the deployed contract

When the deployment is complete, Forge should return the following result. Note the `Deployed to` address, which will be our contract address

{% code overflow="wrap" %}

```sh
No files changed, compilation skipped
Enter keystore password:
Deployer: 0xef0a426C941722B9da46F6C54E78664a8ab154F8
Deployed to: 0x0aF93a258c394d32483664d2EB90d4565087f2c2
Transaction hash: 0xe5dff26f2f76df285f3d9d8a09296254049c9ed862567461c5191e8bf56f49f3
```

{% endcode %}

Append the contract address to the `.env` file to get a result similar to this one:

{% code overflow="wrap" %}

```sh
ETH_RPC_URL=https://alpen.testnet.alpen.org
COUNTER_CONTRACT_ADDRESS="0x7860cCB7..." ## copy/paste the value from Deployed to inside double quotes to avoid parsing errors 
```

{% endcode %}

And issue the command `source .env` to keep the smart contract address handy for the next steps.

Smoke-test the current state of our Counter with

```sh
cast call $COUNTER_CONTRACT_ADDRESS "number()(uint256)" ## It should return 0
```

**Congratulations! We have now successfully deployed the Counter contract on Alpen Testnet.**

### Interacting with the Counter using cast

In this section, you will interact with the Counter contract using a separate account that we'll call *user*.

#### 1. Create the *user* account and fund it

Follow the same steps used for the *deployer*

{% code overflow="wrap" %}

```sh
cast wallet nm ## Backup or transcribe the Phrase and save the Address for later

Generating mnemonic from provided entropy...
Successfully generated a new mnemonic.
Phrase:
urban shop knife please glide question canyon dust unusual initial lyrics taxi

Accounts:
- Account 0:
Address:     0xF4922E4C3193E58ADE54D2BC50A84F38C596E890
Private key: 0x4521e9dc4921233eb42d0a23b66a5d828b293d89f990f6b47bc93d1929cc77b6
```

{% endcode %}

And, similarly to the *deployer*, import the passphrase to the keystore:

{% code overflow="wrap" %}

```sh
cast wallet import user --mnemonic "urban shop knife ..." ## Copy/paste the 12-word passphrase inside the double quotes.
```

{% endcode %}

#### 2. Fund the user account

Similarly to the [deployer account instructions above](#id-6.-add-balance-to-your-deployer), source test tokens from the Bitcoin signet ecosystem, or use `cast` to receive funds from the deployer itself.

In this example we'll use our `deployer` to fund the `user` account:

```sh
cast balance "$(cast wallet address --account deployer)" -e
```

Then use the `send` command to transfer funds

{% code overflow="wrap" %}

```sh
cast wallet address --account user ## Returns the user wallet address
cast send 0xF4922E4C... --value 0.042ether --account deployer \
    --gas-price 2gwei --priority-gas-price 1gwei \
    ## Sends 0.042 sBTC from deployer account to the user address generated above.
```

{% endcode %}

{% hint style="info" %}
Note that we use `ether` to instruct `cast` that it has to send the chain's native currency (on Alpen's testnet it is `sBTC`)
{% endhint %}

#### 3. Interact with the Counter contract

First, view the current state of our Counter contract

```sh
cast call $COUNTER_CONTRACT_ADDRESS "number()(uint256)"
```

Then, increase it by one unit with the command

```sh
cast send $COUNTER_CONTRACT_ADDRESS "increment()" --account user \
    --gas-price 2gwei \
    --priority-gas-price 1gwei
```

You should get a full execution report similar to the one below, with the line `status 1(success)` confirming that our contract call was successful

{% code overflow="wrap" %}

```sh
blockHash            0x05b3529060c1b248afc5639fd64ffdb992a83f8981549c0fc8ae1948a7de2072
blockNumber          86882
contractAddress      
cumulativeGasUsed    26382
effectiveGasPrice    1000000007
from                 0x39b3A1ad2c9d87484A0C2B245Def64c278fe4C18
gasUsed              26382
logs                 []
logsBloom            0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root                 
status               1 (success)
transactionHash      0x60cd10679b12cb73a681166c88d7c9062c05bbadef8e00270e292e782048eb49
transactionIndex     0
type                 2
blobGasPrice         
blobGasUsed          
to                   0x0aF93a258c394d32483664d2EB90d4565087f2c2
```

{% endcode %}

Call again `number()` to see the new state of the Counter contract

```sh
cast call $COUNTER_CONTRACT_ADDRESS "number()(uint256)"
```

### Optional: verify the contract on Alpen's Blockscout explorer

Alpen's Blockscout explorer supports `forge verify-contract` to publish a contract source code and show a *Verified contract* icon next to its address.

Issue the following command to send a verification request

```sh
forge verify-contract \
  --guess-constructor-args \
  --verifier blockscout \
  --verifier-url 'https://explorer.testnet.alpen.org/api/' \
  --rpc-url 'https://explorer.testnet.alpen.org/api/eth-rpc' \
  $COUNTER_CONTRACT_ADDRESS \
  src/Counter.sol:Counter \
  --watch
```

Right after it finished, open Alpen explorer at the [verified contracts page](https://explorer.testnet.alpen.org/verified-contracts) to see our Counter listed at the top of the list.


---

