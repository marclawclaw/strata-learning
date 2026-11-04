---
tags: [strata, alpen, sp1, risc0, lez, proofs, bridge, deep-dive]
created: 2026-11-04
source: strata-common a8b11a9 (predicate); zkaleido 6b04259 (main, 2026-11-03) + PR perf comments #72/#104/#119/#203; alpen bc989490; logos-execution-zone origin/dev @ 4fba494e2; docs.succinct.xyz; dev.risczero.com; docs.rs k256 0.14.0
---
# SP1 and RISC Zero Bridging Options

**Question:** LEZ uses RISC Zero (`risc0-zkvm 3.0.5`, LEZ `Cargo.toml:149`). A Strata SNARK account accepts only `NeverAccept`, `AlwaysAccept`, `Bip340Schnorr` and `Sp1Groth16` (`strata-common/crates/predicate/src/type_ids.rs:27`). How does a LEZ adapter produce something that a Strata account accepts? In the other direction, how does LEZ check a Strata (SP1) proof before it mints?

**Short answer:**
- **SP1 → Strata.** Porting the adapter logic to an SP1 guest is cheap. Verifying RISC0 proofs inside SP1 works but is about 30× more expensive with the only existing verifier. LEZ also has no RISC0 proof of anything that matters to Strata, so there is nothing to verify.
- **SP1 → LEZ.** A verified SP1 Groth16 verify in a RISC0 guest costs **~49.7M cycles**. That is about 5× LEZ's **10M-gas per-transaction and per-block cap**, so a LEZ public program can't do it today.
- **Schnorr.** `Bip340Schnorr` is a plain BIP-340 check over `SHA-256(claim)`, so a FROST secp256k1 key works with no prover. The key would be a new secp256k1 key set. LEZ sequencers sign with Ed25519.

Builds on [[SP1 in Strata]] (§ Challenges for LEZ), [[Strata and LEZ Integration]] and [[LEZ Burn Verifiability]].

## 1. Verifying a RISC0 proof inside an SP1 guest

