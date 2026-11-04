---
tags: [strata, bridge, proofs, sp1, glock, mosaic, lez]
created: 2026-11-04
source: alpenlabs/asm abf2bd5, moho 9c18376, strata-bridge 31fd75bd, g16 f9697ac, mosaic 349c1da, alpen bc989490, strata-common a8b11a9
---
# Strata Bridge Proof Chain

How a [[Strata Bridge]] withdrawal claim is secured, from bitcoin blocks up to the garbled circuit. Read from source. **(V)** = verified in code, **(I)** = inference. Paths are relative to each repo.

```
bitcoin block ─► ASM STF (SP1 step proof, 1 per block)
                   │ admin / checkpoint / bridge subprotocols
                   ▼
              MohoState {inner = H(AnchorState), next_predicate, export_state}
                   │ Moho recursive proof (1 per block, chains from genesis)
                   ▼
 operator:    BRIDGE PROOF (SP1→Groth16, posted in OP_RETURN of Bridge Proof tx)
              "claim (deposit_idx, my_pubkey) is leaf #i of the bridge-v1 MMR
               on a chain from genesis with PoW = P"
                   │
 watchtower:  COUNTERPROOF (SP1→Groth16, blake3, 36-byte public values)
              "that bridge proof is invalid / malformed, OR I have a heavier chain
               where leaf #i is absent or different"
                   │ 128-byte proof fed as labels through adaptor signatures
                   ▼
 operator:    evaluates Mosaic garbled g16 circuit (Groth16 verifier of the
              COUNTERPROOF vkey). Invalid counterproof ⇒ fault secret ⇒ NACK.
```

Key correction: the garbled circuit verifies the **watchtower's counterproof**, not the operator's bridge proof. The bridge proof is checked *inside* the counterproof SP1 program. Mosaic roles are **garbler = watchtower, evaluator = operator** (`strata-bridge crates/mosaic-client-api/src/types.rs:17-28`) **(V)**.

## 1. ASM: [[Anchor State Machine]]

**State** (`asm crates/state/ssz/state.ssz`) **(V)**:
- `spec_id` and `magic`.
- `chain_view.pow_state`: the bitcoin header verification state, including total accumulated PoW.
- `chain_view.history_accumulator`: an MMR of per-block manifests, indexed by L1 height.
- `sections`: one per subprotocol.

**STF per block** (`crates/stf/src/transition.rs:27-110`) **(V)**:
1. Check block integrity and header PoW continuity.
2. Group transactions by subprotocol (magic bytes).
3. Run load → process → finish over the subprotocols.
4. Emit a manifest with logs, and append it to the history MMR.

**Subprotocols, in order** (`crates/spec/src/spec.rs:27-31, 69-73`) **(V)**:

| Subprotocol | What it does |
|---|---|
| Admin | Multisig updates (`txs/src/actions/updates/mod.rs:41-53`): OperatorSet, Sequencer, **OlStfVk** (→ checkpoint predicate), **AsmStfVk** (emits an `AsmStfUpdate` log → Moho `next_predicate`), EeStfVk, SafeHarbor, Defcon (`subprotocol/src/handler.rs:229-253`) |
| Checkpoint | State: sequencer key, `checkpoint_predicate`, pending predicate transitions activated at an L1 boundary, verified tip, and a deposit pool (`verification/ssz/state.ssz:25-49`). Per checkpoint tx it checks the envelope key equals the sequencer key, then progression, then the proof against the predicate, then withdrawals against the pool (`subprotocol/src/handler.rs:47-107`, `verification/src/state.rs:186-206`). It then relays each intent as `DispatchWithdrawal` to the bridge (`handler.rs:123-126`) |
| Bridge (V1, or V2 under spec 1) | State: operators, deposits, assignments, denomination, fee, safe harbor (`bridge/state/src/bridge.rs:22-44`). See the next table |

