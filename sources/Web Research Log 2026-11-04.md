---
tags: [source, research-log]
date: 2026-11-04
---
# Web Research Log 2026-11-04

These are summaries of non-docs sources gathered while building these notes. The pages were read through a summarizing fetch tool, so treat wording as paraphrase unless it is in quotes. Each source's content is **as reported by** that source.

## Alpen blog: "Glock: verification on bitcoin" (2025-07-15)
URL: https://www.alpen.org/blog/glock-verification-on-bitcoin (old URL: alpenlabs.io)
- Glock is "a new cryptographic primitive that creates a new standard for verifying offchain computation on Bitcoin".
- Core principle: "authenticated" conditional disclosure of secrets. One party authenticates the computation's input. If the computation fails, another party can derive a secret.
- It moves "the complexity of verifying computation fully outside Bitcoin while securely binding its integrity to Bitcoin".
- Ingredients: binding circuit inputs and outputs to bitcoin lock data, a compact DV-SNARK, and garbling with malicious-security guarantees.
- Claims: up to **1000×** lower onchain cost, "1000× cheaper challenges" (a design goal), and "radically reduced" design complexity.
- BitVM2 operator stake is "on the order of 5 BTC". Glock's would be "much smaller" (no figure given).
- The DV-SNARK has small proofs and lower complexity than Groth16, cutting offchain storage and compute "by orders of magnitude" (no benchmarks).
- BitVM3's RSA-based construction had flaws, reported by **Liam Eagen and Fairgate Labs**. Alpen considers prior garbled-circuit primitives broken or incomplete for a 1-of-N bridge.
- Alpen had researched garbled circuits for over a year before this post.

## Alpen blog: "Glock is here" (2025-08-19)
URL: https://www.alpen.org/blog/glock-is-here
- Shipped: the Glock paper plus 3 OSS repos: a Rust DV-Pari implementation, a binary circuit generator for DV-Pari, and a generic garbling/evaluation tool.
- Glock25 is the concrete garbled lock, using DV-Pari.
  - SAP variant: ">550× onchain efficiency over BitVM2".
  - QAP variant: >430×, plus better offchain storage. **QAP is the variant Alpen pursues.**
- Metric: total data posted on bitcoin in a worst-case challenge-response game.
- "will eventually secure your BTC deposits to Alpen". There is a Glock interest group on Telegram.

## Alpen blog: "Introducing the Strata bridge" (BitVM2 era, now flagged outdated)
URL: https://www.alpen.org/blog/introducing-the-strata-bridge
- The page itself now says: "Do not use this post as the current source of truth for the bridge."
- Strata was then called a "sovereign validity rollup" and admitted to be validium-like, because full blocks are relayed offchain.
- "1 BTC on Strata is backed by 1 BTC on bitcoin."
- Core problems:
  - Bitcoin script must verify Strata's state transition and fork choice.
  - The proof must be anchored to canonical bitcoin, not a private fork.
- The page distinguishes "functional" from "faulty" entities (malicious, malfunctioning, or coerced).
- Flow: deposit → burn → front → kick-off → claim → optimistic payout | challenge (with a fee) → multi-stage assertions → payout | disprove (part of the collateral is burned, the rest goes to the disprover).
- "All we need is for one functional operator to refuse to sign any noncompliant transaction."
- Connectors (with or without value), Winternitz OTS, stake chains, burn txs in sets, anchor blocks (lookback or chained keys), Groth16 claim proof split into subprograms, Taproot disprove.
- Differences from BitVM2: anchor blocks instead of superblocks, no start-time tx, simplified disproving, multi-stage assertions, stake chains.
- Not stated: lock periods, N, M, collateral, challenge fee, burn split.
- See [[Strata Bridge BitVM2-era Design]], [[Stake Chain]], [[Anchor Blocks]].

## Papers
- **Glock**, ePrint 2025/1485, Liam Eagen (Alpen Labs): https://eprint.iacr.org/2025/1485
  - A formal framework for garbled-circuit optimistic contracts.
  - As in **Delbrag**, the circuit leaks a secret that is used to produce a signature, which is the fraud proof.
  - "First concretely practical construction" without Rubin's **Grug** technique.
  - Reduces bounded computation to SNARK verification via a designated-verifier **Pari** variant.
  - Malicious security via cut-and-choose + VSS + adaptor signatures.
  - Claims lower costs than Groth16-based glocks. Concurrent with Delbrag and BitVM3.
