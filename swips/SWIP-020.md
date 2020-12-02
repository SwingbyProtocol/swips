# SWIP-020 - Swingby Liquidity Provider (SWLP) Token Rate Mechanism

TODO: change system to adjust variable names
TODO: add ratio of two sides of a pool <> fees relationship for withdrawal/deposit

## Summary

A standard interface for handling the fee reward mechanism with a dedicated LP token: the Swingby Liquidity Provider ("SWLP") token.

## Abstract

The following standard allows the implementation of a LP token on the Ethereum blockchain that is used to (1) reward metanodes and (2) distribute swap fees to liquidity providers.

This SWIP describes the logic to collect transaction fees for both network validators and liquidity providers through an exchange rate mechanism with SWLP tokens.  

## Motivation

SWLP tokens allows an efficient distribution of swap fees from cross-chain transactions to both liquidity providers and network validators, with a dynamic exchange rate relative to the assets pooled. It prevents distributing transactions _explicitely_, resulting in better cost execution from the perspective of gas for all network participants.

## Status

Draft.

## Specification

LP tokens represent ownership for the assets held in a Skybridge pool, which is composed of **two assets meant to have the same value** (_"the bridged assets"_).

The first iteration of Skybridge focuses on a **BTC/WBTC pool** and its LP token is referred to as "SWLP-WBTC".

This document explains the process on how the SWLP exchange rate (vs. BTC) fluctuates, with a focus on the following cases: users moving assets (WBTC to BTC & BTC to WBTC) and liquidity providers depositing/withdrawing assets.

Similar to other DeFi protocols like Compound, the price of the LP token increases over time to reflect all transaction fees being collected by liquidity pool holders. There are two categories of LP owners in Skybridge: LP owners and node owners (who collect fees in LP tokens_.

## Implementation

The implementation can be broken down in three different sections:
1. Users swapping assets (WBTC to BTC & BTC to WBTC)
2. Liquidity providers depositing assets
3. Liquidity providers withdrawing assets

### Terminologies

TODO: explain the set of variables

### 1. Swap transactions

When a swap is handled, part of the transaction fee is allocated to the smart contract and defined in the contract through the use of a field called <img src="https://render.githubusercontent.com/render/math?math=$amountReceivedByLP$" alt="$amountReceivedByLP$"> (in BTC).

The total amount can be calculated such as:

<img src="https://render.githubusercontent.com/render/math?math=$amountReceived = amountReceivedByLP / k$" alt="$amountReceived = amountReceivedByLP / k$">

<img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> is the percentage of the total swap fee allocated to existing LPs.

The value for <img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> is handled at the smart contract level and can be adjusted by the TSS address.

As discussed previously, node operators are also rewarded by collecting fees, i.e., it is achieved by receiving LP tokens. These allow the node owners to become liquidity providers and to benefit from the compounding of their stake.

Once the amount is paid to LPs (recorded as <img src="https://render.githubusercontent.com/render/math?math=$amountReceivedByLP$" alt="$amountReceivedByLP$">), the <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> must be re-calculated such as:

<img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate = (newQuantityBTC + newQuantityWBTC)/(numberOfLPTokens + numberOfUnclaimedLPTokens)$" alt="$currentExchangeRate = (newQuantityBTC + newQuantityWBTC)/(numberOfLPTokens + numberOfUnclaimedLPTokens)$">

with (1) $newQuantityBTC = initialQuantityBTC + quantityBTCSwapped + amountReceivedByLP$
and (2) $newQuantityWBTC = initialQuantityWBTC - quantityWBTCSwapped$

Since <img src="https://render.githubusercontent.com/render/math?math=$newQuantityBTC + newQuantityWBTC > initialQuantityBTC + initialQuantityWBTC$" alt="$newQuantityBTC + newQuantityWBTC > initialQuantityBTC + initialQuantityWBTC$">, the new <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> **increases after every new swap**.

Afterward, the number of new LP tokens that must be minted for node owners is calculated as:

<img src="https://render.githubusercontent.com/render/math?math=$newMintedLPTokens = (amountReceived * (1 - k)) / currentExchangeRate$" alt="$newMintedLPTokens = (amountReceived * (1 - k)) / currentExchangeRate$">

Each validator's existing number unclaimed LP tokens is increased by <img src="https://render.githubusercontent.com/render/math?math=$newMintedLPTokens * stakeOfNode / totalSwingbyStaked$" alt="$newMintedLPTokens * stakeOfNode / totalSwingbyStaked$">.

<img src="https://render.githubusercontent.com/render/math?math=$numberOfUnclaimedLPTokens = newMintedLPTokens +  numberOfUnclaimedLPTokens$" alt="$numberOfUnclaimedLPTokens = newMintedLPTokens +  numberOfUnclaimedLPTokens$">

Node owners can claim these LP tokens at any point of time, resetting their dedicated number of tokens that hasn't been claimed.

### 2. Liquidity deposit

**Adding liquidity changes the LP exchange rate (vs. BTC) since a fee is collected by metanode operators**. Furthermore, <img src="https://render.githubusercontent.com/render/math?math=$numberOfLPTokens$" alt="$numberOfLPTokens$"> increases accordingly based on the quantity newly minted.

#### BTC deposit

1. The user deposits BTC using a special tag placed in the KV store that tells the system it is a float deposit. The swap is processed as normal by the nodes but the destination token becomes LP.
2. The parameter <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> is read by the contract.
3. The contract mints a quantity of LP token (1-1 representation) based on <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> and <img src="https://render.githubusercontent.com/render/math?math=$amountDeposited$" alt="$amountDeposited$"> (as supplied by the nodes) and sends the minted LP token to the user on Ethereum.

#### WBTC (and other ERC-20 BTC tokens) deposit

1. User sends the WBTC to the contract on Ethereum.
2. The contract mints a quantity of the LP token (1-1 representation) based on <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> and <img src="https://render.githubusercontent.com/render/math?math=$amountDeposited$" alt="$amountDeposited$"> (as supplied by the nodes) and sends the minted LP token to the user on Ethereum.
3. The contract credits the swap pool with the deposited WBTC.

### 3. Liquidity removal

When a user wishes to withdraw liquidity to one side of the pool, he must burn the LP token, indicate the asset he wishes to redeem his LP token to, while the <img src="https://render.githubusercontent.com/render/math?math=$currentExchangeRate$" alt="$currentExchangeRate$"> is recorded part of the user's request.

**Removing liquidity can change the LP exchange rate (vs. BTC) since a fee is collected by metanode operators if there is any imbalance in the quantities held in two sides of the pools**.

#### 3.1 BTC redemption

1. User sends the SWLP token to the TSS address on Ethereum.
2. The system sends the BTC to the user on the Bitcoin blockchain.

#### 3.2 WBTC (and other ERC-20 BTC tokens) redemption

1. User sends the SWLP token to the swap contract on Ethereum.
2. The swap contract sends the WBTC to the TSS address on Ethereum.
3. The TSS address burns the received LP token.

## License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
