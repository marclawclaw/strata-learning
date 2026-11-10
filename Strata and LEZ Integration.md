---
tags: [alpen, strata, lez, logos, integration, design]
created: 2026-11-04
source: logos-execution-zone origin/dev @ 4fba494e2 (2026-10-10)
---
# Strata and LEZ Integration

How BTC from the [[Strata Bridge]] could reach the **Logos Execution Zone (LEZ)**.

## The model: LEZ (via an adapter) becomes a SNARK account

**Strata doesn't push proofs out to anyone.** An integration is a [[SNARK Account]] on Strata. Its operator:

1. reads deposits from the account's Strata inbox,
2. proves the account's own state transition, in **SP1 → Groth16**,
3. submits a SNARK Account Update Tx to the [[Strata Sequencer]].

So a dedicated LEZ-side bridge component *is* that account's operator/prover. A tempting but wrong picture has Strata proofs flowing into a LEZ channel. What actually happens is:

- **Withdrawals:** LEZ evidence (burns) goes **into** an SP1 proof, and the proof goes **to Strata**.
- **Deposits:** Strata evidence (inbox deposits) goes **into LEZ**, so wrapped BTC can be minted.

LEZ keeps RISC Zero. Only the account's update proof has to be SP1. Inside it, the SP1 program can verify RISC0 receipts (a BN254 Groth16 verify) or LEZ sequencer signatures.

```
bitcoin ──DRT──▶ Strata (LEZ account inbox) ──read──▶ adapter ──mint order──▶ LEZ wrapped BTC
bitcoin ◀─payout─ bridge operators ◀─checkpoint─ Strata ◀─SP1 update─ adapter ◀─burn evidence─ LEZ
```

## What the account's SP1 program must prove

Account state is a **bridge ledger**: consumed deposit IDs, issued mint orders, processed burn IDs, and BTC outstanding on LEZ.

1. **Deposits consumed.** Each inbox message becomes a mint order (LEZ recipient `AccountId`, amount) committed in the account state.
2. **Withdrawals.** Each LEZ burn becomes an outgoing message to the bridge-gateway system account, with a bitcoin destination and a multiple of the [[Fixed Denomination and FIFO UTXOs|fixed denomination]].
3. **Conservation.** Withdrawals never exceed deposits.

Step 2 is the hard part: **which LEZ evidence can an SP1 program check?**

## What LEZ offers today (from code)

- **No state root.** `BlockHeader` is just `{block_id, prev_block_hash, hash, timestamp}` (`lez/common/src/block.rs:52`). There is nothing to Merkle-prove a LEZ balance or event against.
- **No chain-level validity proof.** Public transactions are re-executed, not proven. Only private transactions carry user-made RISC0 proofs. The indexer validates by re-execution.
- **Blocks are Bedrock channel inscriptions.** Each is signed by **one accredited sequencer key per posting turn**: a rotating, staked committee (`lez/programs/sequencer_stake/core/src/lib.rs:188`). Equivocation slashing needs committee approvals. Finality is Bedrock's: LEZ reads `FinalizedOp`s (`lez/sequencer/actors/bedrock/src/actor.rs:228`). There is no per-block multi-signature.
- **LEZ↔LEZ cross-zone trust model.** The target's watcher reads the peer's *finalized* blocks off Bedrock. It checks that an accredited key signed each block and recomputes the block hash. It enforces hash-chain linkage, a committee-size floor and per-source lifetime **mint caps**. In other words, it trusts the peer's sequencer plus Bedrock finality, and it limits the damage with caps.
  - `CrossZoneMessage.l1_inclusion_witness` is "reserved for a future source-state proof; MUST be None in v1" (`lez/programs/cross_zone_inbox/core/src/lib.rs:123`). That is the natural slot for Strata evidence.
  - The crate itself says a **non-LEZ peer needs a separate adapter** with its own trust model (`lez/cross_zone/src/lib.rs:8`). Strata would be the first one.
- **Reusable pieces:**
  - `wrapped_token` mints only via the cross-zone inbox and supports per-source caps (`lez/programs/wrapped_token/core/src/lib.rs:38`).
  - The native `bridge` program's deposit-receipt PDA, keyed by L1 op ID, is a ready replay-guard pattern.
- **Missing: the burn path.** `wrapped_token` has no Burn instruction. The native bridge's `Withdraw` panics: "Withdraws are disabled in the current version of LEZ" (`lez/programs/bridge/src/main.rs:45`).

## Options for proving LEZ burns to Strata

