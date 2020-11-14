# [⏎](./readme.md) SWIP-003: Peer Swingby Bonding

## Summary

Definition of Skybridge Peer active staking logic
The Swingby Node operators can stake SWINGBY tokens, And bonded to a Swingby Node.

## Motivation

Swingby Skybridge is designed as a permissionless network with no central
authority that can determine who can participate in a group or not. The way this
is achieved is that Swingby Skybridge node operators must prove that they own
SWINGBY tokens on Binance Chain, and lock (or stake) them for as long as they
intend to participate in a TSS group.
To join a TSS group, a participant must acquire SWINGBY and have owned it for a
period of time. They are then able to prove this when TSS rounds (public key
creation, signing events) occur.

## Status

Deployed.

## Specification

To validate a stake on the Binance Chain, we check the following:

1. The stake TX must have exactly one “input”, and an output containing at least the amount of staking tokens required by the network.
2. The qualifying “output” of the stake TX must have been for the full amount of stake tokens required.
3. The stake TX must be at least 72 hours old.
4. The stake TX must contain the peer’s P2P public key and BTC rewards address in a specific format: <P2P public key>,<BTC address>
5. This is to avoid having to establish end-to-end authenticated communication between each peer on the network. We can just observe the Staking TX to get this info.
6. The current balance of the staking address must contain at least the amount of staking tokens required by the network.

### Pre-requisites

None

### Details

None

## License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
