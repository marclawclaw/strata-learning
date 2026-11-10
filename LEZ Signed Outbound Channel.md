---
tags: [lez, logos, cross-zone, bridge, trust-model, design]
created: 2026-11-10
source: LEZ zone team design-decision doc (Notion), 2026-11
---
# LEZ Signed Outbound Channel (cross-zone design decision)

The LEZ zone team chose **Option C**. It replaces the current "parse the peer's blocks and trust its sequencer" model of [[LEZ Cross-Zone Messaging]].

## The design
- Each zone runs a **second Bedrock channel**: the *outbound channel*.
- The zone's sequencers collect the outbound messages (outbox writes) that their **valid** blocks produced, then batch them per main-channel block range. They collect **≥ 2/3 signatures from the committee** (the accredited keys of the main channel) and inscribe the batch.
- Batch shape: `{covered range of main-channel heights, messages[] (each with source coords: block id, tx index, source program; target: zone, program, accounts, payload), signatures}`.
  - The covered range doubles as a **validity watermark**. Blocks in range that produced no messages are attested as quiet.
- Receiving zone: follows **only** the peer's outbound channel. It verifies the signatures against the peer's accredited key set **read from L1**, checks range continuity, and enqueues dispatches. Existing replay keys and seen shards still apply. The indexer's verifier becomes a signature-and-membership check.
- Settled principles:
  - The trust target is **2/3 of the peer committee** (same as the zone's own funds).
  - Signing keys = **accredited channel keys**.
  - **Silence never counts as attestation.**
  - Later, STF **proofs replace the signatures at the same gate**. That is quarters away: public execution isn't provable yet.
- Rejected:
  - A: permission lists.
  - B: a watermark-only channel. It keeps the tx-parsing problems.
  - D: an outbound field in the block body. That's a consensus fork and keeps the LEZ-format coupling.
  - E: one global channel. That's a laundering and governance problem.

## What it removes
- `extract_emission`, the emitter registry, the top-level-only restriction and the "reverted tx still emits" class.
  - Any program writing the outbox emits.
  - Only effects of executed, valid txs are included.
- The LEZ block-format coupling. A non-LEZ peer (SQL zone, custom bridge zone) just produces self-describing signed messages.
- The single-sequencer trust and the "honest skip looks like equivocation" halt, because the receiver no longer chain-tracks the peer's main channel.

## Stated costs
- **Liveness**: delivery halts if the peer committee can't reach the threshold. Needs a staleness signal.
- **Latency**: batching period + L1 finality of the batch inscription.
- **L1 cost**: each message is inscribed twice.
- **Roster**: a captured 2/3 can rewrite its own roster (same as the zone's own trust model).
- A second channel to fund and operate.
- **Multi-hop** isn't transitive.

## Fit with the Strata BTC bridge
- **Deposit (custom bridge zone → LEZ):** this is the k-of-n we wanted, built into the platform. The custom zone no longer needs the LEZ block format. It needs to implement the outbound-channel format and signing round.
- **Withdrawal (LEZ → Strata):** a LEZ burn that writes the outbox yields a **2/3-committee-signed message** on L1. That's much better evidence than today's single sequencer signature ([[LEZ Burn Verifiability]]). The Strata-side account could:
  1. verify those Ed25519 signatures in an SP1 program (cheap); trust = LEZ committee 2/3, the same as LEZ itself, with no separate Strata committee; or
  2. have a Strata committee re-sign (FROST/Schnorr predicate).
- Strata can't read L1, so the account must **track LEZ's key set itself**. Options: follow `ChannelConfigOp` rotations, signed by the old set's `configuration_threshold`, which is self-authenticating; or accept committee-signed rotation messages.
- Still needed on LEZ: a burn instruction in `wrapped_token` (or a bridge program) that writes the outbox with a BTC destination and a multiple of 2 BTC.

## Open points raised (2026-11-10)
1. **Which key set, at which L1 slot.** It must be deterministic for watcher and indexer, and handle a rotation inside a batch's range.
2. **Accredited ≠ staked.** The 2/3 assumption needs the accredited set to equal the staked committee (cf. LEZ issue #1048: the channel creator's key is accredited even when unstaked).
3. **"Heights" are ambiguous after skips.** An honest block reuses a skipped block's `block_id`. Ranges and message coords should include the block **hash** (as `src_block_hash` already does).
4. **Sign only L1-final main-channel blocks**, so an attested block can't later fall out of the fold.
5. **Conflicting batches** (double-signing an overlapping range) should be refused by range continuity and made slashable.
6. **What `configuration_threshold` is** for the main channel. It decides who can rotate the roster.
7. For external verifiers such as a Strata SP1 program: **publish the batch encoding and signing domain as a spec**, with standard Ed25519, domain-separated by channel id. Also publish the key-history rules.

Related: [[LEZ Cross-Zone Messaging]], [[Strata and LEZ Integration]], [[LEZ Burn Verifiability]]
