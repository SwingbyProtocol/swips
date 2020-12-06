# SWIP-020 - Swingby Liquidity Provider ("sbLP") Token Rate Mechanism

## Summary

A standard interface for handling the fee reward mechanism with a dedicated LP token: the Swingby Liquidity Provider ("sbLP") token.

## Abstract

The following standard allows the implementation of a LP token on the Ethereum blockchain that is used to (1) reward metanodes and (2) distribute swap fees to liquidity providers.

This SWIP describes the logic to collect transaction fees for network validators and liquidity providers through an exchange rate mechanism with sbLP tokens and the asset(s) provided in the pool.  

## Motivation

sbLP tokens allows an efficient distribution of swap fees from cross-chain transactions to both liquidity providers and network validators, with a dynamic exchange rate relative to the assets pooled. In comparison to multi-send transactions, it prevents distributing transactions explicitely, resulting in better cost execution from the perspective of gas for all network participants. This makes Skybridge suitable for small and large size transactions.

In addition, the issuance of a LP token on Ethereum allows its integration into existing protocols, leveraging the composability value proposition of the DeFi ecosystem.

## Status

Draft.

## Specification

LP tokens represent ownership for the assets held in a Skybridge pool, which is composed of **two assets meant to have the same value** (_"the bridged assets"_).

The first iteration of Skybridge focuses on a **BTC/WBTC pool** and its LP token is referred to as "sbLP-WBTC".

This document explains the process on how the sbLP exchange rate (vs. BTC) fluctuates, with a focus on the following cases: users moving assets (WBTC to BTC & BTC to WBTC), liquidity providers depositing/withdrawing assets, and metanode operators (i.e., the network validators).

Similar to other DeFi protocols (e.g., Compound, Aave), the price of the LP token increases over time to reflect  transaction fees being collected by liquidity pool holders. There are two categories of LP owners in Skybridge: LP owners and metanodes (that collect fees in LP tokens).

## Implementation

The implementation can be broken down in three different sections:

1. Users swapping assets (WBTC to BTC & BTC to WBTC)
2. Liquidity providers depositing assets
3. Liquidity providers withdrawing assets

### 0. Terminologies

To understand the document, the most widely used variables are listed in this introductory section.

#### 0.1 Fees metrics
- <img src="https://render.githubusercontent.com/render/math?math=%24amountReceived_%7BLP%7D%24"> : amount received by all liquidity providers
- <img src="https://render.githubusercontent.com/render/math?math=%24amountReceived_%7Bnodes%7D%24"> : amount received by all metanodes based on their stake in Swingby tokens.
- <img src="https://render.githubusercontent.com/render/math?math=%24amountReceived_%7Ball%7D%24">: total fees collected by all parties in Skybridge.
- <img src="https://render.githubusercontent.com/render/math?math=%24k%24">: percentage of the <img src="https://render.githubusercontent.com/render/math?math=%24amountReceived_%7Ball%7D%24"> allocated to existing LPs.

#### 0.2 Exchange rates and numbers of LP tokens 

- <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%2FBTC%7D">: exchange rate between LP tokens and BTC
- <img src="https://render.githubusercontent.com/render/math?math=numberOfTokens_%7BLP%7D">: number of LP tokens circulating (excluding burnt and unclaimed tokens)
- <img src="https://render.githubusercontent.com/render/math?math=newMintedTokens_%7BLP%7D">: number of LP tokens that are being minted to metanodes. They increase the number of unclaimed LP tokens.
- <img src="https://render.githubusercontent.com/render/math?math=numberOfUnclaimedTokens_%7BLP%7D">: number of LP tokens that will be issued once metanodes claim them

#### 0.3 Quantities provided in pools and ratios
- <img src="https://render.githubusercontent.com/render/math?math=quantity_%7BWBTC%7D">: the number of BTC held in the float for the 2-asset pool
- <img src="https://render.githubusercontent.com/render/math?math=quantity_%7BBTC%7D">: the number of WBTC held in the float for the 2-asset pool
- <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BBTC%7D">: the ratio of the quantity of BTC held relative to all quantity held in the 2-asset pool
- <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BWBTC%7D">: the ratio of the quantity of WBTC held relative to all quantity held in the 2-asset pool

#### 0.4 Swingby token-related metrics

- <img src="https://render.githubusercontent.com/render/math?math=metanodeStake_i">: the stake in Swingby tokens for a metanode
- <img src="https://render.githubusercontent.com/render/math?math=totalSwingbyToken">: the total number of Swingby tokens locked in all metanodes involved the protocol (i.e., that have met the required thresholds)


### 1. Swap transactions

