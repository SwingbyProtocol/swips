# [⏎](../readme.md) Swingby Cross-Chain Swap Protocol - Skybridge v1

# Table of Contents

- SWIP-001: On-Chain Data &amp; Addresses
- SWIP-002: Swap Process for BTC → BTC.B
- SWIP-003: Peer Swingby Bonding
- SWIP-004: Peer Rewards
- SWIP-005: Key/Value Store
- SWIP-009: Peer Blocking
- SWIP-010: Peer States
- SWIP-011: KeyGen Phase
- SWIP-012: Network ReGroups
- SWIP-013: Key Rotations
- SWIP-014: Additional Security Measures
- SWIP-015: Glossary &amp; Definitions
- SWIP-016: Float Staking
- SWIP-017: Token Minting &amp; Burning
- SWIP-099: Future Work

# Overview

Skybridge is a decentralized cross-chain swap protocol for moving assets between blockchains. It builds trustless bridges between BTC, Ethereum, Binance Chain and other blockchains secured by a network of node groups that facilitates fast inter-blockchain swaps based on Threshold Signature Cryptography (TSS) and Multi-Party Computing (MPC) technology.

## Design Axioms

Skybridge, the next generation of decentralized blockchain bridging protocol, is designed based on the following Design Axioms, listed here in order of priority.

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

## Peer Requirements

- Runs in Docker container (STC preferred) on good hardware
- Fast network connection but can be located anywhere
- Stake of Swingby tokens (Cold Wallet) for min. 72 hours.
- Have to self-send memo signed by peer P2P key to prove ownership and NOT move the tokens otherwise other peers ignore you.
- Container cannot access the staking wallets, just broadcasts its address
- BNB hot wallet (Small amount of BNB) - uses `APP_COLD_KEY_Secp256k1`
- A BTC “rewards” address on the Bitcoin chain, specified in the Staking Transaction memo field.

MAX 100 PEERS per swap pair. TSS gets slower beyond this.

Threshold is 60, Swingby operates 50 active peer containers at start.

## Network Structure

- 10 “Ingress” peers operated on the Cloud by Swingby with RPC open to the Internet, hosted behind a DDoS-protected CDN like Cloudflare.
- The rest of the Swingby peers hosted on STC with only P2P and blockchain access.

## Peer States (Note: Latest Version [Here](./SWIPS-011-Peer-States.md))

Each “round” of the protocol lasts for 10 minutes. There are 6 of these per hour.

Each “round” has a unique identifier round_num, calculated with the following:

```js
floor(epoch_time_secs / (60 * 10));
```

### State Transitions

#### Peer Discovery

> (mins. 1-2, \*6 each hour) - send ping messages every 5s, build peer list, check stakes, make sure baddies are blocked.

Ping message: `peer pk, state, stake tx hash, proof of tss share.`

Correct ping triggers stake check (balance, must have been staked for at least 72 hrs and never moved) on each peer, each adds peer to its peer list. See Peer Staking.
Peers are continuously building active peer lists ordered by seniority (stake time)