What the bridge subprotocol does with each input:
- **Deposit**: validated against the DRT given as aux data. It notifies the checkpoint subprotocol (`DepositProcessed`) and logs a `DepositLog` (`subprotocol/src/handler.rs:47-77`).
- **DispatchWithdrawal**: splits the intent into denomination-sized pieces and creates one assignment per piece (`subprotocol.rs:199-243`).
- **WithdrawalFulfillment**: must match the assignment's amount and destination (`validation/withdrawal_fulfillment.rs:28-61`). On a match it removes the assignment and emits a `NewExportEntry(claim-unlock leaf)` (`handler.rs:79-97`).
- **Expired assignments** are reassigned on every block (`subprotocol.rs:134-167`).
- **Accumulated PoW** is written to the export container's `extra_data` on every block (`subprotocol.rs:169-177`).
- **Slash/Unstake** remove the operator.

Two details:
- The claim leaf names the *assignee*. Who actually paid is not checked (`bridge/types/src/claim.rs:34-36, 80-82`) **(V)**.
- Spec 0 commits `OperatorClaimUnlockV0` (deposit_idx, operator_idx). The bridge proof needs **`OperatorClaimUnlockV1`** (deposit_idx, MuSig2 pubkey), which only `StrataAsmSpecV1` (spec 1) commits (`crates/spec/src/spec.rs:42-73`). The comment says no live chain runs it yet (`claim.rs:83-84`) **(V)**. So the bridge proof as written needs spec 1 at genesis or activated **(I)**.

## 2. Moho

- **MohoState** = `{inner_state: Bytes32, next_predicate: PredicateKey, export_state: [ExportContainer{id, extra_data, entries_mmr}]}`. For bridge-v1, `extra_data` is the accumulated PoW and the MMR holds claim unlocks (`moho crates/types/ssz/moho.ssz`) **(V)**.
- **Step proof (ASM program):** for one bitcoin block, it proves `(parent blkid, H(MohoState_pre)) → (blkid, H(MohoState_post))`. The inner state is the SSZ tree root of `AnchorState`. The export state is advanced from the `NewExportEntry` and `ExportExtraDataUpdate` logs. `next_predicate` is replaced by an `AsmStfUpdate` log (`asm crates/proof/statements/src/moho_program/program.rs:33-46, 79-112`; `input.rs:53-65`; `moho crates/runtime-impl/src/runtime.rs:71-87`) **(V)**.
- **Recursive proof** (`moho crates/recursive-proof/src/statements.rs:38-74`) **(V)**:
  1. Merkle-check that `step_predicate` is committed in the step's *from* MohoState.
  2. Verify the step proof against it.
  3. Verify the previous recursive proof against `moho_predicate`.
  4. Chain the two.
  - Output: `MohoRecursiveOutput{attestation: (genesis StateRefAttestation, proven StateRefAttestation), moho_predicate}` (`io.rs:70-82`).
- **Cadence:** one ASM step proof and one Moho proof **per L1 block**. Multi-block ranges are rejected (`asm crates/extensions/prover/worker/src/state.rs:175`, `input.rs:105-111`) **(V)**.
- **Genesis / trust anchor** (`strata-bridge crates/proofs/bridge-proof/src/genesis.rs:78-96`) **(V)**:
  - The genesis attestation is `(anchor blkid, H(MohoState{inner = H(genesis AnchorState under spec_id), next_predicate = asm_vk, export = empty}))`.
  - It is built from `asm-params.json`, `asm-vk.json` and the genesis spec ID.
  - It is baked into the guest ELFs with `include_bytes!`, together with `moho_vk` (`guest-builder/sp1/guest-*/src/main.rs`).
- **Upgrades:** the ASM step program changes through `next_predicate`. "The Moho recursive program is fixed per node" (`asm docs/asm-upgrade-development.md:8-17`) **(V)**.

## 3. Bridge proof (operator)

`strata-bridge crates/proofs/bridge-proof/` **(V)**:

| | Field | Notes |
|---|---|---|
| Embedded | `moho_vk`, `genesis_moho_state` | `types.rs:11-18` |
| Input | `moho_state`, `moho_proof`, `claim_unlock: OperatorClaimUnlockV1`, `claim_unlock_inclusion_proof` | `types.rs:21-34` |
| Output | `total_pow [32]`, `claim_unlock (u32 + 32B)`, `mmr_idx u64` = **76 bytes** SSZ | `types.rs:37-47` |

