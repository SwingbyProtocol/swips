# [⏎](./readme.md) SWIP-004: Peer Rewards

# Summary

Participating as a full node require both dedicated server resources and meeting the SWINGBY token requirement.
Full nodes will participate in signing transactions to receive the fees incurred by those transactions.
Please remember that nodes are selected based on their age, and removing SWINGBY token stake from the node will reset the age to zero.

# Motivation

There are two types of staking to cover these expenses:

1. SWINGBY staking - nodes that stake SWINGBY to participate in swaps will
   receive swap fee / n.
2. Float staking - nodes that deposit bridge currency will get swap fees
   proportional to their deposited amount of tokens.

This SWIP defines the mechanism of swap rewards distributions. (1.)

# Status

Deployed.

# Specification

1. Peer reward distribution: round(tss_out_addr_sequence % tss_n_count)
2. Awarded on each signed TX. This ensures fair distribution of fees among participating peers.
   Fees/Rewards:
3. To swap BTC to BNB chain, take 0.1% (? to be decided) of the amount for designated peer distributed in BTC.S.
4. To swap in the other direction, we take 0.1% of the amount distributed in BTC.
5. Rewards go to the Staking Address of the peer on BNB chain, or for a swap in the opposite direction, the BTC rewards address specified in the peer’s staking transaction memo.

## Pre-requisites

None

## Details

None

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
