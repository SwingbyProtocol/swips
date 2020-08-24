# [⏎](./readme.md) SWIP-009: Peer Blocking

# Summary
Each peer has two tables: unconfirmed (always synced), confirmed (not synced).
Newly recorded swaps are put in unconfirmed and are synced (w/ validation).
Peers watch for incoming txns, and when seen, a record moves to confirmed if valid.
Signing sets are built using the confirmed table, always checked by each peer.
Entries in both tables are namespaced by round_num so we only ever keep the pairs for the current and next rounds.
There is some memory-hard PoW attached to each new unconfirmed entry for extra GPU resistant protection.

# Motivation
There are fully synced nodes is basically doesn't have a full state of the blockchain.
That means all nodes just collect the tables of the incoming swaps. and recent outcoming transactions.
This SWIP defines these K/V table specifications.

# Status
Finalized

# Specification

## K/V Lifecycle Steps
1. K/V is proposed via node RPC. Each peer verifies validity and argon2d_PoW before adding to unconfirmed for the given round. PoW checks run in a single thread, 2.cannot be parallelized, and are queued.
2. Other nodes sync the new unconfirmed entries, validate, and verify PoW. Existing keys cannot be changed.
3. When the inbound transaction is seen, the K/V pair is looked up and used to determine the destination address. The pair is moved to confirmed.

## Pre-requisites
None
## Details
None
# License