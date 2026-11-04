---
tags: [strata, rpc, reference]
source: https://docs.alpen.org/build-alpen-apps/rpc-endpoints
---
# Strata RPC Methods

Public endpoint: `https://strata.testnet.alpen.org`. These methods let you inspect the [[Strata Orchestration Layer]] and **independently verify** finality and bridge messages. All the methods below are also served by a local checkpoint-sync node.

**Alpen's [[SNARK Account]] ID**: `0101010101010101010101010101010101010101010101010101010101010101`

| Method | Returns |
|---|---|
| `strata_getChainStatus` | `tip` (blkid, slot, epoch, is_terminal), `confirmed`, `finalized`, `latest` epoch commitments |
| `strata_getCheckpointInfo(epoch)` | `idx`, `l1_range` (bitcoin blocks), `l2_range` (Strata blocks), `confirmation_status` pending/confirmed/finalized |
| `strata_getAccountEpochSummary(acct, epoch)` | epoch/prev commitment, `final_balance`, `final_state_root`, `update_inputs` (seq nos, consumed msgs, extra data) |
| `strata_getAccountGenesisEpochCommitment(acct)` | the epoch where the account was created, a start point for replay |
| `strata_getAsmManifestCommitment(l1_height)` | the [[Anchor State Machine]] manifest tree-hash root for that bitcoin height |
| `strata_getBlocksSummaries(acct, start, end)` | per-slot balance, `next_seq_no`, updates, new inbox msgs (inclusive range, capped by `max_headers_range`) |
| `strata_getSnarkAcctInboxMsgRange(acct, start, end)` | inbox messages `{source, incl_epoch, payload}` (half-open range, max 1000), e.g. **deposit messages** |
| `strata_getSnarkAccountStateByTag(acct, tag)` | `seq_no`, `inner_state`, `next_inbox_msg_idx`, `update_vk` (no balance) for latest/confirmed/finalized |
| `strata_getSnarkAcctUpdateManifest(acct, seq_no)` | consumed inbox range `[prev_next_msg_idx, new_next_msg_idx)`, new state root, extra data |

The inbox methods let anyone check that Alpen **consumed every deposit message in order**.

Related: [[Strata Epochs and Slots]], [[Alpen RPC Methods]]