Statement (`statements.rs:31-71`, `common/src/lib.rs:126-172`):
1. The Moho proof builds on the embedded genesis.
2. It commits to `moho_state`.
3. It verifies under `moho_vk`.
4. The bridge-v1 container exists.
5. `sha256(ssz(claim_unlock))` is in its MMR.

What it does **not** assert: which chain tip it is, its depth, or freshness. A minority fork with the claim also passes. The heavier-chain counterproof covers that **(V by omission)**.

How it is posted: a borsh `ProofReceipt` in `OP_RETURN` output 0 of the Bridge Proof tx. That tx spends the ContestProof connector with the operator key tweaked by `game_index` (`tx-graph/src/transactions/bridge_proof.rs:20-51`; `connectors/src/timelocked.rs:147-200`) **(V)**.

**The 36 bytes are not the bridge proof.** They are the counterproof's public values (see §4).

## 4. Counterproof (watchtower)

`crates/proofs/bridge-counterproof/` **(V)**:
- **Embedded:** `bridge_proof_vk`, `moho_vk`, `genesis_moho_state` (`types.rs:11-21`). The bridge proof VK is baked in at build time (`guest-builder/sp1/build.rs:79-97`).
- **Input:** `game_idx`, `operator_pubkey`, `n_of_n_pubkey`, `proof_timelock`, the raw Bridge Proof tx and its prevouts, the input index, and `mode` (`types.rs:72-98`).
- **Output:** `operator_pubkey [32] ‖ game_idx u32` = **36 bytes**. "Field order is load-bearing" for the garbled circuit (`types.rs:100-111`).

Steps (`statements.rs:40-265`):
1. Verify the operator's Schnorr signature on the Bridge Proof tx against the ContestProof connector key. This binds the proof to this operator and game (`:79-108`).
2. **Auto-win** if any of these hold: sighash ≠ DEFAULT, no output 0, no `OP_RETURN <data>`, the receipt or output doesn't parse, or the claim is for another game (`deposit_idx ≠ game_idx-1`, or a different pubkey) (`:113-171`).
3. Then one of two modes:

| Mode | Watchtower proves | Covers |
|---|---|---|
| `InvalidBridgeProof` | `bridge_proof_vk.verify(...)` **fails** (`:174-186`) | bad SNARK, wrong genesis, wrong Moho VK, claim not in MMR |
| `HeavierChain` | a valid Moho proof from the **same genesis**, whose bridge container PoW is **strictly greater** than `total_pow`, and either `num_entries ≤ mmr_idx` or leaf `mmr_idx` is a **different** claim (`:187-258`) | a wrong/minority chain, and a **missing withdrawal fulfillment** on the canonical chain. The claim never reached the heavier chain's MMR, so the honest MMR is shorter or holds another claim at that index |

"Missing fulfillment" is **not** its own mode. It reduces to the honest chain lacking the leaf.

Honest watchtower logic (`bridge-exec/src/graph/counterproof.rs:66-105, 369-425`):
- It contests a claim that has no prior fulfillment (`bridge-sm/src/graph/machine.rs:176-177`).
- It counterproves only if the proof is bad, or if its own canonical Moho state is heavier and contradicts the claim.

## 5. g16 circuit, vkeys, and upgrades

- g16 is the **SP1 v6** Groth16 verifier as a boolean circuit. Generating it needs ~44 GB of RAM (`g16 README.md:3, 27`) **(V)**.
- **Compile-time constants** (`g16gen/src/circuit_args.rs:29-80`) **(V)**:
  - The Groth16 VK, which is the same for every SP1 program.
  - **`sp1_vkey_hash`**, the program commitment.
  - `exit_code`, `vk_root` and `proof_nonce`.
  - These are folded into the VK's constant term.
