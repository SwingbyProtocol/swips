# [⏎](./readme.md) SWIP-002: Swap Process for BTC → BTC.B

# Summary

Definition of Skybridge Swap flows of BTC → BTC.B
This SWIP defines the process flow of BTC → BTC.B

# Motivation

Swingby Node has a logic of coin swap between cross-chain.
This logic is optimized to receive and process BTC transactions.

# Status

Deployed.

# Specification

Following the steps is Skybridge Swap flows the case of BTC → BTC.B

1. User tells Swingby the amount and dest. address on website
2. Swingby tells user the real amount to send
3. Swingby peer validates BNB address and params, saves amount and address in K/V
4. User confirms address stored in K/V from a different peer (UI asks another peer)
5. When peers see the incoming TX, K/V pair is moved to confirmed table. The incoming TX must meet a confirmation requirement (e.g. 3 on BTC, 1 on BNB)
6. IF not found, coins are returned, otherwise swap happens.

### Diagram

![](./images/swap-lifecycle.png)

## Pre-requisites

None

## Details

### Attack Vectors

#### Front-Running

Someone can watch BTC mempool and catch a tx that was erroneously entered or not synced yet by telling the network about it to trick it into sending to the attacker instead of the intended destination address.

For the purposes of solving this problem, unfortunately BTC does not support tx memos. So, we code the destination address into the amount.

For example, I want to swap 0.12341234 BTC, but that is not what I send:

##### The formula to compute the “send” amount is as follows:
  `floor(in_amt)-rs(sha512_256(nonce + dest_addr + floor_amt_coin + nxt_round_no) % 0x400)`

  nonce is included because a hash must generated that begins with two zero bytes (CPU PoW)
  
  `floor_amt_coin` and `send_amt_coin` represent the `<amount, coin>` pair in string form.
  
  floor(): changes amount to end in 000 satoshis, e.g. `0.12341234 -> 0.12341000`.
  
  rs(): rejection sample: ensures randomness. `abs(sample)` must be < 1000 or the hash function is run again.

These hashes cannot be pre-computed as they include the amount and round_num.

Why SHA512/256: It is more efficient on 64-bit architectures and has length extension safety.

1000 satoshis is $0.08, so worst case swingby fee will be less max $0.08 with BTC @ $8420

##### Swingby tells user to send: e.g. 0.12340877 (the “send” amount)
The Client sends to the peer: `<send_amount_coin, dest_address, nonce, argon2d_PoW(sha512256_hash)>`

Peer can easily compute the sha512_256 and check leading zeros & equality to stop spam.

K/V namespace: keyed on `round_num`; old rounds are simply discarded in memory.

K/V pair: `send_amount_coin → <dest_address, nonce, argon2_PoW>`

If a K/V slot is taken in any visible round namespaced maps, the peer rejects their request this round (user is asked to change amount).

Namespaces exist for variable time depending on blockchain (BNB:-1 round, BTC:-6 rounds)

K/V sync/validation: other peers will only sync unconfirmed pairs that pass all validations, and only then confirm keys which exist on-chain.

When a peer sees incoming tx for that amount, it can look up in K/V and do the `amt + (sha512_256(nonce + dest_addr + floor_amount_coin + round_num) % 0x400)` to restore the original amount of `0.12341000` .. if it ends in 000, it's all good!

##### Why it works:
If someone front-runs with their own receiving address, `sha512_256(nonce + dest_address + floor_amount_coin + round_num) % 0x400` will be different: the amount will not end in 000 so that K/V pair will be ignored by other nodes, even if a malicious node were to broadcast it.

At worst a 1/1000 chance of a bad person snatching someone’s money using front-running, and pre-computing the hashes is completely impractical because hashes change completely every round and due to the PoW (sha512/256 = CPU hard, argon2d = memory hard).

# License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
