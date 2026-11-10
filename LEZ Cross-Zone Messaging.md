---
tags: [lez, logos, cross-zone, bridge, trust-model]
created: 2026-11-04
source: logos-execution-zone origin/dev @ 4fba494e2 (2026-10-10)
---
# LEZ Cross-Zone Messaging

How one LEZ zone delivers a message (e.g. "mint wrapped tokens") to another, and what the target actually checks. This is the path a "Strata verifier zone → LEZ" design would use ([[Strata and LEZ Integration]]).

## The flow

```
SOURCE ZONE (peer)                         Bedrock                 TARGET ZONE (LEZ)
user tx → bridge_lock.Lock                                         sequencer: cross_zone_watcher
  ├ escrow native balance                                            reads peer's FINALIZED blocks
  └ chain-call cross_zone_outbox.Emit                                screen + link (hash chain)
       writes OutboxRecord PDA      ── block inscribed ──▶          extract_emission(tx instruction)
peer sequencer signs block            on peer's channel              build inbox Dispatch tx (pending list)
                                                                   block production: inject Dispatch
                                                                     cross_zone_inbox.Dispatch
                                                                       replay check (seen shard)
                                                                       chain-call target with source marker
                                                                     wrapped_token.Mint
                                                                       caller == inbox, marker ∈ sources
                                                                       per-mint cap, lifetime cap
                                                                   indexer: cross_zone_verifier
                                                                     re-derives the same Dispatch
                                                                     from the peer chain; mismatch = halt
```

1. **Emit (source).** A user calls an *emitter* program at top level. Only two are recognised: `ping_sender` (demo) and `bridge_lock` (`lez/cross_zone/src/lib.rs:64`). `bridge_lock.Lock` escrows the user's native balance and chain-calls `cross_zone_outbox.Emit`, which writes a write-once `OutboxRecord {emitter, target_zone, ordinal, target_account_id, target_accounts, payload}` into a PDA. Emitters refuse to be called by other programs.
2. **Transport.** The peer's sequencer inscribes the block on its Bedrock channel. Bedrock only checks that the inscription is signed by the channel's scheduled accredited key.
3. **Watch (target sequencer, `lez/sequencer/core/src/cross_zone_watcher.rs`).** One task per configured peer reads the peer's *finalized* blocks:
   - `screen_peer_block`: recompute the block hash. The signature doesn't cover `header.hash`, so the declared one is never trusted.
   - `link_to_tip`: accept only the next block that hash-links to the last accepted one, starting from the peer's genesis. This stops a peer from claiming block ids ahead of its chain and burning replay keys. Equivocation at the same id is detected and halts delivery from that point.
   - Committee floor: if the peer's live accredited-key count is below `min_committee_size`, reading is suspended (fail-closed). Default 0 = disabled.
   - `record_block_deliveries`: for each public tx in the block, decode the **top-level instruction data** with `extract_emission`. If it targets this zone, build an inbox `Dispatch` tx and add it to a pending list.
