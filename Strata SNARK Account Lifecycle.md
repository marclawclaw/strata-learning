---
tags: [strata, snark-account, lez, integration, code-reading]
created: 2026-11-04
source: alpenlabs/alpen bc989490, asm abf2bd5, strata-common a8b11a9 (predicate, identifiers), alpen-ee f66da7fb
---
# Strata SNARK Account Lifecycle

How a [[SNARK Account]] on the [[Strata Orchestration Layer]] (OL) is created, updated, verified and upgraded, read from source. It answers the open questions in [[Strata and LEZ Integration]]: could LEZ become a second account next to [[Alpen]]?

Legend: **V** = verified in code, **I** = inference.

Paths: `alpen/` = `alpenlabs/alpen`, `stf-v1/` = `alpen/crates/ol/stf-v1/src`, `asm/` = `alpenlabs/asm/crates/subprotocols`.

## TL;DR for LEZ

| Question | Answer | |
|---|---|---|
| Create an account at runtime? | **No.** Genesis only. There is no create-account tx. | V |
| Who may submit an update? | **Anyone** holding a valid proof. Txs carry no signature and no fee. The sequencer decides inclusion. | V |
| Per-account balance? | **Yes.** An account can't spend more than its balance. Withdrawals must be multiples of the denomination. | V |
| Can a deposit target a non-Alpen account with a 32-byte recipient? | **Yes.** It names the account by serial, and the subject can be up to 32 bytes. | V |
| Does the checkpoint proof re-verify each account's proof? | **Yes.** The guest re-runs the full STF, including `verify_claim_witness`. | V |
| Can accounts send messages or value to each other? | **Yes, at the OL level.** Alpen EE already credits value-bearing deposit-typed messages from any sender, but it can only *emit* messages to the bridge gateway. | V / I |
| Predicate upgrade? | The account rotates its own predicate (`new_predicate`). The admin `EeStfVk` path is **hard-coded to serial 128 (Alpen)**, and it is only an inbox message. | V |
| Must accounts post DA to bitcoin? | **No.** The OL doesn't require it. Alpen's DA binding lives in Alpen's own guest. | V |

## 1. Creation: genesis only

- **V** Genesis params hold `accounts: BTreeMap<AccountId, GenesisSnarkAccountData>` (`alpen/crates/ol/params/src/params.rs:34`). Each entry is `{predicate, inner_state, balance (default 0)}` (`ol/params/src/account.rs:16-26`).
- **V** `OLStateV1::from_genesis_params` walks that map. It hands out serials starting at `SYSTEM_RESERVED_ACCTS = 128` and creates a `Snark` account with the given predicate, state root and balance (`ol/state-types-v1/src/toplevel.rs:33-55`; `strata-common/crates/identifiers/src/acct.rs:21`). Called from `build_genesis_artifacts` (`ol/genesis/src/lib.rs:66-95`).
- **V** The tx union has only `GenericAccountMessage | SnarkAccountUpdate` (`ol/tx-types-v1/ssz/transaction.ssz:96-99`). No STF code calls `create_new_account`. Only DA replay and tests do (`ol/da-types-v1/src/types/payload.rs:175`).
  - The DA format already carries a `new_accounts` list (`da-types-v1/src/types/ledger.rs:15,50-74`). Runtime creation is anticipated but not implemented.
- **V** Serials follow `BTreeMap` order, i.e. ascending `AccountId`. Alpen's ID `0x0101…01` gets serial 128.
- **I** Adding LEZ needs a **new genesis** or a **new OL spec**. A new spec could insert the account at an epoch take-over, the way V2 takes over V1 (`ol/stf/src/v2.rs:1-40`). It would be enacted through the admin `OlStfVk` checkpoint-predicate update. Either way the decision is Strata's governance ([[Protocol Administration]]), not permissionless.
- **I** In a multi-account genesis, LEZ's `AccountId` must sort **after** `0x0101…01`. Otherwise LEZ takes serial 128 and receives Alpen's admin predicate rotations (see §7).

## 2. Submitting an update: permissionless, sequencer-gated

- **V** `OLTransactionV1 = {data: {payload, constraints, effects}, proofs}`. It has no signature, no sender and no fee (`transaction.ssz:105-133`). A SAU's sender **is** its `target` (`stf-v1/transaction_processing.rs:57-66`). Authorization comes **only** from the predicate check.
- **V** Path: RPC `strata_submitTransaction` (`ol/rpc/api/src/lib.rs:146`; `bin/strata/src/rpc/node.rs:1289`) → mempool. The mempool checks only slot bounds, that the account exists, and the seq-no range. It does **not** verify proofs (`ol/mempool/src/validation.rs:118-176`).
- **V** At block assembly the **sequencer generates the inbox and L1-ref MMR proofs** for the tx (`ol/block-assembly/src/block_assembly.rs:767,1110-1174`). It then executes the tx. Failures are dropped and later seq-nos for that account are blocked (`:696-800`).
- **I** Liveness depends on the single [[Strata Sequencer]], which can censor. Safety doesn't: a forged update can't pass the predicate.
- **V** GAM txs are also permissionless. A GAM is one **zero-value** message to any account (`transaction_processing.rs:120-143`). Its sender is `SEQUENCER_ACCT_ID`, which is **aliased to `BRIDGE_GATEWAY_ACCT_ID`** (`stf-v1/constants.rs:6`).
  - ⚠ A LEZ guest must **not** treat "source == bridge gateway" as proof of a deposit. It has to credit the message's attached `value`, as Alpen does (`alpen-ee/crates/acct-runtime/src/ee_program.rs:157-161`).

