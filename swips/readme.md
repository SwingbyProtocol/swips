# [⏎](../readme.md) ![Swingby Logo](../images/logo.png) Swingby Improvement Proposals (SWIPs)

> Many of these specifications are still in progress and may not have been fully converted to a proper format. <br />
> The team is actively working on it and always appreciate any help in writing these up.

## Overview

Skybridge is a decentralized cross-chain swap protocol for moving assets between blockchains. It builds trustless bridges between the Bitcoin blockchain, Ethereum, Binance Chain and other distributed ledgers. Swingby consists of node groups that facilitate fast inter-blockchain swaps based on Threshold Signature Cryptography (TSS) and Multi-Party Computing (MPC) technology.

This design documentation targets primarily **developers and a technical audience**. Look here if you are a developer or otherwise technically inclined. 👩🏻‍💻👨🏾‍💻

The protocol specifications are organized into _iterations_ in the follow list.

This emoji legend is used throughout the SWIPS specifications to indicate the completeness status of various designs.

- 🌕 Completed, released.
- 🌓 In progress, on track.
- 🌑 In progress, planned.
- 🔜 Future plans, to be finalized.

Each specification is given a unique identifier in the format: `SWIP-<iteration><index 01-99>` to make it easier to refer to them in pull requests, issues and on task boards. Example: `SWIP-101`.

For finer grained detail about the completion progress of the tasks, follow any of the links below to see a list.

## Table of Contents

- 🌕 [SWIP-001](./SWIP-001.md): On-Chain Data &amp; Addresses
- 🌕 [SWIP-002](./SWIP-002.md): Swap Process for BTC → BTC.B
- 🌕 [SWIP-003](./SWIP-003.md): Peer Swingby Bonding
- 🌕 [SWIP-004](./SWIP-004.md): Peer Rewards
- 🌕 [SWIP-005](./SWIP-005.md): Key/Value Store
- 🌓 [SWIP-009](./SWIP-009.md): Peer Blocking
- 🌕 [SWIP-010](./SWIP-010.md): Peer States
- 🌕 [SWIP-011](./SWIP-011.md): KeyGen Phase
- 🌓 [SWIP-012](./SWIP-012.md): Network ReGroups
- 🌓 [SWIP-013](./SWIP-013.md): Key Rotations
- 🌑 [SWIP-014](./SWIP-014.md): Additional Security Measures
- 🌑 [SWIP-015](./SWIP-015.md): Glossary &amp; Definitions
- 🌑 [SWIP-016](./SWIP-016.md): Float Staking
- 🌑 [SWIP-017](./SWIP-017.md): Token Minting &amp; Burning
- 🌑 [SWIP-018](./SWIP-018.md): Swap Process for BTC to BTCE
- 🌑 [SWIP-019](./SWIP-019.md): A Standard Interface for Data Oracles
- 🌑 [SWIP-020](./SWIP-020.md): Swingby Liquidity Provider ("sbLP") Token Rate Mechanism

## Design Axioms

Skybridge, the next generation of decentralized blockchain bridging protocol, is designed based on the following design axioms.

1. Ease and Flexibility of Integration.
2. Lightness and Low Running Cost.
3. Simple Transfers Only – No Special TX Types.
4. Stateless.
5. Quick.
6. Secure.

## Building Blocks

Skybridge makes use of the following 'building blocks' to implement a solution that satisfies its requirements.

- P2P with Kademlia discovery/routing (based on libp2p, used by IPFS)
- Secure Tunnels (Station-to-Station protocol)
- Identity (Ed25519)
- Encryption (forward secrecy, replay attack protection, anti-MitM)
- NAT Hole-Punching (UPnP)
- Distributed K/V Store (Eventually Consistent)
- Memory-Hard Proof of Work for GPU/ASIC Resistance (argon2d, used here)
- ECDSA/EdDSA Threshold Signature Scheme
