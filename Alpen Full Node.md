---
tags: [alpen, node, operations]
source: https://docs.alpen.org/running-an-alpen-node/alpen-full-node-operator-guide
---
# Alpen Full Node

An Alpen full node:
- follows the Alpen chain over **P2P** (Reth-based `alpen-client`),
- serves EVM and Alpen JSON-RPC,
- **reads Strata (OL) state from a published Strata RPC endpoint**.

It does **not** sequence, post DA to bitcoin, or prove.

**Setup (Testnet III):**
- Repo `alpenlabs/alpen-ee`, tag `v0.4.0-rc.3`, run from the `params/` dir.
- Docker image `public.ecr.aws/alpenlabs/alpen:4ef60bd`. You can also build it locally from `docker/alpen-client/Dockerfile`.
- `openssl rand -hex 32 > jwt.hex` creates the secret for Reth's authenticated Engine API.
- Config `fullnode-testnet.toml`:
  ```toml
  mode = "full_node"            # archival
  [ol]
  source = "rpc"
  client_url = "https://strata.testnet.alpen.org"
  [full_node]
  sequencer_pubkey = "a90b390260da71f87d2af52f612a9f4f77797bdfaa010343cc1a435047aa05d7"
  sequencer_http_url = "https://alpen.testnet.alpen.org"
  ```
- Run with `--disable-discovery --trusted-peers` set to the 3 bootnodes `bootnode-{1,2,3}.testnet.alpen.org:30303`.
- Ports: 9545 (HTTP RPC, mapped from 8545), 9546 (WS), 30303 tcp/udp (P2P), 8080 (local).
- Healthy logs show "Received gossip package", "Block added to canonical chain". `net_peerCount` should be `0x3`.

**Trust note:** with `[ol] source = "rpc"`, the node relies on Alpen's Strata RPC for OL state rather than deriving it from bitcoin itself. That makes this a weaker trust model than the architecture docs' "full nodes read checkpoints from bitcoin". See [[Centralization Points]].

Related: [[Alpen Full Node Hardware]], [[Alpen Troubleshooting]]