| | What SP1 verifies | Trust | Cost / work |
|---|---|---|---|
| **A. Sequencer attestation** | Ed25519 signatures on the LEZ blocks that contain burns, the hash chain from a known tip, signer ∈ accredited set (account state tracks committee changes), and the decoded burn tx | **Any single accredited LEZ sequencer can forge a burn** and drain the account's BTC. Bedrock finality isn't proven. | Cheap. Buildable now. |
| A+. Hardened attestation | Same, plus **k-of-n committee co-signatures** over burn batches, caps, and a delay that lets indexers object | k-of-n committee | New LEZ work: committee signing |
| **B. Validity** | A RISC0 **LEZ block-validity proof** (new), wrapped in SP1 by verifying its Groth16 receipt, plus Bedrock inclusion and finality | Cryptographic, apart from Bedrock consensus | A large new LEZ prover. Needs a state root in the header. |
| C. Partial validity | Re-execute only the transactions that touch wrapped BTC | Breaks down | Chained calls from DeFi programs move BTC, so the slice can't be isolated without scanning everything. Not viable. |

The realistic path is **A or A+ for v1, then B**. A comes with an explicit cap on the account's BTC. B is the end state where the SP1 program verifies LEZ proofs, and it needs LEZ to commit to state in its headers.

## The deposit direction (Strata → LEZ)

- LEZ must learn that a deposit is real before minting.
  - **v1:** trust the adapter. It posts mint orders to its own Bedrock inscription channel, which LEZ treats as a cross-zone peer through the new non-LEZ adapter, with mint caps.
  - **Later:** a LEZ public program verifies the Strata checkpoint proof (SP1 Groth16; verification is just execution in public mode) plus inclusion of the mint order. Knowing the checkpoint is *buried on bitcoin* also needs a bitcoin header light client.
- Minting is safe only after Strata finality ([[Alpen Finality Levels]]): deposit 12 confs → Strata block (~60 min) → checkpoint (~10 h) → maturity. Minting earlier means taking Strata reorg risk.

## Design consequences

- **Data availability for the bridge committee:** largely moot. Strata bridge operators and watchtowers act on **Strata checkpoints only**, and they verify the account proof, not LEZ data. LEZ data is needed by the **prover**, and by anyone who wants to take over proving. Open question: does Strata require every account to post DA to bitcoin, the way Alpen's proof binds it ([[SNARK Account Update Proof]])?
- **Decentralised sequencer set:** irrelevant to Strata. The account has one prover. LEZ's committee matters only as the *source of attestations* (option A).
- **Shielded BTC:** mint to a public account, then shield. To exit, the user unshields and makes a public burn. A private burn would need its RISC0 proof verified inside SP1, and the destination and amount would still have to be public.
- **Denomination:** exits are 2 BTC multiples (testnet). The adapter can't merge several users into one payout, so small holders need a LEZ-side LP or market maker ([[Fixed Denomination and FIFO UTXOs]]).
- **Cadence and fees:** each account update costs an SP1 Groth16 proof plus a Strata transaction. Withdrawal latency is roughly Bedrock finality + the next update + Strata block + checkpoint + 12 blocks, so hours to over 10 h.

## Open questions

- [ ] Does Strata enforce a **per-account BTC balance**, so that a compromised LEZ account can lose only its own BTC and not Alpen's? (The notes in [[SNARK Account]] say accounts hold their own balance. Not checked in code.)
- [ ] Is account-update submission permissioned? Who controls the LEZ account's VK ([[Verification Key Upgradeability]])?
- [ ] Does Strata require bitcoin DA from non-Alpen accounts?
- [ ] DRT format: can the `OP_RETURN` carry a 32-byte LEZ `AccountId` for a non-Alpen account?
- [ ] How does an external chain prove "deposit X is in account Y's inbox / state" from a checkpoint (the public inputs of the checkpoint proof)?
- [ ] LEZ side: will LEZ add a **state root** to `BlockHeader`, committee multi-sig attestations, or a block-validity prover? This decides A vs B.
- [ ] Is Strata pinned to a specific SP1 version? Is verifying RISC0 Groth16 inside SP1 within their cycle budget?

Deep dive on the burn-evidence problem: [[LEZ Burn Verifiability]]

Related: [[Execution and Orchestration Separation]] (Alpen explicitly anticipates other VMs as further SNARK accounts), [[Centralization Points]], [[Alpen Open Questions]]

> [!note] Update 2026-11
> LEZ cross-zone messages will be 2/3-committee-signed on a per-zone outbound channel ([[LEZ Signed Outbound Channel]]). That gives burns k-of-n evidence that a Strata SP1 program can verify (option A+ above, built into the platform).
