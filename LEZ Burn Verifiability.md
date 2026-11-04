---
tags: [lez, logos, bridge, trust-model, deep-dive]
created: 2026-11-04
source: logos-execution-zone origin/dev @ 4fba494e2; logos-blockchain @ 3926a53 (LEZ's pinned rev); logos-co/roadmap (2026-10-27); logos-lips; logos-docs-main; rfp
---
# LEZ Burn Verifiability

**Question:** can anything outside LEZ (the Strata SP1 program, an Alpen or Ethereum contract) verify that a burn really happened on LEZ? Is that planned to change?

**Short answer:** today the only evidence is **sequencer signatures**. That is also exactly what Bedrock itself relies on for Logos' own zone→L1 withdrawals. A validity-proven zone ("Native Zone") is on the roadmap only as **post-mainnet, undated, with nothing checked off**. Nothing plans a LEZ state root or a LEZ light client.

## What the code shows

**LEZ (origin/dev):**
- `BlockHeader` = `{block_id, prev_block_hash, hash, timestamp}` (`lez/common/src/block.rs:52`). There is **no state root**. `hash` is SHA-256 over the block. The cross-zone code warns that the sequencer's signature doesn't even cover the `header.hash` field.
- The only Merkle tree is the **private commitment set** (`lee/state_machine/src/state/mod.rs`). Its root is internal to private-tx membership proofs and is not committed in the header. Public account state has no commitment.
- History: early prototypes (2024–25) had sparse Merkle trees for UTXO commitments. In May 2025 these were replaced with sets (`cb22a95df` "smt/imt replaced with sets"). No public-state root has ever existed.
- There is no block prover and no light-client code. Branches only touch private-state provability (`Pravdyvy/private-state-provability`, May 2025, unmerged).
- **Invalid blocks.** If a sequencer posts a block that doesn't apply, the indexer retries 3 times and then **parks**: it stalls and records it (`lez/indexer/core/src/lib.rs:28,502`). There is no fraud proof and nothing happens on-chain. Honest LEZ nodes won't accept a forged burn, but an external verifier that checks only signatures would.
- **Slashing.** `sequencer_stake` slashes by committee approvals, and "the reason for the offence is not checked". Equivocation is the case that's built (2/3 threshold per roadmap updates). Punishing an invalid-state block would be a social, off-chain call.
- Burn and withdraw paths are absent. The native `bridge` program's `Withdraw` panics, and `wrapped_token` has no burn ([[Strata and LEZ Integration]]).

**Bedrock (logos-blockchain @ 3926a53, LEZ's pinned rev):**
- Each channel has `transfer_threshold`: "how many keys are required to transfer or withdraw funds from the channel" (`core/src/mantle/channel.rs:132`).
- `ChannelTransfer` validation checks that inputs belong to the channel, that value is balanced, and that **exactly `transfer_threshold` Ed25519 signatures from accredited keys** are present (`core/src/mantle/ops/channel/channel_transfer.rs:166`). It does **not** check LEZ state.
- LEZ sets `DEFAULT_SEQUENCER_WITHDRAW_THRESHOLD = 1` (`lez/system_accounts/src/lib.rs:20`). So **one sequencer can authorize a native withdrawal today**. The m-of-n primitive exists (merged as #3495, per the 2026-09/10 updates), but LEZ doesn't use it yet.

## What the docs and specs say

- logos-docs FAQ (`docs/blockchain/faq-logos-blockchain.md:41`): Bedrock "stores whatever state updates a Zone's sequencer inscribes without verifying their validity… enforces only one aggregate invariant: total withdrawn ≤ total deposited."
- Mantle spec (`logos-lips/.../bedrock-v1.1-mantle-specification.md:383`, 2026-10-04): "**a deposit is a transfer of custody**… its sequencers can… release it to whoever they choose with `CHANNEL_WITHDRAW`… no on-chain recourse."
- The roadmap (`lez_bridging.md`, `blockchain_bridging.md`, v0.2, done) lists "threshold-signed sequencer withdrawals" and names the risk: "must handle malicious sequencer scenarios".
- **Doc conflict:** `about-zones.md:59` says LEZ state is "verified via zero knowledge proofs". The FAQ, the LEZ intro and the code all say public execution is **re-executed**, not proven.

## Roadmap

| Item | Status | Relevance |
|---|---|---|
| **Native Zone**: "ZK proven zones where the STF is enforced by the base-layer" (`roadmap/native_zone.md`) | **Post-mainnet**, no date, spec and impl unchecked, 3 researchers | The only path to proven LEZ state |
| Proving-time benchmarks for the sovereign-zone ledger (2026-05-25); risc0 rollup PoC at 0.8 TPS on an RTX 4090 (2025-11-03) | Research | Shows proving the STF is far from production throughput |
| Holder-authorized channel notes RFC (logos-lips#457); programs governing channel notes RFC (#481) | In review | Would remove the sequencers' unilateral custody of *Bedrock* funds. It doesn't make LEZ state provable to outsiders. |
| PoS for bridged funds: "authorization rules preventing a malicious sequencer set…" | v0.3, unchecked | Same: limits sequencer custody |
| Permissionless sequencing (per-channel lottery, Proposal VIII) | Research | Changes who signs, not what a signature proves |
| Bedrock `epoch_state_root` (checkpoint fast-bootstrap RFC) | RFC | A state root of **Bedrock**, not LEZ |
| A LEZ / Bedrock light client | **Not on the roadmap** | — |
| Mainnet | No date in the public roadmap | Mainnet "required" items: bridging, decentralized sequencing. No proofs. |

RFP-022 (`rfp/RFPs/RFP-022-wrapped-erc20.md:427,748`) requires "cryptographic verification of a valid LEZ burn, verified natively on Ethereum" and leaves the proof system to the proposer. Nothing in the protocol currently gives a proposer anything to verify against beyond sequencer signatures. The same gap applies to any Strata integration.

## Implications for a Strata integration

- **The strongest evidence available is what Bedrock itself trusts:** an m-of-n accredited-sequencer Ed25519 signature over the burn/withdraw batch. The SP1 program can verify this cheaply. LEZ has to raise the threshold above 1 and have sequencers sign burn batches (option A+ in [[Strata and LEZ Integration]]).
- That puts wrapped BTC on **the same footing as LEZ's own native bridge**. Honest framing: "trust a threshold of the LEZ sequencer committee, plus caps". That's weaker than Strata's 1-of-N bridge, and the weakest link sets the overall security.
- A validity-proven path depends on Native Zone, which is post-mainnet and undated. Proving the STF also needs a state root in the header first.
- An intermediate idea that would need LEZ-team buy-in: a **state root in `BlockHeader`**, signed by the committee. With it, burns become Merkle-provable against a signed root, and later the same root becomes the public output of a validity proof.

Related: [[Strata and LEZ Integration]], [[1-of-N Trust Model]], [[Centralization Points]]