When a swap is handled, part of the transaction fee is allocated to the smart contract and defined in the contract through the use of a variable called $amountReceived_{LP}$ (expressed in BTC for sbLP-BTC).

For any given transaction moving BTC to WBTC, the total fee amount can be calculated such as:

<!-- $amountReceived_{all}  = amountReceived_{LP} / k$ --> <img src="https://render.githubusercontent.com/render/math?math=amountReceived_%7Ball%7D%20%20%3D%20amountReceived_%7BLP%7D%20%2F%20k"> 

The value for _k_  is handled at the smart contract level and can be adjusted by the TSS address.

As discussed previously, node operators are also rewarded by collecting fees. This fee collection mechanism is achieved by minting LP tokens to metanode operators; allowing them to become liquidity providers and to benefit from the compounding of their stake.

Once the amount is paid to LPs (recorded as <!-- $amountReceived_{LP}$ --> <img src="https://render.githubusercontent.com/render/math?math=amountReceived_%7BLP%7D">), the <!-- $exchangeRate_{LP}$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%7D"> must be re-calculated such as:

<!-- $exchangeRate_{LP/BTC} = (newQuantity_{BTC} + newQuantity_{WBTC})/(numberOfTokens{LP} + numberOfUnclaimedTokens_{LP})$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%2FBTC%7D%20%3D%20(newQuantity_%7BBTC%7D%20%2B%20newQuantity_%7BWBTC%7D)%2F(numberOfTokens%7BLP%7D%20%2B%20numberOfUnclaimedTokens_%7BLP%7D)">

with (1) <!-- $newQuantity_{BTC} = initialQuantity_{BTC} + quantitySwapped_{BTC} + amountReceived_{LP}$ --> <img src="https://render.githubusercontent.com/render/math?math=newQuantity_%7BBTC%7D%20%3D%20initialQuantity_%7BBTC%7D%20%2B%20quantitySwapped_%7BBTC%7D%20%2B%20amountReceived_%7BLP%7D">
and (2) <!-- $newQuantity_{WBTC} = initialQuantity_{WBTC} - quantitySwapped_{WBTC}$ --> <img src="https://render.githubusercontent.com/render/math?math=newQuantity_%7BWBTC%7D%20%3D%20initialQuantity_%7BWBTC%7D%20-%20quantitySwapped_%7BWBTC%7D">

Since <!-- $newQuantity_{BTC} + newQuantity_{WBTC} > initialQuantity{BTC} + initialQuantity{WBTC}$ --> <img src="https://render.githubusercontent.com/render/math?math=newQuantity_%7BBTC%7D%20%2B%20newQuantity_%7BWBTC%7D%20%3E%20initialQuantity%7BBTC%7D%20%2B%20initialQuantity%7BWBTC%7D">, the <!-- $exchangeRate_{LP/BTC}$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%2FBTC%7D"> **increases after every new swap**.

Afterward, the number of new LP tokens that must be minted for node owners is calculated as:

