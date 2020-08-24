# [⏎](./readme.md) SWIP-011: KeyGen Phase

# Summary
Each peer has the same “keygen until” time set in config. Keygen runs repeatedly aligned to 5 minute intervals until this time is reached. The in/out addresses will change until a certain block when it becomes “locked in” and permanent.
The mainnet network needs to spend some time repeatedly generating the TSS shares for about 1 week to prove that no individuals know the full private key.

# Motivation
All peer need to make a tss signer set indipendently, That state means "KeyGen state".
Each node should have derived private key and secret share. 
This SWIP defines of the way that to make a TSS group and it generates a keypair of single TSS signer set.

# Status
Finalized

# Specification

TODO

## Pre-requisites
None
## Details
None
# License