# Understanding the Bitshares Market Engine

## User Issued Assets

Bitshares is a token/asset factory. Once an account is created the user can inmediatly create and issue asset. Once an asset is created it can inmediatly trade with all the other assets in the bitshares blockchain unless specific whitelist/blacklist rules are configured. User Issued Assets(UIAs) trading price ...

## Market Pegged Assets

Users of the bitshares blockchain can also create Market Pegged Assets(MPAs) that can follow the value of a conventional asset like the USD or the SP500 . In this kind of asset the creator will whitelist one or several feed producers that will fed the system with external underlying asset prices. 

## Smartcoin

Smartcoin and MPA is the same.

## BitAsset

In bitshares the term smartcoin generally refers to specific MPAs that are owned by the committee and fed by the witnesses. Examples of this assets are USD, CNY, EUR and other system stable coins. They are also known by the "bit" prefix like bitUSD however the real name registered in the blockchain is just USD in that example.

## Feeds

In a market pegged asset the feed is how the witnesses valuate asset, at was price, using external sources. Witnesses collect market prices from different sources, apply some algorithm and feed the bitshares network. Effective feed in bitshares is the medium of all the witness provided feeds. 

Feeds haves expiration.

## Feed manipulation

To avoid black swan events the witnesses can agree on changing the algorithm they use to feed the network. Witnesses fed the network in general by getting a medium from external sources however the sources they use and the algo they implement to calculate the final value sent to bitshares is flexible among certain standards.

For the witnesses to make drastic changes on their algos they need to be approved from the stakeholders. This is the case of controlvertial Bsip40 where experminetation on the algo was introduced and executed for a limited period of time where the bsip was active(voted/approved by stakeholders)

## Settelement price

Also known as medium feed price. 

## Collateral

In general it is the amount of BTS(can be other) that is backing up a position. Collateral in position depends on MCR. Borrow/Call orders should have at least bitasset*MCR in BTS to be created.

## Debt

When borrowing the debt is the amount of bitasset the network is lending you, for example you will want to borrow 10 bitusd and collateral at 100 BTS. Position debt and collateral are defined by the call order parameters.

## CR - Collateral Ratio

The collateral ratio is calculated as:

(Collateral / Settlement Price) / Debt

## Call Price

Is the price at what the position will be margin called. Call price is independent from feed and it is calculated as:

Call price = Collateral / (Debt * MCR)

Open positions can increase call price by adding more collateral to their open positions to avoid margin call.

## TCR - Target Collateral Ratio

Is defined by the borrower as a call order parameter.

## MCR - Maintenance Collateral Ratio

maintenance_collateral_ratio - Fed by the witneses

## MSSR - Maximum Short Squeeze Ratio

maximum_short_squeeze_ratio - Fed by the witness

## MSSP - Max Short Squeeze Price

`bitasset.current_feed.max_short_squeeze_price();`

## Shorter

Is the business of borrowing bitasset from the network with BTS as collateral and sell the asset in the market. Shorters are bitasset creators.

## Premium

If bitAsset value > Asset value the bitasset is in premium. A holder will need to pay more real USD to get 1 bitUSD. https://bitsharestalk.org/index.php?topic=27701.msg326859#msg326859

## Discount

If bitAsset Value < Asset value then the bitasset is in discount. A Holder will pay less real USD to get 1 bitUSD.

## Call Order

In bitshares a call order is a blockchain operation(operation type: `3`, `call_order_update`). The action of creating or updating a call order is also called `borrow`. Any bitasset can be borrowed from the network if participant put enough collateral into the position, then the trader will usually want to sell the asset short for a profit and providing bitasset liquidity. The borrowing part is always done with a call order.

## Limit Order

The blockchain operation `limit_order_create`(operation type: `1`) is used to sell your asset for another in the market. Participant will execute the operation with a minimum amount to receive for the selling asset and the market engine will match with the buyer only if the `min_to_receive` field is satisfied.

Partiipants can also cancel limit orders with blockchain operation `limit_order_cancel`(operation type: 2). When cancelled the order will lot be available in the order book anymore.

Selling is also the next logical step for when borrowing. When borrowing bitasset the trader will offer it in the market, trader at this moment is short the bitasset.

## Margin Call

