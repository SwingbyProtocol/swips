# [⏎](./readme.md) SWIP-012: Network ReGroups

# Summary

The TSS algorithm can support reassigning shares among peers to prune out peers that have gone offline or are bad.

This will happen periodically to bring in new peers and maintain the 100 peer pool.

Peers are sorted by seniority in descending order.

If anyone misbehaves during this process, the keygen attempt is skipped and each node records this event and the culprits that caused it.

If this happens repeatedly, a software update must be issued to address the problem.

Meanwhile, the protocol can continue using the old shares.

Important: We must not allow peers to block each other from this process. This can be dangerous (a cartel can enter the network and attempt a takeover).

# Motivation

The state of the signer set is changed then the node is churned out from the current signer set, but this action is essentially happening independently. So each node trying to reset a signer set and it will relocate each share at the same time.

This SWIP defines of the way that to do reconfigure the curernt signer group such that it consists of a new set of signers.

# Status

In testing.

# Specification

This specification is appropriately explained with an example.

Consider this scenario with n=25, t=15:

1. 4 nodes churn out. 4 nodes keep 4 old shares. 4 < t.
2. 5 more nodes churn out. 4 + 5 nodes keep old shares. 4 + 5 < t.
3. 5 more nodes churn out. 4 + 5 + 5 nodes keep old shares. 4 + 5 + 5 < t.
4. 2 more nodes churn out. t < 4 + 5 + 5 + 2. nodes move to a new address. funds are moved.

In reality, the threshold for triggering a full churn would not equal t. It would perhaps be set to something like t\*0.66 and occur regularly every interval T which could be dynamically adjusted based on churn rate.

Even if t old shares can be recovered through 'bribery', the funds would have moved away from the old address. the balance would be 0.

Current 'churned in' nodes are disincentivized to reveal their current key shares:

- A hack or a breach of the active network would have a negative affect on bonded token value.
- The node owners are generating revenue (cash flow) by keeping the network running.

Re-sharing generates a new polynomial for distributing private key `x`, and the nodes update the indexes `k_i` of each node for a re-share. This means that old shares that were produced during older churn cycles cannot be combined into later ones.

1. 4 nodes churn out. 4 nodes keep 4 old shares. 4 < t. Churn cycle #1: 4 old shares.
2. 5 more nodes churn out. 4 + 5 nodes keep old shares. 4 + 5 < t. Churn cycle #2: 5 old shares.
3. 5 more nodes churn out. 4 + 5 + 5 nodes keep old shares. 4 + 5 + 5 < t. Churn cycle #3: 5 old shares.

These 'churned out' nodes could not combine 5 + 5 + 4 shares to produce 14 'shares'.

This would only yield a problem for the network if `t` nodes were 'churned out' in a single churn cycle, leaving a threshold `t` of old shares that could be interpolated on a single polynomial.

## Pre-requisites

SWIP-011

## Details

None

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
