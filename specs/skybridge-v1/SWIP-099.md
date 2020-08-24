# [⏎](./readme.md) SWIP-099: Future Work

# Summary
The future work items (idea based)

# Ideas
* Implement Zcoin’s Argon2-MTP for faster PoW verification
* Support Schnorr signatures (BIP-Schnorr) and other such multi-sig schemes
* Governance feature for forcing reorgs through majority vote
* Chaos Monkey 🐵
* On testnet, and even on mainnet, we will intentionally run some misbehaving nodes to continuously ensure the resiliency of the network. (The Netflix model)
* Better blocking: The permanent list is recorded on the blockchain so no doubt about who gets blocked. peer BNB self xfers BNB to own peer staking address to signal a BLOCK if someone is misbehaving including peer to block (peer pk + stake address) in signed memo to record on blockchain, majority of peers blocking peer puts that peer in the bad list for 48 hours (since first tx)
Schnorr/BLS Signatures
* TSS pre-computation optimisations 

# Status
Draft

# Specification

TODO

## Pre-requisites
None
## Details
None
# License