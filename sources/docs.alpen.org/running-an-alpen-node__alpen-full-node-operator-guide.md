---
tags: [source, raw]
fetched: 2026-11-04
url: https://docs.alpen.org/running-an-alpen-node/alpen-full-node-operator-guide
---

# Running an Alpen Full Node

An Alpen full node follows the Alpen rollup chain over P2P, serves EVM-compatible and Alpen-specific JSON-RPC methods, and reads Strata orchestration layer (OL) state from a published Strata RPC endpoint. It does not sequence blocks, post data to Bitcoin Signet, or generate proofs.

### Network Topology

The operator stack runs one local service:

| Component       | Purpose                                                                                                                                |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Alpen full node | Syncs execution blocks over Alpen P2P, reads OL state from Strata RPC, and exposes EVM-compatible and Alpen-specific JSON-RPC methods. |

The full node connects to:

| Endpoint        | Purpose                                                              |
| --------------- | -------------------------------------------------------------------- |
| Strata RPC      | Provides OL chain status used by the Alpen full node's OL tracker.   |
| Alpen P2P peers | Provide execution blocks over the Alpen execution-layer P2P network. |

### Requirements

Install on the host:

* Docker Engine 29.x or newer.
* Git.

The host must be able to reach:

* The published Strata RPC endpoint over HTTPS.
* A published Alpen P2P endpoint over TCP and UDP.

The Alpen endpoint is the execution-layer peer used by the full node. It should be a published Alpen P2P endpoint run with `alpen-client`, which is Alpen's Reth-based execution client.

### Clone the Repository

```bash
git clone -b v0.4.0-rc.3 https://github.com/alpenlabs/alpen-ee.git
cd alpen-ee/params
```

### Create the JWT Secret

{% hint style="info" %}
All commands in the rest of this guide run from `alpen-ee/params`.
{% endhint %}

Create the JWT secret file mounted by the Alpen full node:

```bash
openssl rand -hex 32 > jwt.hex
chmod 644 jwt.hex
```

Alpen container file will mount this file at `/app/keys/jwt.hex` for Reth's authenticated Engine API.

### Configure the Node

Generate a `fullnode-testnet.toml` configuration file:

```bash
touch fullnode-testnet.toml
```

Edit the `.toml` file with the values published for the target Alpen network:

```bash
# Image: public.ecr.aws/alpenlabs/alpen:4ef60bd
mode = "full_node"

[ol]
source = "rpc"
client_url = "https://strata.testnet.alpen.org"

[full_node]
sequencer_pubkey = "a90b390260da71f87d2af52f612a9f4f77797bdfaa010343cc1a435047aa05d7"
sequencer_http_url = "https://alpen.testnet.alpen.org"

```

`mode = "full_node"` deploys an Alpen archival full node.

### Run the Full Node

