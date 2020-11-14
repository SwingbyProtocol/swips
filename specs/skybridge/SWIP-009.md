# [⏎](./readme.md) SWIP-009: Peer Blocking

# Summary

There are two cases we need to protect the network against:
A peer is suddenly not responding due to network issues or downtime
A peer is being actively malicious (sending bad data to the network or targeted peers)
Since each peer is considered to have an equal voice, we follow a similar model to Bitcoin to keep it simple. Each peer maintains a “block list” of peers that it has blocked and the time that an entry exists in this list is configurable by each peer (by default, 72 hours).

# Motivation

Peers are included in the threshold list for the signing round.
The minimal subset of shared peers in all sign sets meeting the threshold is chosen deterministically by each peer.
If a peer advertises a sign set that does not meet the threshold, it is ignored.

# Status

Deployed.

# Specification

Since each peer is considered to have an equal voice, we follow a similar model to Bitcoin to keep it simple. Each peer maintains a “block list” of peers that it has blocked and the time that an entry exists in this list is configurable by each peer (by default, 72 hours).

A peer takes a risk by blocking another peer. Since its sign set becomes smaller, it is selected to perform more work (which consumes more power) due to the following algorithm.

For example, with a threshold of 2, when a single peer (3) decides to block a peer:
Peer 1 - 10 other peers advertised in its sign set
Peer 2 - 10 other peers advertised in its sign set
Peer 3 - 9 other peers advertised in its sign set

In an alternate example, where more nodes are being individually blocked by peers:
Peer 1 - 9 other peers advertised in its sign set
Peer 2 - 5 other peers advertised in its sign set
Peer 3 - 8 other peers advertised in its sign set

## Pre-requisites

None

## Details

None

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