A margin call is the market forcing you to sell your collateral in order to buy enough USD to close your position.
- Feed price is below order call price.

## Margin Call Territory

- When borrowing bitasset from the network and collateral is not enough to support current bitasset price(determinated by feeds), call order will be placed in margin call territory. This means a margin call will be produced when a match is available.

## Global Settle - Black Swan


## Market API Calls

- get_limit_orders
- get_account_limit_orders
- get_call_orders
- get_settle_orders
- get_margin_positions
- get_collateral_bids
- get_ticker
- get_24_volume
- get_order_book
- get_top_markets
- get_trade_history
- get_trade_history_by_sequence

## Bitshares Market engine BSIPs

Market engine changes are protocol changes and by this they need bsip creation and approvment fropm stakeholders.

### BSIP 30

- [BSIP 30 - Always Allow Increasing Collateral Ratio If Debt Not Increased](https://github.com/bitshares/bsips/blob/master/bsip-0030.md)

Implementation:

https://github.com/bitshares/bitshares-core/pull/827

### BSIP 31 - 34

- [BSIP 31 - Update Short Position's Margin Call Price After Partially Called Or Settled](https://github.com/bitshares/bsips/blob/master/bsip-0031.md)
- [BSIP 32 -  Always Match Orders At Maker Price](https://github.com/bitshares/bsips/blob/master/bsip-0032.md)
- [BSIP 33 - Maker Orders With Better Prices Take Precedence](https://github.com/bitshares/bsips/blob/master/bsip-0033.md)
- [BSIP 34 - Always Trigger Margin Call When Call Price Above Or At Price Feed](https://github.com/bitshares/bsips/blob/master/bsip-0034.md)

Implementration:

https://github.com/bitshares/bitshares-core/pull/829

## Bitshares Market engine Unit Tests

### Market Tests

- `issue_338_etc` - Reproduce bitshares-core issue #338 #343 #453 #606 #625 #649
- `hardfork_core_338_test` - Fixed bitshares-core issue #338 #343 #606 #625 #649
- `hardfork_core_453_test` - Fixed bitshares-core issue #453: multiple limit order filling issue
- `hardfork_core_625_big_limit_order_test` - Tests (big) limit order matching logic after #625 got fixed
- `hard_fork_453_cross_test` - Fixed bitshares-core issue #453 #606: multiple order matching without black swan, multiple bitassets
- `hard_fork_338_cross_test` - Fixed bitshares-core issue #338 #453 #606: multiple order matching with black swan
- `hard_fork_649_cross_test` - Fixed bitshares-core issue #649: Black swan detection fetch call order by call_price but not collateral ratio
- `hard_fork_343_cross_test` - Fixed bitshares-core issue #343: change sorting of call orders when matching against limit order
- `target_cr_test_limit_call` - BSIP38 "target_collateral_ratio" test: matching a taker limit order with multiple maker call orders
- `target_cr_test_call_limit` - BSIP38 "target_collateral_ratio" test: matching a maker limit order with multiple taker call orders
- `mcr_bug_increase_before1270` - 
- `mcr_bug_increase_after1270` - 
- `mcr_bug_decrease_before1270` - 
- `mcr_bug_decrease_after1270` - 
- `mcr_bug_cross1270` - 
- `hardfork_core_338_test_after_hf1270` - 
- `hardfork_core_453_test_after_hf1270` - 
- `hardfork_core_625_big_limit_order_test_after_hf1270` - 
- `target_cr_test_limit_call_after_hf1270` - 
- `target_cr_test_call_limit_after_hf1270` - 

### Swan Tests

### BitAsset Tests

### Market Rounding Tests


## References

- http://docs.bitshares.org/bitshares/user/dex-margin-mechanics.html
- https://steemit.com/bitshares/@haruka/detailed-mechanism-of-global-settlement-black-swan-and-reviving-of-bitassets
- http://docs.bitshares.org/bitshares/user/dex-short.html
- http://www.bittwenty.com/bts_smartcoins.php
- http://bytemaster.github.io/article/2015/01/27/BitAssets-and-Black-Swan-Events/
- https://github.com/bitshares/bitshares-core/wiki/resolvable-bitassets - check if this feature is available
- https://github.com/MichelSantos/share201811/blob/master/gs_likelihood.md