4. **Deliver (target block production).** The sequencer injects the `Dispatch` as a sequencer-only tx (users can't call the inbox).
   - `cross_zone_inbox.Dispatch` (`lez/programs/cross_zone_inbox/src/main.rs`): asserts `l1_inclusion_witness` is `None`, refuses messages from its own zone, records `(src_zone, src_block_id, src_tx_index)` in a per-peer-block **seen shard** (replay protection; one block hash per id), then chain-calls the target with the **source marker** PDA (derived from `src_zone` + `src_account_id`) at position 0.
   - The inbox **doesn't authorise anything**. It authenticates transport only. Each target must check the marker itself.
5. **Mint (target program).** `wrapped_token.Mint {recipient, amount}` asserts caller == inbox and that the marker matches an authorised `(src_zone, src_account_id)` source, then enforces `MAX_MINT_AMOUNT` (u64::MAX per mint) and the source's lifetime `mint_cap`. Sources are seeded at genesis from the operator's `allowed_routes`; an optional `source_authority` can update them (renounce-only, no rotation). A capped route requires an authority, or the cap would be a fuse that strands escrow.
6. **Verify (target indexer, `lez/indexer/core/src/cross_zone_verifier.rs`).** The indexer independently reads the peer chain (same acceptance rules) and **re-derives each Dispatch byte-for-byte**. A mismatch is a forgery and halts ingestion; a missing peer block is retried. This stops the *target's own sequencer* from injecting fake deliveries.

## What is and isn't checked

| Checked | Not checked |
|---|---|
| Peer block signed by an accredited key on the peer's channel (Bedrock) | **Whether the source tx actually succeeded or the peer state transition is valid.** The watcher decodes the tx's instruction data; it doesn't re-execute the peer block. |
| Bedrock finality of the peer block | Any proof of peer state (`l1_inclusion_witness` "MUST be None in v1") |
| Hash-chain continuity from peer genesis; no id skipping | k-of-n co-signatures (one accredited key per block) |
| Replay: each `(zone, block, tx index)` once | |
| Target program authorises the `(zone, program)` source; per-mint and lifetime caps | |
| Target's own sequencer can't forge deliveries (indexer re-derivation) | |

So **a dishonest peer sequencer can put a `Lock` that would have failed (no balance) into a finalized block, and the target will mint up to the cap.** The peer's own honest indexers would refuse that block (they park on invalid blocks; [[LEZ Burn Verifiability]]), but nothing propagates that verdict to the target. Trust = peer sequencer key + Bedrock finality + caps.

The crate says this design is valid **only because the peer runs identical LEZ code**: the target recognises emitters by LEZ program ids and decodes LEZ instruction formats. A non-LEZ peer "needs a separate adapter with its own block-reading, emission-extraction, delivery-building, and trust model" (`lez/cross_zone/src/lib.rs:7-11`).

## Mapping a "Strata verifier zone" onto this

Idea: a dedicated zone verifies Strata proofs (checkpoint + deposit inclusion + bitcoin finality), then emits a cross-zone mint to LEZ.

- **If the zone runs LEZ code**, it plugs in as-is. It needs its own emitter program (e.g. `strata_deposit`) that verifies the proof and chain-calls the outbox. LEZ then needs:
  - that emitter added to `extract_emission`, a code change on both watcher and indexer, since emitters are hardcoded;
  - a route `(verifier_zone, strata_deposit) → wrapped_token` with a `mint_cap`, plus a `source_authority`;
  - `min_committee_size` set for the peer.
- **The proof check runs only on the verifier zone.** LEZ's watcher reads the emitter's instruction data, not the result. A dishonest verifier-zone sequencer can include a `strata_deposit` tx with a bogus proof, and LEZ mints anyway. Note also that the instruction data is what LEZ reads, so the emitter instruction itself must carry recipient and amount.
- **Cycle cap still applies.** As a LEZ instance, the verifier zone has the same 10M-cycle cap per tx, so SP1 Groth16 verification (~50M cycles in RISC0) doesn't fit unless that zone raises the cap or adds a native verifier. Using a non-LEZ stack avoids the cap but needs the new non-LEZ adapter on LEZ.
- **Trustless upgrade path**: the reserved `l1_inclusion_witness` slot. A future LEZ could carry the Strata proof itself (or a proof of the verifier zone's execution) and have the inbox or target verify it, which hits the same verification-cost question.

## What Bedrock (L1) enforces on a zone's channel

Source: logos-blockchain @ 3926a53, `core/src/mantle/channel.rs`, `ops/channel/{inscribe,config}.rs`.

Bedrock has **no concept of a zone block**. It has **channels**: an ordered chain of opaque inscriptions. A zone block (e.g. a LEZ block) is just the bytes of one inscription.

- **`ChannelState`** (in the L1 ledger): `accredited_keys`, `configuration_threshold`, `tip_message` (last inscription's `MsgId`), `config_tip_hash`, round-robin fields (`tip_sequencer`, `tip_slot`, `posting_timeframe`, `posting_timeout`), and `transfer_threshold`.
- **`InscriptionOp {channel_id, inscription (≤ 7/8 of max block tx size), parent: MsgId, signer}`**. `MsgId` = hash of the encoded op.
  - **`parent` must equal the channel's `tip_message`**, else `InvalidParent`. On a given L1 chain, a channel's history is **linear and unique**: two inscriptions can't share a parent, so a zone can't fork *on its channel*.
  - **`signer` must be the scheduled key**: `accredited_keys[round_robin(block_slot)]`. The turn rotates every `posting_timeframe` slots, or after `posting_timeout` slots of silence. One plain Ed25519 signature over the tx hash. There's no co-signing for inscriptions.
  - Contents are never interpreted.
- **Channel config** (`ChannelConfigOp`): its own parent chain (`config_tip_hash`), and needs `configuration_threshold` signatures from current keys. It sets keys, rotation timing and thresholds. A channel is also created implicitly by its first inscription, with that signer as the sole key.
- **Funds**: `DepositOp` (L1 notes into the channel), `ChannelWithdrawOp` and `ChannelTransferOp` (need exactly `transfer_threshold` signatures, `verification.rs`). Bedrock checks only value balance, not zone state.

**Consequences**
- A zone's "chain" exists at two levels. At the L1 level, the `parent` MsgId links are enforced by Bedrock. At the zone level, LEZ's `prev_block_hash` is interpreted only by LEZ nodes.
  - Honest LEZ sequencers can't remove a bad block from the channel. The next inscription must have the bad one as its Bedrock parent. They can only skip it at the LEZ level, by pointing `prev_block_hash` at an earlier block.
  - A cross-zone watcher has already accepted and delivered from the bad block. It sees the skipping block as not linking (`DoesNotLink`/equivocation) and halts.
- "This zone message is final" = "the L1 block holding the inscription is final" (`security_param` deep). Ordering within the channel is fixed by the MsgId chain.
- An external verifier could prove "inscription X is in channel C" with an L1 inclusion proof (`body_root`). But it can't prove the channel's current `tip_message` or key set, since there is no ledger state root in the L1 header.

Related: [[Strata and LEZ Integration]], [[LEZ Burn Verifiability]], [[SP1 and RISC Zero Bridging Options]]

> [!note] Superseded design (2026-11)
> The zone team chose a signed outbound-messages channel (2/3 committee signatures, read from L1). See [[LEZ Signed Outbound Channel]]. This note describes the current v1 code.
