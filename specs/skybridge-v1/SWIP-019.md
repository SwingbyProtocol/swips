# SWIP-019 - A Standard Interface for Data Oracles

## Summary
A standard interface for integrating data oracles into the risk management of Swingby's Skybridge protocol.

## Abstract
The following standard allows the implementation of a data oracle within the Skybridge protocol for risk management features. 

It provides a generic integration model for price oracles to prevent over-staking and a safety mechanism to control for any price mismatch between two bridged assets on distinct networks.

## Motivation
This interface allows the Skybridge protocol to integrate an added risk layer to control external factors through one or multiple data oracle providers.

## Status

Draft.

## Specification

Most oracle providers require fees to **be paid in their native tokens** (e.g., BAND, LINK). These must be pre-loaded in the smart contract handling the logic for node operators to be spent.

The high-level steps for integrating an oracle are the following:

1. At a fixed frequency <img src="https://render.githubusercontent.com/render/math?math=$n$" alt="$n$"> or upon a user’s request, the Skybridge protocol sends a request to get the oracle provider's prices.
2. The request (or set of requests) asks for the latest prices of (1) SWINGBY and (2) all bridged assets on Skybridge (e.g., BTC, WBTC, tBTC, renBTC), (3) oracle tokens (e.g., LINK, BAND).
3. Swingby protocol must also retrieve the quantities of each asset staked in the pools, the quantities of SWINGBY tokens bound by validators, and the quantities held in the TSS for paying oracle services.

Once the quantities staked (in the float), the amount of SWINGBY bonded, the token quantities held by the TSS address used for paying oracle services, and the set of associated prices for all these assets are retrieved, these are used in various risk control mechanisms (described in the two subsections below).

While the price oracle is designed to be called at a periodic frequency by network validators, it is **not sufficient** to prevent the system from responding at a pace necessary for Skybridge to be constantly safe. This explains why **third parties** can also call the oracle directly **only if they bear the associated cost to retrieve the required set of price elements**. If any of the two risk controls described below were triggered, this third party would be rewarded SWINGBY tokens for an amount larger than the data request's associated cost.

### 1. Risk control to prevent over-staking

1. Based on the set information retrieved, the system calculates a parameter <img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> to determine the protocol's current staking status. 

<img src="https://render.githubusercontent.com/render/math?math=$k = \frac{(qtySwingbyBindedByChurnNodes * priceSwingby)}{ \sum_{i=1}^{n}qtyAsset_i * price_i}$" alt="$k = \frac{(qtySwingbyBindedByChurnNodes * priceSwingby)}{ \sum_{i=1}^{n}qtyAsset_i * price_i}$">

Note that <img src="https://render.githubusercontent.com/render/math?math=$n$" alt="$n$"> is the number of assets included in the float (e.g., BTC, WBTC) **AND** the number of different oracle tokens held in the smart contract address.

2. The <img src="https://render.githubusercontent.com/render/math?math=$l$" alt="$l$"> parameter is defined to determine limits on quantities that can be deposited in the protocol such as:

<img src="https://render.githubusercontent.com/render/math?math=$l = \frac{(qtySwingbyBindedByChurnNodes * priceSwingby)}{m} - \sum_{i=1}^{n}qtyAsset_i * price_i$" alt="$l = \frac{(qtySwingbyBindedByChurnNodes * priceSwingby)}{m} - \sum_{i=1}^{n}qtyAsset_i * price_i$">

with <img src="https://render.githubusercontent.com/render/math?math=$m$" alt="$m$"> being the lower boundary risk ratio required for new assets to be added on both legs of Skybridge

<img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> represents the protocol's current staking status, while <img src="https://render.githubusercontent.com/render/math?math=$l$" alt="$l$"> is used to determine the maximum liquidity provision for the protocol. 

#### Simple example of how the system could work 

1.5 would be the accepted deviation from the m parameter for a soft risk trigger (NOTE: <img src="https://render.githubusercontent.com/render/math?math=$m$" alt="$m$"> is not meant to be equal to 1).

25% and 50% would be proposed values to cap a single deposit to avoid overstaking.

| k < m | m <= k < m * 1.5 | k >= m * 1.5 |
| -------- | -------- | -------- |
| Liquidity deposits are suspended.  | It triggers a risk alert to the user. Single deposits are bounded by 25% * ```l```.| Normal status. A single deposit's value is bounded by 50% * ```l```|

### 2. Risk controls to prevent the unpegging of any set of two bridged assets

1. The system calculates a set of relative ratios for each asset pegged in a two-way fashion such as:

<img src="https://render.githubusercontent.com/render/math?math=$r = max(\frac {priceOriginalAsset}{pricePeggedAsset}, \frac {pricePeggedAsset}{priceOriginalAsset})$" alt="$r = max(\frac {priceOriginalAsset}{pricePeggedAsset}, \frac {pricePeggedAsset}{priceOriginalAsset})$">

3. If <img src="https://render.githubusercontent.com/render/math?math=$abs(1 - r)$" alt="$abs(1 - r)$"> is greater than <img src="https://render.githubusercontent.com/render/math?math=$f$" alt="$f$"> (i.e., the threshold to accept the price imbalance), **trigger a peg alert**. The proposed value <img src="https://render.githubusercontent.com/render/math?math=$f$" alt="$f$"> should be equal to the minimum fee to conduct a cross-chain swap between two bridged assets on Skybridge (e.g., WBTC --> BTC or BTC --> WBTC).

Once the system receives a peg alert, the following set of responses is triggered:

(1) liquidity provision is suspended (i.e., minting of LP token is suspended)
(2) redemptions for liquidity providers can continue based on the new price logic to adjust for the deviation in the peg between the two assets being bridged 
(3) user swaps are suspended