{% hint style="info" %}
You can follow the [#build-the-image-locally](#build-the-image-locally "mention") section below if you prefer not to rely on the precompiled image.
{% endhint %}

Start the Alpen full node with:

{% code overflow="wrap" %}

```bash
docker run -d --name alpen-fullnode \
  --mount "type=bind,src=$PWD/fullnode-testnet.toml,dst=/app/configs/alpen-config.toml,readonly" \
  --mount "type=bind,src=$PWD/testnet.json,dst=/app/configs/generated/alpen-params.json,readonly" \
  --mount "type=bind,src=$PWD/jwt.hex,dst=/app/keys/jwt.hex,readonly" \
  -v alpen-fullnode-data:/app/data \
  -p 127.0.0.1:9545:8545 \
  -p 127.0.0.1:9546:8546 \
  -p 127.0.0.1:8080:8080 \
  -p 30303:30303/tcp \
  -p 30303:30303/udp \
  public.ecr.aws/alpenlabs/alpen:4ef60bd \
  --port 30303 \
  --disable-discovery \
  --trusted-peers "enode://7148268aa1386af4b3a987d21ce7fab49254761b7c5f1859a3791f300090163c92caa61620c488d56e187f4ce31924eda678b4a7f683772954ec5977d1b7277c@bootnode-1.testnet.alpen.org:30303,enode://46dc0b4972b32e644f9289ed55be7dc114773ef202e028224e44b672eaf9570bbc7b585423f9e8061c0d8e1867ac8fa59b401eadbcc0bfc2b8adeabe0cdc4a62@bootnode-2.testnet.alpen.org:30303,enode://b44074ce9f9e9dbaec6980b75c9c9d8633a6f30b7116c7923f410ae19304f4ba16955bf74b3b9740eebfaf74e97ba2b97ed8f1354e6113375ae44d41fc072d58@bootnode-3.testnet.alpen.org:30303"
```

{% endcode %}

Follow the logs to check it's running normally:

```bash
docker logs -f alpen-fullnode
```

Healthy sync logs include:

```
Received gossip package
Block added to canonical chain
Canonical chain committed
```

### Verify

Check that the service is running and healthy:

```bash
docker ps
```

Check peer count:

```bash
curl -s -X POST http://127.0.0.1:9545 \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"net_peerCount","params":[]}'
```

The expected result is `{"jsonrpc":"2.0","id":1,"result":"0x3"}` . `0x0` means the full node has no peers and won't be able sync.

Check if the node is still downloading blocks:

```bash
curl -s -X POST http://127.0.0.1:9545 \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_syncing","params":[]}'
```

The expected result is `{"jsonrpc":"2.0","id":1,"result":false}` .

Check Alpen block height:

```bash
curl -s -X POST http://127.0.0.1:9545 \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

Check the upstream Strata RPC status used in `client_url`:

```bash
curl -s -X POST https://strata.testnet.alpen.org \
  -H 'content-type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"strata_getChainStatus","params":[]}'
```

The response must contain a `result` object with `tip`, `confirmed`, `finalized`, and `latest` fields.

### Ports

The container file exposes:

| Host port | Container port | Purpose                                              |
| --------- | -------------- | ---------------------------------------------------- |
| `9545`    | `8545`         | EVM-compatible and Alpen-specific HTTP JSON-RPC      |
| `9546`    | `8546`         | EVM-compatible and Alpen-specific WebSocket JSON-RPC |
| `30303`   | `30303/tcp`    | Alpen P2P TCP                                        |
| `30303`   | `30303/udp`    | Alpen P2P discovery                                  |

Only expose HTTP or WebSocket RPC publicly if you intend to operate a public RPC endpoint. Keep administrative and auth-style endpoints private.

### Troubleshooting

#### Peer count stays zero

* Check that `--trusted-peers` parameter is using the known upstream Alpen peers above.
* Check that the peer is reachable over TCP.
* Check firewall rules for TCP and UDP on `30303`.
* Confirm that outbound TCP and UDP traffic to the Alpen P2P endpoint is allowed by the host and network firewall.

#### Block height does not increase

* Check `net_peerCount`.
* Check logs for `Received gossip package`.
* Compare `eth_blockNumber` with a known upstream RPC e.g. `https://alpen.testnet.alpen.org`.

#### OL tracker errors appear

* Check that `client_url` in the config file points to the same network as the Alpen peers.
* Check that `sequencer_pubkey` in the config file match the target network.
* Query `strata_getChainStatus` on the `client_url` endpoint to confirm it is returning a valid chain status.

### Stop the Node

Stop the service:

```bash
docker stop alpen-fullnode
```

Remove local data:

```bash
docker rm alpen-fullnode
docker volume rm alpen-fullnode-data
```

{% hint style="warning" %}
Do not remove volumes unless you intentionally want to resync your Alpen node from scratch.
{% endhint %}

### Build the Image locally

To build locally your Alpen node, launch the `docker build` command from the `alpen-ee` folder:

```bash
docker build \
  -f docker/alpen-client/Dockerfile \
  -t alpen-client:local \
  .
```

Once done, replace `public.ecr.aws/alpenlabs/alpen:4ef60bd` with `alpen-client:local` in the launch command:

{% code overflow="wrap" %}

```bash
docker run -d --name alpen-fullnode \
  --mount "type=bind,src=$PWD/fullnode-testnet.toml,dst=/app/configs/alpen-config.toml,readonly" \
  --mount "type=bind,src=$PWD/testnet.json,dst=/app/configs/generated/alpen-params.json,readonly" \
  --mount "type=bind,src=$PWD/jwt.hex,dst=/app/keys/jwt.hex,readonly" \
  -v alpen-fullnode-data:/app/data \
  -p 127.0.0.1:9545:8545 \
  -p 127.0.0.1:9546:8546 \
  -p 127.0.0.1:8080:8080 \
  -p 30303:30303/tcp \
  -p 30303:30303/udp \
  alpen-client:local \
  --port 30303 \
  --disable-discovery \
  --trusted-peers "enode://7148268aa1386af4b3a987d21ce7fab49254761b7c5f1859a3791f300090163c92caa61620c488d56e187f4ce31924eda678b4a7f683772954ec5977d1b7277c@bootnode-1.testnet.alpen.org:30303,enode://46dc0b4972b32e644f9289ed55be7dc114773ef202e028224e44b672eaf9570bbc7b585423f9e8061c0d8e1867ac8fa59b401eadbcc0bfc2b8adeabe0cdc4a62@bootnode-2.testnet.alpen.org:30303,enode://b44074ce9f9e9dbaec6980b75c9c9d8633a6f30b7116c7923f410ae19304f4ba16955bf74b3b9740eebfaf74e97ba2b97ed8f1354e6113375ae44d41fc072d58@bootnode-3.testnet.alpen.org:30303"
```

{% endcode %}


---