### What the OL checks in an update (`process_update_tx`, `transaction_processing.rs:145-231`)

1. `verify_effects_safe`: every destination exists or is special, and **Σ outputs ≤ account balance** (`:350-382`).
2. `verify_update_correctness` (`alpen/crates/snark-acct-sys/src/verification.rs:11-38`):
   - seq_no;
   - **message index**: `new_next_msg_idx = old + #processed`, so messages are consumed in order and contiguously (`:59-81`; missing from the "already known" list);
   - L1 ledger refs, checked as MMR proofs (`:84-101`);
   - inbox MMR proofs (`:105-130`);
   - the predicate over the SSZ `UpdateProofPubParams` claim (`:133-177`).
   - The claim includes the transfers, the messages **and `new_predicate`** (`snark-acct-sys/src/update.rs:109-127`).
   - The function's doc comment says it checks balances. It doesn't; step 1 does.
3. Every supplied proof must be consumed (`verifier.is_exhausted()`, `:171`).
4. The OL writes the new state root, next message index and `seq_no + 1`, plus any `new_predicate` after a type-id check (`:181-223`).
5. `apply_tx_effects` debits the account once and splits a linear `Coin` per effect (`:234-310`).

Confirmed: there are four `PredicateKey` types (`strata-common/crates/predicate/src/type_ids.rs:32-50`). **I** A `Bip340Schnorr` predicate is allowed. A "SNARK" account can be controlled by a plain key, which would be a cheap way to bootstrap [[Strata and LEZ Integration]] option A, at full key-holder trust.

## 3. BTC balance accounting

| Flow | Mechanism | Evidence |
|---|---|---|
| Overdraw guard | `total_sent > balance` → `BalanceUnderflow`. `take_balance` also fails with `InsufficientBalance`. | `transaction_processing.rs:377`; `ol/state-types-v1/src/account.rs:104-114` (V) |
| Deposit credit | ASM `DepositLog` → OL mints a `Coin` → `process_message(from = BRIDGE_GATEWAY, to = dest)` → `add_balance`, plus an inbox entry | `stf-v1/manifest_processing.rs:333-370`; `account_processing.rs:24-47,98-108` (V) |
| Withdrawal debit | A message to `BRIDGE_GATEWAY_ACCT_ID` with value. The sender is debited, a `SimpleWithdrawalIntentLog` is emitted and the coin is burned. | `account_processing.rs:159-195` (V) |
| Denomination | `amount > 0 && amount % denomination == 0 && ≤ max_withdrawal_amount`, plus a valid BOSD descriptor | `stf-v1/bridge_message.rs:42-73`; `alpen/crates/bridge-params/src/lib.rs:145-151` (V) |
| Invalid withdrawal | **Swept to limbo**, not refunded. The same happens to a plain transfer to the bridge gateway. | `account_processing.rs:169-172,124-128` (V) |
| L1 backstop | ASM checkpoint: each intent is a denomination multiple, and the global count ≤ the deposit-UTXO pool. The bridge splits each intent into per-denomination assignments. | `asm/checkpoint/verification/src/deposit_pool.rs:106-149`; `asm/bridge/subprotocol/src/subprotocol.rs:203-215` (V) |

**I** Per-account isolation is enforced by the OL STF, and so by the checkpoint proof. The ASM only enforces a **global** cap. A buggy or compromised LEZ predicate can drain **LEZ's balance only**. A bug in the OL STF itself could break isolation.

## 4. Deposit routing

- **V** DRT `OP_RETURN` aux data = `recovery_pk[32] || destination` (≤ 42 bytes) (`asm/bridge/txs/src/deposit_request/aux_input.rs:22,38`). The ASM copies `destination` into `DepositLog` without interpreting it (`asm/bridge/subprotocol/src/handler.rs:64-67`).
- **V** The OL decodes it as a `DepositDescriptor` = control byte + 1–3 byte **account serial** + **subject bytes (≤ 32)**, left-padded to a 32-byte `SubjectId` (`alpen/crates/bridge-types/src/deposit.rs:1-5,216-219,313`; `identifiers/src/acct.rs:17,224-229`). An unknown serial or a malformed descriptor sends the funds to **limbo** (`manifest_processing.rs:379-429`).
- **V** **Yes:** a 32-byte LEZ recipient fits (1 + 1 + 32 = 34 bytes ≤ 42). The account receives an inbox `DepositMsgData{subject}` carrying the value.

## 5. Does the checkpoint proof re-verify account proofs? Yes

**V** Trace:

1. `guest-checkpoint/src/main.rs:18`: `process_ol_stf`. `PROVED_SPEC = V2` (`spec.rs:4`). V2 runs the V1 code.
2. `proof-impl/checkpoint/src/statements.rs:46` → `process_ol_stf_core` → `execute_block_batch` (`:324-384`).
3. `strata_ol_stf::construct_block` (`stf/src/block.rs:32`) → `stf-v1/assembly.rs:399` → `execute_block_inputs` → `process_block_tx_segment` (`:159-161`, `transaction_processing.rs:24`).
4. `process_update_tx` → `TxProofVerifierImpl` (`transaction_processing.rs:161-170`) → `verify_local_predicate_next` → `PredicateKey::verify_claim_witness` (`stf-v1/proof_verification.rs:184-200`).

Every account's Groth16 proof (or other predicate) is therefore verified **inside SP1**. A failure panics the guest (`statements.rs:368`). The computed header must equal the block header (`:377`). Txids exclude the proofs (`transaction.ssz:114-117`), but the guest re-checks them whatever bytes are supplied. See [[SP1 in Strata]] and [[Strata Checkpoint]].

## 6. Inter-account messages and value

- **V** `TxEffects` can hold transfers (value only) and messages (value + payload) to **any existing account** (`transaction_processing.rs:277-286,365`). A message to a snark account credits its balance and appends an inbox entry (`account_processing.rs:98-108`). A send to a non-existent account goes to limbo (`:82-90`).
- **V** Alpen EE handles a `Deposit`-typed message from **any** source and credits the attached value to the subject (`alpen-ee/crates/acct-runtime/src/ee_program.rs:157-161`). It ignores `SubjTransfer` and `Commit` (TODO STR-3685, `:163-170`).
- **I** **LEZ → Alpen** works today: LEZ emits a deposit-format message with value to Alpen's account ID, naming an EVM-address subject.
- **I** **Alpen → LEZ** does not work yet. Alpen's EVM only emits messages to `BRIDGE_GATEWAY_ACCT_ID` (`alpen-ee/crates/evm-ee/src/execution.rs:73`). It would need Alpen EE changes.
- **I** Delivery is asynchronous. A message lands in the inbox during the sender's update and is processed in the receiver's next update.

## 7. Predicate upgrades

| Path | Who | Effect | Evidence |
|---|---|---|---|
| Self-rotation | Whoever proves an update under the **current** predicate | `new_predicate` is bound into the claim. It becomes active after this update. The OL places no limit on the successor. | `transaction_processing.rs:199-220` (V) |
| Admin `EeStfVk` | `AlpenAdministrator` multisig (ASM admin) | Emits `EePredicateKeyUpdate` for **serial 128 only** (hard-coded "Alpen is the first account"). The OL **queues an inbox message** from `ADMIN_MSG_ACCT_ID`. It does not replace the key. | `asm/admin/types/src/updates.rs:67`; `asm/admin/subprotocol/src/handler.rs:248-268`; `stf-v1/manifest_processing.rs:449-510` (V) |
| Admin `OlStfVk` | `StrataAdministrator` | Replaces the **checkpoint** predicate (the OL STF program). A new STF can change any account's state or rules. | `asm/admin/subprotocol/src/handler.rs:237-239` (V); its power over accounts is I |

- **V** The admin message is honoured only because Alpen's guest makes it so: it requires `source == ADMIN_MSG_ACCT_ID` and declares exactly the queued key (`ee_program.rs:172-190`; OL comment at `transaction_processing.rs:203-208`).
- **I** For LEZ there is **no admin override at the account level**. Its predicate is fully self-sovereign unless LEZ's own guest adds a policy. The ultimate override is the Strata admin replacing the OL STF. See [[Verification Key Upgradeability]].

## 8. DA requirements

- **V** The OL never requires an account to reference L1. `ledger_refs` is optional, and only the refs the update chooses to include are MMR-checked (`snark-acct-sys/src/verification.rs:89-98`; `SauTxLedgerRefsV1` `Optional`, `transaction.ssz:60-62`).
- **V** Alpen's bitcoin-DA binding is Alpen-specific:
  - the EE sequencer builds `ledger_refs` from its DA tx blocks (`alpen-ee/crates/common/src/utils/ledger_refs.rs:21-37`);
  - Alpen's guest checks the DA-to-L1 binding (`alpen-ee/crates/da/runtime/src/verification/mod.rs:150`).
- **V** What the OL itself posts to L1 for every account is its own state diff: balance, seq_no, inner state root, inbox appends and `update_vk` (`da-types-v1/src/types/account.rs:14-19`, `snark.rs:188-203`). A LEZ account's internal data never has to reach bitcoin unless its own proof requires it. Compare [[Alpen is Closer to a Validium]].

## Open items

- [ ] Will Strata add runtime account creation, using the `new_accounts` DA slot? Ask Alpen Labs.
- [ ] Would a second account make the admin `EeStfVk` path account-generic? Today it is hard-coded to serial 128.
- [ ] Is RPC `submitTransaction` exposed publicly on mainnet sequencer nodes? (Deployment question, not code.)