**An existing crate does this, and Alpen benchmarked it.**
- `zkaleido/adapters/risc0/groth16-verifier` (`zkaleido-risc0-groth16-verifier`) wraps `risc0-groth16 3.0.4`. It recomputes the RISC0 `ReceiptClaim` digest from `image_id` plus `SHA-256(journal)` without pulling in `risc0-zkvm` (`src/verifier.rs:67-123`). It pins `control_root` and `bn254_control_id`.
- `risc0-groth16` is built on **arkworks** (`ark-bn254`, `ark-groth16`; zkaleido `Cargo.lock`).
  - Neither zkVM accelerates arkworks. SP1 patches `substrate-bn` and RISC0 patches `substrate-bn` ([SP1 patched crates](https://docs.succinct.xyz/docs/sp1/optimizing-programs/precompiles), [RISC0 precompiles](https://dev.risczero.com/api/zkvm/precompiles)). Neither has an arkworks patch.
- zkaleido had a `groth16-verify-risc0` example that ran this verifier in an SP1 guest. It was **removed in `7b7c9a7`** (2026-04-29) because:
  - `risc0-groth16`/`risc0-zkp` pull `risc0-zkvm-platform` into the SP1 guest dependency graph;
  - SP1 v6's nightly toolchain rejects that crate's `inlateout` asm.
  - Re-adding it is tracked as STR-3248. **Today the RISC0 verifier does not build in an SP1 v6 guest.**

**Measured cycles** (zkaleido CI perf comments, execution only):

| Guest program | SP1 guest | RISC0 guest | Source |
|---|---:|---:|---|
| `groth16_verify_sp1` (SP1 Groth16, `substrate-bn`, patched) | **4.77M** (v6) / 10.7M (v5) | **49.7M** | [PR #203](https://github.com/alpenlabs/zkaleido/pull/203) (2026-11-03), [PR #104](https://github.com/alpenlabs/zkaleido/pull/104) |
| `groth16_verify_risc0` (RISC0 Groth16, arkworks, unpatched) | **162.9M** (SP1 v5) | 163.3M | [PR #104](https://github.com/alpenlabs/zkaleido/pull/104) (2026-03-01), [PR #72](https://github.com/alpenlabs/zkaleido/pull/72) |
| `schnorr_sig_verify` (reference) | 1.31M | 5.67M | PR #203 |

- The arkworks path is about 34× the cost of the `substrate-bn` path in SP1. At SP1's quoted GPU throughput of ~1–2M PGU/s per GPU node ([prover FAQ](https://docs.succinct.xyz/docs/provers/setup/faq)), 163M cycles is roughly **1.5–3 GPU-minutes** before the wrap (est., assuming gas ≈ cycles). That is workable but wasteful.
- **Cheaper route (est.).** Re-implement the RISC0 Groth16 check (5 public inputs: control root ×2, claim ×2, bn254 control ID) over `substrate-bn` reusing `zkaleido-sp1-groth16-verifier`'s pairing code (`verification.rs`). That should land near the SP1 figure, **~5M cycles**. It is a few hundred lines of code and needs care over RISC0's public-input encoding (reversed-byte BN254 control ID, split claim digest). Nobody has built it.

**The bigger problem is that there is nothing to verify.**
- LEZ public transactions are re-executed, not proven ([[LEZ Burn Verifiability]]).
- The only RISC0 proofs on LEZ are **private-transaction (PPE) receipts**. They are **succinct STARK receipts of ~223 KB** (`lez/programs/fee/core/src/market.rs`, `PROOF_BYTES = 223_551`), not Groth16.
- To use them in SP1 you would either:
  - have the user also wrap each receipt to RISC0 Groth16 (extra proving per transaction), or
  - verify the STARK in SP1, which is far more expensive (not benchmarked).
- A PPE receipt proves "this private transaction is valid against some commitment root". It does not prove "this burn is in a finalized LEZ block". That still needs sequencer signatures or a future LEZ block-validity proof (option B in [[Strata and LEZ Integration]]).

## 2. The reverse: an SP1 Groth16 proof inside a LEZ (RISC0) program

**Technically it works.**
- `zkaleido-sp1-groth16-verifier` is **not `#![no_std]`**. Its deps are `substrate-bn 0.6`, `sha2`, `blake3` (no default features) and `thiserror`, and it uses `Vec`.
- RISC0 guests support `std`, so `no_std` isn't required. zkaleido already builds `groth16-verify-sp1` as a **RISC0 guest** (`artifacts/risc0/Cargo.toml` methods list) with `substrate-bn` patched to `risc0/paritytech-bn v0.6.0-risczero.0` (`artifacts/risc0/templates/Cargo.toml.tera`).
- Measured cost is **49.7M RISC0 cycles** (PR #203). That is ~10× the SP1 cost, even with the accelerated `substrate-bn`.
- Version match: zkaleido uses `risc0-build 3.0.5`, the same as LEZ.

**LEZ meters execution in RISC0 cycles, and the caps rule this out.**

| Constant | Value | Where |
|---|---|---|
| `TARGET_GAS_EXEC` | 5,000,000 | `lez/programs/fee/core/src/market.rs:10` |
| `MAX_GAS_EXEC` (per-tx `gas_limit` cap **and** per-block exec total) | 10,000,000 | `market.rs:15`; enforced in `validity.rs:22` (`GasLimitAboveCap`) and `validity.rs:28` (`ExecGasCapExceeded`) |
| `DEFAULT_PUBLIC_CYCLE_BUDGET` (calls with no tx budget) | 32M (`1024*1024*32`) | `lee/state_machine/src/program/mod.rs:28` |
| Charged gas, public tx | executed cycles, clamped to `gas_limit` | `fee/core/src/assess.rs:92` |
| Charged gas, private tx | fixed `PRIVATE_VERIFY_GAS = 409,764` | `market.rs` |

- Gas is literally the RISC0 session cycle count. The executor's `session_limit` maps to `OutOfGas` (`program/mod.rs:112`).
- **49.7M > 10M.** A public LEZ program can't verify a Strata/SP1 Groth16 proof under the current fee constants. It would also take a whole block's execution budget five times over.
- LEZ's own benchmark says every shipped program is under 30k cycles (`docs/benchmarks/cycle_bench.md:145`).

**Ways around it:**
- **(a) Native/host verification.** LEZ adds a built-in (host-side, unmetered or fixed-gas) SP1 Groth16 verifier, like a precompile. Natively that is a few ms of pairings. This is a LEZ protocol change.
- **(b) Private path.** The user proves the verification client-side inside a PPE, and the chain pays only the fixed `PRIVATE_VERIFY_GAS`.
  - 49.7M cycles pads to a 2^26 po2 bucket.
  - At LEZ's measured ~73 µs per padded cycle on an M2 Pro CPU (`cycle_bench.md:86`), that is **~80 min per mint on CPU** (est., by linear extrapolation; a GPU would be far faster).
  - The minted result must then be made public.
- **(c) Raise the caps.** Requires a protocol-version change (`market.rs:1-4` says the constants are protocol constants).
- **(d) Trust the adapter.** The v1 in [[Strata and LEZ Integration]]: mint orders arrive via a cross-zone channel with mint caps, and no proof is checked on LEZ.

## 3. Write once with zkaleido, compile for both

- zkaleido's purpose is exactly this ("reuse of guest logic across multiple ZKVM implementations", `README.md`).
  - Guest logic is a function over `impl ZkVmEnv` (`read_ssz`/`commit_ssz`/`read_serde`…, `zkaleido/src/env.rs`).
  - The `artifacts/{sp1,risc0}` templates build the same example crate into both ELFs, with VM-specific `[patch.crates-io]` for `sha2`, `k256`/`secp256k1` and `substrate-bn`.
- **What it buys:**
  - one audited implementation of the bridge-ledger STF (deposits → mint orders, burns → withdrawal outputs, SSZ `UpdateProofPubParams`);
  - the SP1 build feeds the Strata `Sp1Groth16` predicate, and a RISC0 build of the same logic could be used on the LEZ side (e.g. to recompute the expected Strata claim in a LEZ program);
  - a native build (`zkaleido-native-adapter`) for tests;
  - freedom to switch zkVM later.
- **What it does not buy:**
  - **No cross-VM verification.** `verify_native_proof` is same-VM recursion only (`env.rs:33`). Strata still needs an SP1 proof, so a RISC0 build doesn't help that side.
  - **No LEZ program interface.** LEZ programs are RISC0 guests with LEZ's own plan/apply I/O and account model (`lee/state_machine/src/program/mod.rs`). They are not zkaleido programs. The shared part is a plain library crate called from a thin LEZ guest and a thin zkaleido/SP1 guest.
  - **Dependency-graph risk.** One crate that pulls RISC0 platform code breaks the SP1 v6 build (`7b7c9a7`). The shared crate must stay VM-agnostic.
- **Recommendation:** write the adapter STF as a VM-agnostic library. Host it in an SP1 guest through zkaleido. Don't try to verify RISC0 inside it unless a block-validity proof appears.

## 4. SP1 version pinning

The `Sp1Groth16` condition is a canonical `SP1Groth16Verifier` (`sp1_groth16.rs:30`, `zkaleido .../verifier.rs:106`). Alpen builds it from `sp1_verifier::GROTH16_VK_BYTES`, the program ID and `sp1_verifier::VK_ROOT_BYTES` (`alpen/crates/proof-impl/predicate-keys/src/lib.rs:103-110`, `alpen/provers/sp1/build.rs:186`).

| Component | What it is | Changes when |
|---|---|---|
| **Groth16 VK** | VK of SP1's gnark wrap circuit, the same for every program of an SP1 release | The wrap/recursion circuit changes. The v5 and v6 VKs differ (`examples/groth16-verify-sp1/vk/sp1_groth16_vk_v{5,6}.bin`, different SHA-256s). In v6 the public inputs grew from 2 to 5: `program_vk_hash, hash(pv), exit_code, vk_root, proof_nonce` ([PR #119](https://github.com/alpenlabs/zkaleido/pull/119)). |
| **Program ID** (`program_vk_hash`) | BN254 hash of the guest's SP1 verifying key. Folded into `K0` at load (`verifier.rs:137-146`). | Any guest ELF change, or an SP1 version that changes the core machine/VK derivation (inferred) |
| **VK root** | Merkle root of SP1's allowed recursion-program VKs (new in v6) | Recursion programs change between SP1 releases |
| `vk_hash_tag` | First 4 bytes of `SHA-256(groth16_vk)`, an advisory proof prefix | With the Groth16 VK |

**On an SP1 upgrade:**
- The LEZ adapter must rebuild with the new SP1. That gives a new program ID, and probably a new VK and VK root.
- The account's predicate must be rotated with a `new_predicate` in an update proven under the **old** predicate ([[SP1 in Strata]]), or by admin.
- Strata's own verifier crate must also understand the new proof format. The v5→v6 change broke the format (new prefix fields, new public inputs), so the `zkaleido-sp1-groth16-verifier` version inside the OL/checkpoint guest has to move first. That is Alpen's schedule, not LEZ's.
- If LEZ verifies Strata proofs (§2), it pins the same three constants and must upgrade in lockstep.

## 5. SP1 Groth16 proving cost and latency

- **Local hardware** ([SP1 hardware requirements](https://docs.succinct.xyz/docs/sp1/getting-started/hardware-requirements)):
  - the Groth16 wrap needs ~14 GB RAM (16 GB+ recommended), 16+ cores;
  - optional NVIDIA GPU (CC ≥ 8.0, ≥ 24 GB VRAM, CUDA 12, Linux x86_64);
  - local GPU proving uses one GPU, and parallel proving is network-only.
- **Network** ([SP1 prover-network FAQ](https://docs.succinct.xyz/docs/sp1/prover-network/faq)):
  - fixed STARK→SNARK wrap overhead "~6s" for Groth16 (vs ~70 s for PLONK);
  - for small programs "the fixed overhead of proving will dominate";
  - price = base fee per proof type + PGUs × auction price per PGU, in $PROVE, both dynamic. No fixed dollar schedule is published.
  - Alpen already proves on the network (`SP1_PROVER=network`, `SP1_PROOF_STRATEGY=reserved`, `alpen-ee/bin/prover-perf/README.md:43`).
- **Throughput:** ~2M PGU/s per RTX 4090/5090 node, ~1M on an L4 ([prover FAQ](https://docs.succinct.xyz/docs/provers/setup/faq)). SP1 Hypercube proves 99.7% of Ethereum L1 blocks in under 12 s on 16× RTX 5090 ([Succinct blog](https://blog.succinct.xyz/real-time-proving-16-gpus/), vendor-reported).
- **Estimate for the adapter (est., not measured):**
  - an STF that checks a few Ed25519 burn-batch signatures, MMR proofs and SSZ is probably **1–10M cycles**;
  - adding the RISC0-Groth16 verify adds ~5M (substrate-bn) or ~163M (arkworks).
  - On the network that is roughly **10–60 s of proving plus queue/auction latency per update**. Independent measurements of small SP1 Groth16 proofs on GPU report ~40–50 s end to end ([arXiv 2608.02986](https://arxiv.org/pdf/2608.02986)).
  - Cost per proof is likely **cents to low dollars** (est.). It is small compared with the ~10 h checkpoint cadence, which dominates withdrawal latency ([[Alpen Finality Levels]]).

## 6. The Bip340Schnorr alternative (FROST)

- `SchnorrVerifier` (`strata-common/crates/predicate/src/verifiers/schnorr.rs`) takes:
  - condition: 32-byte x-only key;
  - witness: 64-byte signature;
  - `verify_inner` calls `k256::schnorr::VerifyingKey::verify(claim, sig)` (`schnorr.rs:66-79`).
- In k256 0.14 that **SHA-256-hashes the claim, then runs standard BIP-340 `verify_raw` on the 32-byte digest** ([docs.rs source](https://docs.rs/k256/0.14.0/src/k256/schnorr/verifying.rs.html)). The code comment at `schnorr.rs:72` and a pinned known-answer test (`test_schnorr_known_answer`) lock this in.
- **The signed message is therefore `m = SHA-256(claim)`**, where `claim = UpdateProofPubParams.as_ssz_bytes()` (`alpen/crates/snark-acct-sys/src/verification.rs:139,156-176`). The claim is the same for every predicate type.
- Any BIP-340 signer works, including **FROST with a BIP-340/Taproot ciphersuite** (e.g. ZF `frost-secp256k1-tr`, which handles even-y key normalization). The aggregate signature is an ordinary BIP-340 signature, and nothing in Strata can tell it from a single-key one.
- **Caveats:**
  - **New keys.** LEZ/Bedrock accredited sequencer keys are **Ed25519** ([[LEZ Burn Verifiability]]). A secp256k1 FROST key needs a separate DKG and its own key lifecycle (rotation via `new_predicate`, resharing when the committee changes).
  - **Signers must check what they sign.** Each signer must recompute the full `UpdateProofPubParams` (inbox messages consumed, outputs, ledger refs, any `new_predicate`) before signing. Otherwise a threshold can sign away all the BTC, or rotate the predicate. This is the same trust as option A+ (k-of-n attestation), with no prover.
  - **Rotation path.** A later update signed under the Schnorr predicate can set `new_predicate = Sp1Groth16`, so this does not lock anything in.
  - **Policy.** Whether Alpen will let a bridge-relevant account be Schnorr-controlled is unknown. Account creation was found only at genesis ([[SP1 in Strata]]).

## Comparison

| Option | What Strata verifies | Trust | Cost per update | Work | Who must act |
|---|---|---|---|---|---|
| **A. Verify RISC0 in SP1** | SP1 Groth16 of an adapter that verifies RISC0 Groth16 receipts | Cryptographic for whatever the RISC0 proof proves. **Today that is nothing useful**: no LEZ block proof exists, and PPE receipts are STARKs of private txs only. | +163M cycles (arkworks, measured, SP1 v5) or ~+5M (substrate-bn port, est.), plus RISC0 Groth16 wrapping per receipt | Fix SP1 v6 build (STR-3248) or write a substrate-bn RISC0 verifier. Needs a LEZ validity proof to be worth it. | Adapter team. LEZ for any block proof (Native Zone: post-mainnet, undated). |
| **B. Port the logic to SP1** | SP1 Groth16 of the adapter STF checking sequencer signatures + caps | k-of-n LEZ sequencers (option A/A+) | ~1–10M cycles (est.) plus ~6 s wrap; network fee | Moderate: VM-agnostic STF crate (zkaleido), SP1 guest, network proving, SSZ types tracking Alpen | Adapter team only. LEZ raises its threshold/signs burn batches. |
| **C. `Risc0Groth16` predicate upstream** | RISC0 Groth16 natively (type ID in 20–29) | Same as whatever the RISC0 proof proves | Alpen pays inside the checkpoint SP1 guest: ~163M cycles per update with today's crate, ~5M if rewritten (est.) | Small for LEZ. For Alpen it is a new verifier, an audit, and a change to every guest that runs the STF (OL checkpoint) | **Alpen** (strata-common, alpen, checkpoint guests). Their call. |
| **D. Bip340Schnorr / FROST** | One BIP-340 signature over `SHA-256(ssz(UpdateProofPubParams))` | t-of-n holders of a new secp256k1 FROST key | Nothing to prove; ~1.3M SP1 cycles for Alpen to re-verify in the checkpoint (zkaleido `schnorr_sig_verify`) | FROST DKG + signing service, with signers re-deriving the claim | LEZ committee (or adapter operators), plus **Alpen's consent** for a Schnorr-controlled account |
| *Mint direction: SP1 verify in LEZ* | n/a (LEZ verifies Strata) | Cryptographic for Strata checkpoint inclusion | 49.7M RISC0 cycles (measured), **above the 10M cap** | Native verifier, private-path proving (~80 min CPU, est.), or cap change | **LEZ** protocol team |

**Takeaways:**
- **B (or D) is the v1.** Both give the same sequencer-threshold trust that [[LEZ Burn Verifiability]] says is the ceiling today.
- B costs a prover but keeps the account on Alpen's default predicate type and allows caps and logic in-proof. D costs no prover but puts all checking on the signers.
- A and C only matter once LEZ produces a block-validity proof. When it does, the cheap path is a `substrate-bn` RISC0 verifier in the SP1 adapter, not the arkworks crate.
- In the mint direction, LEZ cannot verify Strata proofs in a public program without a protocol change.

## Open questions

- [ ] Will Alpen accept a Schnorr-controlled or non-Alpen SP1 account at all? What is the account-creation path?
- [ ] STR-3248: will zkaleido restore RISC0-in-SP1, and with a substrate-bn backend?
- [ ] Would LEZ add a native SP1/BN254 Groth16 verifier, or raise `MAX_GAS_EXEC` for verification-heavy programs?
- [ ] What is the actual cycle count of the adapter STF? Prototype and run `just report-sp1`.

Related: [[SP1 in Strata]], [[Strata and LEZ Integration]], [[LEZ Burn Verifiability]], [[SNARK Account Update Proof]], [[Verification Key Upgradeability]]