<!-- $newMintedTokens_{LP} = \frac{amountReceived_{all} * (1 - k)}{exchangeRate_{LP/BTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=newMintedTokens_%7BLP%7D%20%3D%20%5Cfrac%7BamountReceived_%7Ball%7D%20*%20(1%20-%20k)%7D%7BexchangeRate_%7BLP%2FBTC%7D%7D">

Each _i_ metanode has a stake in Swingby tokens (defined as <!-- $metanodeStake_i$ --> <img src="https://render.githubusercontent.com/render/math?math=metanodeStake_i">).

Thereby, the number of unclaimed LP tokens increases by <!-- $\frac{newMintedTokens_{LP} * metanodeStake_i}{totalSwingbyStaked}$ --> <img src="https://render.githubusercontent.com/render/math?math=%5Cfrac%7BnewMintedTokens_%7BLP%7D%20*%20metanodeStake_i%7D%7BtotalSwingbyStaked%7D">.

while,

<!-- $numberOfUnclaimedTokens_{LP} = newMintedTokens_{LP} +  numberOfUnclaimedTokens_{LP}$ --> <img src="https://render.githubusercontent.com/render/math?math=numberOfUnclaimedTokens_%7BLP%7D%20%3D%20newMintedTokens_%7BLP%7D%20%2B%20%20numberOfUnclaimedTokens_%7BLP%7D">

At any point of time, metanodes can claim these LP tokens, resetting the number of tokens that have not been claimed.

### 2. Liquidity deposit

For liquidity deposits, a fee can apply if there is an imbalance in the pool quantities, as defined by its ratio relative to the total quantities held in a two-asset pool.

The current ratios of a BTC/WBTC pool are defined for both assets, such as:

<!-- $ratio_{BTC} = \frac{quantity_{BTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

<!-- $ratio_{WBTC} = \frac{quantity_{WBTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BWBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BWBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

The associated fees are as follows:

| ratio < 1/3 | 1/3 < ratio < 2/3 | ratio > 2/3 |
| -------- | -------- | -------- |
| 0.00%     | 0.10%     | 0.20%     |

However, these ratios are calculated based on the notionals deposited.

<!-- $futureRatio_{BTC} = \frac{quantity_{BTC} + quantityDeposited_{BTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=futureRatio_%7BBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BBTC%7D%20%2B%20quantityDeposited_%7BBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

<!-- $futureRatio_{BTC} = \frac{quantity_{WBTC} + quantityDeposited_{BTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=futureRatio_%7BBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BWBTC%7D%20%2B%20quantityDeposited_%7BBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

with <!-- $quantityDeposited$ --> <img src="https://render.githubusercontent.com/render/math?math=quantityDeposited"> being the notional a user wishes to deposit.

**Adding liquidity can change the LP exchange rate (vs. BTC)**. If so, these deposit fees are collected exclusively by metanode operators. Thus, <!-- $numberOfTokens_{LP}$ --> <img src="https://render.githubusercontent.com/render/math?math=numberOfTokens_%7BLP%7D"> increases accordingly based on the fee collected (if any).

#### BTC deposit

1. The user deposits BTC using a special tag placed in the K/V store that tells the system it is a float deposit. The swap is processed as normal by the nodes but the destination token becomes a LP token.
2. The parameter <!-- $exchangeRate_{LP/BTC}$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%2FBTC%7D"> is read by the contract.
3. The contract mints a quantity of LP token (1-1 representation) based on <!-- $exchangeRate$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate"> and <!-- $quantityDeposited$ --> <img src="https://render.githubusercontent.com/render/math?math=quantityDeposited"> (as supplied by the nodes) and sends the minted LP token to the user on Ethereum.

#### WBTC deposit

1. User sends the WBTC to the contract on Ethereum.
2. The contract mints a quantity of the LP token (1-1 representation) based on <!-- $exchangeRate_{LP/BTC}$ --> <img src="https://render.githubusercontent.com/render/math?math=exchangeRate_%7BLP%2FBTC%7D"> and $quantityDeposited$ (as supplied by the nodes) and sends the minted LP token to the user on Ethereum.
3. The contract credits the swap pool with the deposited WBTC.

### 3. Liquidity removal

When a user wishes to withdraw liquidity to one side of the pool, the LP token must be burnt. The $exchangeRate_{LP/BTC}$ is recorded part of the user's request.

**Removing liquidity can change the LP exchange rate (vs. BTC)**. Similar to liquidity deposits, a fee would be collected exclusively by metanode operators if there is any imbalance in the quantities held in two sides of the pools.

The current ratios of the pool are defined for both assets, such as:

<!-- $ratio_{BTC} = \frac{quantity_{BTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

<!-- $ratio_{WBTC} = \frac{quantity_{WBTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=ratio_%7BWBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BWBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

The associated fees are as follows:

| ratio < 1/3 | 1/3 < ratio < 2/3 | ratio > 2/3 |
| -------- | -------- | -------- |
| 0.00%     | **0.00%**     | 0.20%     |

The ratio is calculated based on the notional to deposit.

Thus,

<!-- $futureRatio_{BTC} = \frac{quantity_{BTC} + quantityWithdrawn_{BTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=futureRatio_%7BBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BBTC%7D%20%2B%20quantityWithdrawn_%7BBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

<!-- $futureRatio_{WBTC} = \frac{quantity_{WBTC} + quantityWithdrawn_{WBTC}}{quantity_{BTC + WBTC}}$ --> <img src="https://render.githubusercontent.com/render/math?math=futureRatio_%7BWBTC%7D%20%3D%20%5Cfrac%7Bquantity_%7BWBTC%7D%20%2B%20quantityWithdrawn_%7BWBTC%7D%7D%7Bquantity_%7BBTC%20%2B%20WBTC%7D%7D">

with <!-- $quantityWithdrawn$ --> <img src="https://render.githubusercontent.com/render/math?math=quantityWithdrawn"> being the notional a user wishes to redeem.


#### 3.1 BTC redemption

1. User sends the sbLP token to the TSS address on Ethereum.
2. The system sends the BTC to the user on the Bitcoin blockchain.

#### 3.2 WBTC redemption

1. User sends the sbLP token to the swap contract on Ethereum.
2. The swap contract sends the WBTC to the TSS address on Ethereum.
3. The TSS address burns the received LP token.

## License

Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
