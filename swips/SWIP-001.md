# [⏎](./readme.md) SWIP-001: On-Chain Data & Addresses

## Summary

Definition of Skybridge Swap flows and Address use structure.

Skybridge uses a single custody bridge model for 2 way-pegging.
This SWIP defines the address structure and requirements of Skybridge-v1.

## Motivation

As mentioned above, in designing Skybridge-v1, we built one custody address on each chain.

Each address is generated through a threshold signature scheme, and its private key is distributed and held by an unspecified number of Swingby Nodes. Such a model is very simple and will also be designed without the need for smart contracts.

## Status

Deployed.

## Specification

Requirements of the Skybrige-v1 addresses.

- One custody address on Source chain (BTC) (controlled by TSS group)
- One custody address on Destination chain (BNB) (controlled by TSS group)
- Staking (swingby tokens) staking address (controlled by peer owners)
- The “rewards” address to collect fee distributions, specified in the Staking Transaction. (controlled by peer owners)

### Pre-requisites

None

### Details

None

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
