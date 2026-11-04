---
tags: [governance]
source: https://docs.alpen.org/how-alpen-works/protocol-administration-and-safeguards
---
# Strata Sequencer Manager

The [[Strata Sequencer]]'s pubkey is **hardcoded** and checked during checkpoint verification, so random parties can't hijack block production.

The **Strata Sequencer Manager** is an M-of-N multisig that can **rotate the sequencer key**, for compromise or routine rotation, and rotate its own keys. Unlike other admin updates, a key change takes effect **at the confirmation height**, with no delay, to minimize downtime.

Related: [[Protocol Administration]]
