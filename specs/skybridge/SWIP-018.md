# [⏎](./readme.md) SWIP-018: Swap Process for BTC to BTCE


# Summary

Users will be able to simply send their native Bitcoins to a trustless multi-signature address governed by the network validator nodes in return for pegged ERC20 Bitcoin tokens on the ethereum.
Therefore, the address holding BTCs and ERC20 assets are controlled by a single TSS address/private key. private keys are managed by a decentralized network. 

# Motivation

Swingby can also move Bitcoin to Ethereum and move Ethereum assets to other chains. SWIP-018 has defined to model ERC20 assets that move in a trustless way.

# Status

Draft.

# Specification

The steps for swapping native Bitcoin to an ERC20 pegged equivalent are as follows:

1. User wants to swap BTC to BTC.E (Ethereum)
2. User inputs their BTC refund address and ETH destination address
3. The system creates a swap request in the distributed “KV store”
4. User sends their BTC to the Swingby network address (displayed in UI)
5. Swingby validator nodes form a consensus and multi-sign an outbound BTC.E transaction to the users destination address.
6. User now has BTC.E ERC20 tokens on the Ethereum chain

## Pre-requisites

None

## Details

### Communicating with the Ethereum chain
To handle this interaction, We are using the existing and proven abstraction layer [blockbook](https://github.com/trezor/blockbook)
. The Trezor developed indexer plugs into an Ethereum or Bitcoin node and uses a standardised interface for getting transaction information. This means that code written for interacting with the Bitcoin blockchain can be shared and re-used to help integrate the Ethereum chain.

### Supporting ERC20 multi-send
In the world of Ethereum and the current ERC20 standard, multi-send is not supported by default. without multi-send support, the Swingby network would have to sign 50+ individual transactions per swap. Not good. (see the swip-004 for reward distribution)
To solve this issue, the team has designed a [smart-contract wallet](https://github.com/SwingbyProtocol/BEP20Token/blob/master/contracts/MultiSendWallet.sol) that acts as a custodian for the funds traveling through the network bridge. When a user deposits their funds for a swap, they are actually depositing into a contract and when a swap has finished processing, the funds are withdrawn from that contract.

The swingby network address is the sole owner of this contract. This means that the validators still have to form the appropriate consensus in order to withdraw from the contract wallet.

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
