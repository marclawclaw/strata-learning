---
tags: [bridge, economics, security]
aliases: [operator bond]
source: https://docs.alpen.org/how-alpen-works/bitcoin-bridge
---
# Operator Stake and Slashing

Each [[Bridge Operator]] posts a **slashable bond** during [[Bridge Setup Stage|setup]]. It serves two purposes:

1. **It compensates** a [[Watchtower]] that successfully challenges a false claim.
2. **It deters** deviation from the protocol.

Slashing happens when:
- the watchtower wins the [[Glock Challenge-Response Game]] (watchtower ACK), or
- the operator fails to spend a defended payout within ~1.5 weeks.

Bond size isn't published. Alpen's Glock post claims glocks allow a "much smaller" stake than BitVM2's "on the order of 5 BTC", because challenges are much cheaper. Cheaper disputes need less collateral to cover dispute costs.

In the older design, stake was reused across claims via a [[Stake Chain]].
