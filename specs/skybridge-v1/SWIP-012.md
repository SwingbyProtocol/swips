# [⏎](./readme.md) SWIP-012: Network regroup

# Summary
The TSS algorithm can support reassigning shares among peers to prune out peers that have gone offline or are bad. 
This will happen every night at 00:00 UTC to bring in new peers and maintain the 100 peer pool. 
Peers are sorted by seniority in descending order. 
If anyone misbehaves during this process, the keygen attempt is skipped and each node records this event and the culprits that caused it. 
If this happens repeatedly, a software update must be issued to address the problem. 
Meanwhile, the protocol can continue using the old shares.
Important: We must not allow peers to block each other from this process. This can be dangerous (a cartel can enter the network and attempt a takeover).

# Motivation
The State of the TSS signer set is changed then the node is out from the current TSS signer set. but this action is basically happening independently. So each node trying to reset a signer set and it will relocate each share at the same time.
This SWIP defines of the way that to do regroup the curernt TSS group.

# Status
Draft

# Specification

TODO

## Pre-requisites
None
## Details
None
# License