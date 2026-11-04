---
tags: [alpen, critique, security]
---
# Centralization Points

The trust surface today (testnet), gathered from the docs:

- **[[Strata Sequencer]]** and **[[Alpen Sequencer]]**: both single and run by Alpen Labs. They can censor and reorder, but can't forge state, since that needs proofs. Consensus is labelled Proof of Authority.
- **Admin multisigs** ([[Protocol Administration]]): they can swap VKs (i.e. rules) and bridge operator keys. A 2-week delay gives an exit window. Alpen Labs holds all keys on testnet.
- **Bridge operator set**: permissioned, N chosen operators. Safety is [[1-of-N Trust Model|1-of-N]], but **deposits** need all N online to pre-sign.
- **[[Payout Administrator]]**: a human backstop for the one-shot watchtower limitation.
- **[[Security Council and Safe Harbor]]**: an emergency sweep power.
- **Operator capital**: withdrawal liveness depends on some operator having 2 BTC of float ([[Operator Fronts Withdrawals]]).
- **Full nodes trust Strata RPC**: the operator guide configures `[ol] source = "rpc"` against Alpen's endpoint instead of deriving OL state from bitcoin ([[Alpen Full Node]]). P2P uses 3 Alpen bootnodes with discovery disabled.
- **DA**: state diffs only. Full blocks are relayed offchain ([[Alpen is Closer to a Validium]]).

Alpen says it intends to relax these safeguards over time, toward "fully secured by cryptographic code rather than humans-in-the-loop".
