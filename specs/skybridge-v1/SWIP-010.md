# [⏎](./readme.md) SWIP-010: Peer States

# Summary
Each “round” of the protocol lasts for 10 minutes. 
There are 6 of these per hour. Each “round” has a unique identifier round_num: floor(epoch_time_secs / (60 * 10))

# Motivation
All peer has an independent state based on the node life cycle.
Each peer selects a candidate tx which contains the most valid transfers (exact match or closest intersection) with correct reward distribution output in its view. must contain at least one transfer or round is skipped. TX includes participant peer table checksum, also verified.

# Status
Finalized

# Specification

1. Peer Discovery (mins. 1-2, *6 each hour) - send ping messages every 5s, build peer list, check stakes, make sure baddies are blocked.
Ping message: peer pk, state, stake tx hash, proof of tss share.
Correct ping triggers stake check (balance, must have been staked for at least 72 hrs and never moved) on each peer, each adds peer to its peer list. See Peer Staking.
Peers are continuously building active peer lists ordered by seniority (stake time)
2. Blackout (mins. 2-10, *6 each hour) - no ping messages sent, but still acknowledged until in one of the below states, only sync K/V for future rounds.
Clears the network chatter so the below can happen:
3. Sign List Building (mins. 2-4, *6 each hour)
All peers look at INCOMING address for new coins, put them in list, build txn.
Incoming coins must be in confirmed transactions and exist in unconfirmed K/V.
TX contains: tx inputs and outputs, reward distribution, refunds (multi-send).
Each peer broadcasts a sign proposal (max. one per peer): 
<sha512_256(tx_sign_doc), sha512_256(peer_id + tx_sign_doc), tss_proof>
Each peer: m[0] == own[0], m[1] == sha512_256(peer_id + our_sign_doc), ok(tss_proof)
Peers that pass are added to own signing set, which each peer will broadcast. Peers that broadcast signing sets that do not meet the threshold are ignored by other peers.
4. Sign List Voting (mins. 4-6, *6 each hour)*
The minimal subset of signing sets shared across at least t peers is selected by each peer (all must be visible by each other). Subsets are selected repeatedly until one that meets the full threshold is found. This solves an issue whereby two nodes each broadcast a signing list that is t in size but contain different peers.
Of this list, peers are ordered by seniority and peers with any non-unique tss_proofs are pruned from bottom-up to form the threshold set. Each peer in this list broadcasts a checksum of its own threshold set to prepare for signing. If any of these do not match up, outliers are blocked and we must wait until the next round; this one aborts.
5. TSS TX Signing (mins. 6-10, *6 each hour)
All peers pick the agreed upon unsigned tx start signing, then the signed tx is sent.
Any peers that send bad data or cause a timeout are temporarily blocked.
6. No New Requests (mins. 9-10, *6 each hour)
Peers do not accept requests for the next round at this time as it will start soon.
The peers list is wiped at this point as it will be rebuilt when the next round starts.

## Pre-requisites
None
## Details
None
# License