- **Mosaic**, ePrint 2026/812: https://eprint.iacr.org/2026/812
  - Received 2026-04-24, revised 2026-07-15.
  - Authors: Nakul Khambhati (Alpen, UCLA), Mukesh Tiwari, Azz, Sapin Bajracharya, Manish Bista, Christian Lewe, Aaron Feickert (Alpen Labs), and Liam Eagen (**Ideal Group**). Eagen's affiliation changed from Alpen.
  - Polynomial (degree-t) label correlation across N cut-and-choose copies. The t opened shares are one short of reconstruction. The adaptor signature reveals the last share.
  - The onchain footprint is independent of N.
  - Instantiated for a bitcoin bridge with a **Groth16 verifier circuit**, a full spec, and a Rust implementation.
  - Linked from the docs bridge page as *the* glock bridge protocol.
- **BitVM2 bridge paper**: https://bitvm.org/bitvm_bridge.pdf

## Press and third-party (lower confidence)
- Testnet III went live on signet **2026-07-28**: the "partner integration phase", described as the "final major validation stage before mainnet".
  - [Kalkine/FinanceWire](https://kalkinemedia.com/uk/news/announcements/financewire-fnews-alpen-launches-testnet-iii-on-bitcoin-signet-advancing-to-critical-partner-integration-stage), [Investegate](https://www.investegate.co.uk/announcement/fnw/financewire-news--fnews/alpen-enters-partner-integration-phase-with-t-/9692199), [Cryptobriefing](https://cryptobriefing.com/alpen-labs-testnet-iii-bitcoin-defi/)
- 21+ launch partners. Bridge operators **Stakely** and **Chainflow**, with **P2P.org** expected ([RootData](https://www.rootdata.com/projects/detail/alpen%20labs?k=MTAwNjY%3D)).
- Testnet III has "the first publicly usable garbled circuit-based Bitcoin verifier".
- Alpen and the Strata bridge are "decoupled and independently upgradeable", so consensus updates no longer need a regenesis.
- Funding: an **$8.5M strategic round led by DBA and Cyber Fun** (2024). It said it would help "launch of Strata into mainnet in 2025", a target that **slipped** ([Bitcoin.com](https://news.bitcoin.com/alpen-labs-raises-8-5-million-to-advance-development-of-bitcoin-zk-rollup-strata), [ChainCatcher](https://chaincatcher.com/en/article/2162175), [Alpen blog](https://alpenlabs.io/blog/strategic-round)).
- **Starknet**: the Starknet Foundation gave Alpen a grant for the **development and audit** of a **shared Glock verifier**, to be a public good for the bitcoin ecosystem ([Starknet blog](https://www.starknet.io/blog/starknet-alpen-bitcoin-glock/), [Alpen blog](https://www.alpenlabs.io/blog/starknet-shared-glock-verifier)).
- **Conflicting claim**: [Zelcore](https://zelcore.io/academy/coin-guides/bitvm2-citrea-alpen-validity-proofs) describes Alpen as using "a proprietary proving system called Glock plus a Groth16 verifier". Alpen's own pages describe a Groth16-based verifier *for* Glock. They are probably describing different components.
- Other pages seen, not relied on: [Stacks blog repost](https://www.stacks.co/blog/alpen-labs-introduces-the-strata-bridge), [Bitfinity "State of Bitcoin"](https://www.blog.bitfinity.network/state-of-bitcoin-bitvm3-glock-bitcoin-dollar/), [GOAT research HackMD](https://hackmd.io/@goatresearch/HkLx3FYigg), [yespress](https://yespress.io/alpen-labs.md), [btcpp interview with John Light](https://insider.btcpp.dev/p/an-interview-with-john-light), [Cryptonomist](https://en.cryptonomist.ch/2026/07/29/bitcoin-native-defi-infrastructure/), [ValueTheMarkets](https://www.valuethemarkets.com/cryptocurrency/news/alpen-labs-milestones-and-innovations-in-bitcoin-defi). Simplify.jobs had garbled text and was ignored.

## Not yet done
- The full PDFs of the Glock and Mosaic papers were not read.
- The source code at github.com/alpenlabs was not reviewed (no local clone at `~/src/alpenlabs`).
