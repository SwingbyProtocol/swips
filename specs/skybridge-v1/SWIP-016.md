# [⏎](./readme.md) SWIP-016: Flaot Staking

# Summary
Float staking may be done through deposits of the swapped tokens. As example, a BTC to BTC.B swap would have float staking using BTC and BTC.B; it would not use SWINGBY tokens as stake. It is similar to "lending" to the bridge as liquidity provider, and will incur an extra fee to transactions. This extra fee is distributed proportionally to the float stakes.

# Motivation
Any bridge for tokens that are deposit-based will give "float staking". This type of bridge depends on deposits for liquidity, and any user, even users that are not running any node, can stake the bridged tokens to receive float staking rewards proportional to their deposited amount of tokens.
Float staking need to be done with proper risk consideration. 1-to-1 swaps need sufficient deposit on both sides of the bridge to ensure liquidity. Sudden demand in any of the bridge directions may potentially drain deposits on the target chain. 

# Status
Draft

# Specification

TODO

## Pre-requisites
None
## Details
None
# License