- **Runtime inputs:** 36 public bytes, hashed with **blake3**, plus a 128-byte compressed gnark proof (`:13-24`; `pipeline/README.md:49-64`) **(V)**. Only the counterproof guest enables `blake3` (`guest-counterproof/Cargo.toml:12`) **(V)**.
- **Mosaic input wires** (`mosaic crates/common/src/constants.rs:45-50`; `strata-bridge crates/mosaic-client-api/src/types.rs:36-45`) **(V)**:
  - Setup: 32 bytes = the operator pubkey.
  - Deposit: 4 bytes = `game_idx`.
  - Withdrawal: 128 bytes = the proof.
  - So the watchtower posts **132 adaptor-completed signatures** in the Counterproof tx (`connectors/src/contest_counterproof.rs:10-19`).
- **Cut-and-choose parameters:** 181 circuits, 174 opened (`constants.rs:9-10`) **(V)**.
- **Pinning:** at startup the node checks that the Mosaic circuit's `vk_hash` equals the **counterproof** host vkey (`bin/strata-bridge/src/mode/services/startup_checks.rs:181-200`). It also checks both ELFs against the `bridge_proof_predicate` and `counterproof_predicate` params (`:105-119`) **(V)**.

What each upgrade requires:

| Change | New counterproof vkey? | New g16 circuit and Mosaic setup? |
|---|---|---|
| ASM STF upgrade (AsmStfVk) | no. The step predicate lives in MohoState | **no** **(V/I)** |
| OL checkpoint predicate (OlStfVk) | no | **no** **(V/I)** |
| Bridge-proof program, Moho program, ASM genesis | yes. They are embedded in the counterproof guest | **yes** **(I)** |
| SP1 version | yes, and the g16 constants are SP1-v6-specific | **yes**, possibly g16 code too **(I)** |

How setup scales:
- **A setup is per ordered operator pair**, not per deposit. At boot each node runs `ensure_mosaic_setup` with every other operator, in both roles (`bin/strata-bridge/src/mode/services/mosaic_client.rs:137-170`) **(V)**.
- **Per deposit**, only the lightweight `init_*_deposit` runs: adaptors for the 4-byte deposit input and all withdrawal values (`mosaic-client-api/src/api.rs:76-104`) **(V)**.
- **A tableset is consumed by one contested withdrawal.** Later deposits can still attach, but it can't serve another game's counterproof (`api.rs:117-160`; `mosaic crates/cac/protocol/src/garbler/stf.rs:25-27`) **(V)**. This matches "a watchtower can block an operator once" ([[Watchtower]]). How a pair re-runs setup was not found **(I: open)**.
- Old deposits keep their per-deposit adaptor and fault keys in the graph. So a circuit change probably means re-signing graphs or migrating the old deposits **(I)**.

## 6. Transaction graph per deposit

One game graph exists **per (deposit, operator)**. The N-1 other operators are its watchtowers (`bridge-sm/src/graph/watchtower.rs:1-5`; `tx-graph/src/game_graph.rs:150-178`) **(V)**. Connectors and paths:

| Tx | Spends (path) | Who / when |
|---|---|---|
| Claim | operator funds | operator, after fulfillment |
| Uncontested Payout | Deposit (N/N), ClaimContest **timeout**, ClaimPayout | operator, `contest_timelock` after Claim |
| Contest | ClaimContest (N/N + watchtower_i sig) | any watchtower, before `contest_timelock` (`connectors/src/claim_contest.rs:12-14, 78-91`) |
| Bridge Proof (not presigned) | ContestProof (operator key tweaked by game_idx) | operator |
| Bridge Proof Timeout | ContestProof **timeout** (`proof_timelock`) + ContestPayout | watchtowers, kills contested payout |
| Counterproof_i | ContestCounterproof_i (132 operator adaptor sigs) | watchtower i, after Bridge Proof |
| Counterproof NACK_i (not presigned) | Counterproof_i connector, key = `wt_i_fault` | operator, using the fault secret from evaluating the GC (`bridge-exec/src/graph/counterproof_nack.rs:73-93`) |
| Counterproof ACK_i | Counterproof_i **timeout** (`nack_timelock`) + ContestPayout | watchtower i, blocks contested payout |
| Contested Payout | Deposit, ClaimPayout, ContestPayout **timeout** (`ack_timelock`), ContestSlash | operator, once every counterproof is NACKed or none was posted |
| Slash | ContestSlash **timeout** (`contested_payout_timelock`) + operator Stake | anyone. The stake is **split evenly among all watchtowers** (`tx-graph/src/transactions/slash.rs:81`) |
| Admin Burn / Unstaking Burn | ClaimPayout tapleaves (admin k-of-n / unstaking preimage) | [[Payout Administrator]] (`connectors/src/claim_payout.rs:85-97, 174-183`) |