#### 1. Liquidity provision is suspended

When the system is in an "alert" state, liquidity provision is suspended. 

From the perspective of users, the UI would detect that the system is in an "alert" state.

However, if a user sends a swap transaction by interacting with the TSS address or the smart contract directly, the following logic is applied.

- For ERC20 token assets (e.g., WBTC), the transaction reverts.
- For BTC, the TSS nodes will refund it (minus a fee).

#### 2. Liquidity redemption continues based on the new price 

When the system is in an "alert" state, liquidity redemption is not suspended since it could reflect a long-lasting incident. Thus, a dedicated logic applies:

##### (A) Normal situation

Let's assume that one pool has 100 BTC and 100 WBTC.
1 BTC = 1 WBTC, and 200 LP tokens are outstanding.
Each LP token can be redeemed for 1 BTC or 1 WBTC.

##### (B) Collapse situation

Let's assume that one pool has 100 BTC and 100 WBTC. Once again, 200 LP tokens are outstanding. However, the price of WBTC collapses to 0.5 BTC. 

In this scenario, each LP token can be redeemed for 0.75 BTC or 1.5 WBTC.

#### General situation

Once the peg alert is in place, the price of the LP token can be expressed as:

<img src="https://render.githubusercontent.com/render/math?math=$price_{LP/Y} = \frac{quantity_X \times price_{X/Y}+ quantity_Y}{numberOfLPs}$" alt="$price_{LP/Y} = \frac{quantity_X \times price_{X/Y}+ quantity_Y}{numberOfLPs}$">

<img src="https://render.githubusercontent.com/render/math?math=$price_{LP/X} = \frac{quantity_X + quantity_Y \times price_{Y/X}}{numberOfLPs}$" alt="$price_{LP/X}  = \frac{quantity_X + quantity_Y \times price_{Y/X}}{numberOfLPs}$">

<img src="https://render.githubusercontent.com/render/math?math=$price_{LP/X}= \frac{price_{LP/Y}}{price_{X/Y}}$" alt="$price_{LP/X}= \frac{price_{LP/Y}}{price_{X/Y}}$">

#### 3. Swaps are suspended

When the system is in an "alert" state, swaps are suspended. From the perspective of users, the UI would display the current status of the protocol and prevent users from swapping assets.

However, if a user sends a swap transaction by interacting with the TSS address or the smart contract directly, the following logic is applied.

- For ERC20 token assets (e.g., WBTC), the transaction reverts.
- For BTC, the TSS nodes will refund it (minus a fee).

### 3. Exit of the risk checks

To exit these risk status, the protocol needs to receive a set of prices that allow it to calculate new risk metrics that fit within their respective normal limits.

- For the peg alert, the system requires the price difference between the two bridged assets to be lower than the specified <img src="https://render.githubusercontent.com/render/math?math=$f$" alt="$f$"> threshold. 
- For the over-staking protection, the protocol requires the system's <img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> parameter to return within appropriate limits (a multiplier of <img src="https://render.githubusercontent.com/render/math?math=$m$" alt="$m$">).

Like the oracle logic discussed before, either validators or a third-party actor can retrieve (from the oracle provider) the set of prices necessary to assess the current risk status of the protocol. If the new risk status of the protocol properly changes, the network rewards the third party for a reward higher than the cost of retrieving and posting the set of relevant prices from the oracle provider.

## Potential risks

### Front-running the execution of the oracle

**Problem:** There is an asymmetry of information between the person knowing an asset is/will become unpegged and the passive liquidity provider.

**Solution:** The protocol would allow anyone external to request updated prices from the oracle (e.g., an active LP can monitor for potential risks) by paying fees by themselves. If the system were effectively at risk, the party updating the prices into the risk logic would be rewarded in SWINGBY tokens.

Potentially, a penalty system could be applied to validators if they don't monitor deviation from the peg (transfer of risks from liquidity providers to validators)

### Oracle failure (e.g., delay, wrong prices)

**Problem:** The oracle may not be responsive or report the wrong prices.

**Potential solution:** The system must incorporate a safe switch where nodes can suspend the protocol. It could have a logic off-chain for validators to monitor whether the system is sufficiently collateralized. Alternatively, a majority of nodes could trigger a liquidity provision suspension.

### Lack of liquidity of an asset

**Problem:** some assets that are being bridged may not be liquid enough to have a price feed that is not manipulable.

**Potential solution**: Liquidity must be one of the criteria to consider the asset solution for what can be bridged. For SWINGBY tokens, while liquidity is expected to improve over time, it may be possible to start with a set of conservative parameters for <img src="https://render.githubusercontent.com/render/math?math=$m$" alt="$m$">.


### Lack of funds available in the TSS address for oracle payments

**Problem:** most external oracles require funds to have the native tokens of the provider.

**Potential solution:** The system can operate with a top-up account and a reputation system for each validator. Each validator that tops up improves its "reputation". If one doesn't add the "oracle asset", he would earn fewer fees. One validator is requested to top up the TSS address at a periodic frequency with a defined amount of protocol tokens for the oracle to be used (e.g., 100 BAND).


### Validators wishing to exit the protocol (unbinding SWINGBY tokens)

**Problem:** A Swingby validator may wish to resign from his position and unbound his SWINGBY token, which could leave the protocol undercollateralized.

**Potential solution**: The SWINGBY tokens being unbonded will be submitted to a time-lock, and the protocol would recalculate the future <img src="https://render.githubusercontent.com/render/math?math=$k$" alt="$k$"> parameter and associated deposit limits.

## License
Copyright (c) 2020 Swingby Labs Pte. LTD. The text content of this specification file is licensed under an MIT license found in the root of this repository.
