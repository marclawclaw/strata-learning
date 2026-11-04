---
tags: [strata, alpen, sp1, proofs, lez]
created: 2026-11-04
source: alpenlabs/alpen bc989490, alpen-ee f66da7fb, strata-common (predicate), asm abf2bd5, moho 9c18376, strata-bridge 31fd75bd, g16 f9697ac (all 2026-10-27..11-03)
---
# SP1 in Strata

Every proof in the Strata stack is an **SP1** (Succinct's RISC-V zkVM) program wrapped to **Groth16 over BN254**. This answers the proving-system half of [[Alpen Open Questions]]. Code goes through Alpen's own zkVM abstraction, `zkaleido`, which also has a RISC Zero adapter. RISC Zero is not used for anything that gets verified on-chain.

## The predicate: what an account (or checkpoint) accepts

`strata-common/crates/predicate` defines `PredicateKey`, a type ID plus a "condition". Only four types exist (`src/type_ids.rs:27`):

| ID | Type | Meaning |
|---|---|---|
| 0 | `NeverAccept` | always fails |
| 1 | `AlwaysAccept` | test/placeholder |
| 10 | `Bip340Schnorr` | a BIP-340 signature by a fixed x-only key |
| 20 | `Sp1Groth16` | an SP1 Groth16 proof. The condition holds the Groth16 VK + **SP1 program ID** + SP1 **VK root**. The claim is the SP1 public values (SHA-256 or Blake3). |

**There is no RISC Zero predicate type.** Adding one is possible (IDs 20–29 are reserved for ZK), but it would be an upstream change to strata-common.

The condition embeds SP1's Groth16 VK and recursion VK root, so a predicate is **pinned to an SP1 major version**. The verifier crate is `sp1-verifier 6.1.0`, and Alpen EE builds with SP1 `v6.8.1`.

## Where SP1 is used: four layers of proofs

1. **Alpen EE account proof** (`alpen-ee/provers/sp1`): `guest-alpen-chunk` proves chunks of EVM blocks. `guest-alpen-acct` aggregates them into the [[SNARK Account Update Proof]].
2. **SNARK account update, verified by the OL** (`alpen/crates/snark-acct-sys/src/verification.rs`):
   - seq-no check;
   - inbox MMR proofs for each consumed message;
   - optional L1 block-ref MMR proofs (`ledger_refs`). This is how an account binds itself to bitcoin blocks, e.g. Alpen's DA.
   - **predicate check of the claim**. The claim is `UpdateProofPubParams`: seq-no, old and new `(state root, next inbox idx)`, processed messages, ledger refs, outputs (transfers, messages, an optional new predicate) and extra data.
   - An update can set `new_predicate`, so **an account can rotate its own verifier** through a proven update.
3. **OL checkpoint** (`alpen/provers/sp1/guest-checkpoint`): an SP1 guest runs the OL STF over an epoch's blocks plus the DA diff and commits a `CheckpointClaim`.
   - The OL params are compiled into the ELF.
   - The STF calls the same predicate verification (`ol/stf-v1/src/proof_verification.rs:184`), so the account proofs are apparently re-verified *inside* the checkpoint proof. That last point is an inference from the code path; I haven't traced it end-to-end.
4. **Bitcoin side.**
   - The **ASM** checkpoint subprotocol verifies checkpoints against `checkpoint_predicate`. The admin subprotocol can replace it ([[Verification Key Upgradeability]]).
   - **Moho** is a recursive SP1 proof of the ASM's state chain over bitcoin blocks. Its "SP1 Groth16 predicate" is the bridge's trust anchor.
   - The **bridge proof** (`strata-bridge/crates/proofs/bridge-proof`) is an SP1 program. It verifies the Moho proof, plus MMR inclusion of the operator's claim unlock in the bridge export entries.
   - **g16** compiles the **SP1 v6 Groth16 verifier into a boolean circuit** (generating it needs ~44 GB RAM). [[Mosaic]] / [[Glock]] garble that circuit, pinned to the **counterproof** program's vkey, not the bridge proof's. The watchtower garbles and the operator evaluates it on the watchtower's counterproof. The counterproof SP1 program checks the operator's bridge proof (posted in an `OP_RETURN`) or a heavier chain. Its public values are the 36 bytes (operator pubkey + game index, blake3). Details: [[Strata Bridge Proof Chain]].

This settles the [[DV-SNARK]] vs Groth16 discrepancy in [[Alpen Doc Discrepancies]] for current code: the garbled circuit verifies an **SP1 Groth16** proof, the watchtower's counterproof.

## Challenges for LEZ

- **RISC Zero can't be verified directly.** The options:
  - (a) the account's SP1 guest verifies RISC0 receipts (BN254 Groth16 verification inside SP1; cost in cycles to be measured);
  - (b) the logic is ported to an SP1 guest (via zkaleido it can be shared with RISC0);
  - (c) Alpen adds a `Risc0Groth16` predicate type. That would also have to be verified inside their SP1 checkpoint guest, so it is their cost and their call.
- **Exact claim format.** The SP1 public values must equal Strata's SSZ `UpdateProofPubParams` byte-for-byte. The adapter has to depend on Alpen's types crates and track their changes (they already split OL spec V1 and V2).
- **SP1 version lock-in.** The predicate pins the SP1 Groth16 VK and VK root. When Alpen moves SP1 versions, the LEZ account's predicate has to be rotated, either by its own update or by admin.
- **Proving cost and latency.** Every update needs an SP1 proof wrapped to Groth16, which takes a GPU prover or Succinct's network and minutes per proof. This drives the cadence and fee questions.
- **Shortcut: `Bip340Schnorr`.** The predicate can be a plain Schnorr key. A LEZ account could be driven by a key with no SP1 at all, for example a **FROST threshold key of the LEZ sequencer committee**. That is the "k-of-n attestation" option in [[LEZ Burn Verifiability]], with no prover. It is equal in trust to an SP1 program that only checks signatures, and it can later rotate to `Sp1Groth16` via `new_predicate`.
  - Open question: will Alpen allow non-SP1 accounts, and how are accounts created? In the code they appear at genesis. A permissionless creation path wasn't found.
- **Bridge security composes.** A LEZ withdrawal is only as safe as LEZ evidence → LEZ account predicate → OL checkpoint → ASM/Moho → bridge proof. Alpen's layers are SP1 throughout. LEZ's layer is the weak link ([[LEZ Burn Verifiability]]).

Deeper dives: [[Strata SNARK Account Lifecycle]], [[Strata Bridge Proof Chain]], [[SP1 and RISC Zero Bridging Options]]

Related: [[Strata and LEZ Integration]], [[Strata Checkpoint]], [[Anchor State Machine]], [[SNARK Account]]
