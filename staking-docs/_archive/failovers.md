---
description: Infrastructure resilient to failures on multiple fronts.
---

# 🚧 Failovers

### Local failovers

Running two Ethereum stacks (a stack being a Beacon Node that is paired with an Execution Client), one being the primary and the other being the secondary. The validator client is configured to use both, so should the primary stack go offline then it will automatically use the secondary stack until the primary becomes available again.

### Limitations

There are a few limitations that exist on a consensus level layer, where the Validator Clients will continue to use the Beacon Node it is pointed at when the execution client that it is paired with goes offline or is otherwise unavailable.

This is because the consensus engine is _technically_ still online and working, just not for validation purposes as there is nowhere to store or reference the transactions while the execution engine is offline. So in this state, all the attestations will be missed.

However, this issue does not exist across all consensus clients. [Teku has addressed this in their 23.3.0 release](https://github.com/ConsenSys/teku/releases/tag/23.3.0) while the [Lighthouse team are aware and are working on a fix](https://github.com/sigp/lighthouse/issues/3613).

That being said, this issue only occurs in a fairly specific situation and should not cause any major problems for us.