Timelocks. Dev and test values in `bin/dev-cli/params.toml:35-49`, all relative to the parent tx:

| Timelock | Value (blocks) |
|---|---|
| `contest_timelock` | 144 |
| `proof_timelock` | 144 |
| `nack_timelock` | 144 |
| `ack_timelock` | 432 |
| `contested_payout_timelock` | 1,008 |
| `unstaking_timelock` | 2,016 |
| `recovery_delay` | 1,008 |

Enforced ordering (`crates/common/src/params.rs:140-174`): `proof < ack`, `proof+nack < ack`, `ack < contested_payout < unstaking`. No mainnet values are in the repo.

**Where the code differs from the notes:**
- [[Bridge Withdrawal Flow]]:
  - The ASM assigns an operator **in the same block** that it processes the checkpoint (`asm .../bridge/subprotocol/src/subprotocol.rs:199-243`). I found no 12-block checkpoint maturity in the ASM. Any maturity would be in the clients **(V for ASM)**.
  - An intent can be any **multiple** of the denomination and is split into several assignments (`bridge/types/src/withdrawal.rs:73-90`). "Fixed 2 BTC" is a parameter: dev uses 10 BTC.
  - 504 and 1,064 are not in code. They map to `assignment_duration` and `contest_timelock` (144 in dev).
- [[Glock Challenge-Response Game]]:
  - The order Claim → Contest → Bridge Proof → Counterproof → NACK (operator) / ACK (watchtower) matches.
  - The garbled circuit verifies the **counterproof**, and its leaked secret lets the **operator NACK**. It is not a fraud proof against the operator. A bad bridge proof is caught by a *valid* counterproof plus the ACK timeout.
  - Missing from the note: **Bridge Proof Timeout**, the ACK window being `ack_timelock` from Contest, and Slash only after `contested_payout_timelock`.
  - The slashed stake goes to **all** watchtowers in equal parts, not just the winner ([[Watchtower]], [[Operator Stake and Slashing]]).

## 7. A second SNARK account (e.g. LEZ)

**Intents are account-agnostic.**
- Any OL account that sends a value-carrying message to `BRIDGE_GATEWAY_ACCT_ID` gets a `SimpleWithdrawalIntentLogData{amt, selected_operator, dest}`, emitted under the **gateway's** serial. The sender is not recorded (`alpen crates/ol/stf-v1/src/account_processing.rs:72-73, 159-195`; `bridge_message.rs:42-72`) **(V)**.
- The ASM accepts only logs from `BRIDGE_GATEWAY_ACCT_SERIAL` (`asm .../checkpoint/verification/src/verification.rs:221-273`) **(V)**.
- The claim leaf, bridge proof and counterproof hold only `deposit_idx` and the operator pubkey. Nothing about accounts appears in them **(V)**.

What this means for a second account:
- LEZ needs **no change to the ASM, the proofs or the circuits**. It only needs its withdrawals to appear as gateway messages inside an OL checkpoint that passes the `checkpoint_predicate` ([[SNARK Account]], [[Strata and LEZ Integration]]) **(I)**.
- The deposit pool is a **shared global** limit on withdrawals (`verification/src/state.rs:191-201`). LEZ withdrawals compete with Alpen withdrawals for the same UTXOs. A buggy or malicious account that the OL accepts can drain any deposit. Isolating accounts is the OL STF's job (balances), not the bridge's **(I)**.
- `DepositLog.destination` is opaque bytes (`asm crates/logs/src/deposit.rs:9-14`). The OL decides which account is credited **(V/I)**.

Related: [[SP1 in Strata]], [[Mosaic]], [[Glock]], [[Strata Checkpoint]], [[Protocol Administration]], [[Verification Key Upgradeability]], [[Bridge Timing Parameters]], [[LEZ Burn Verifiability]]
