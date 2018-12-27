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

## CR - Collateral Ratio

The collateral ratio is calculated as:

(Collateral / Settlement Price) / Debt

## TCR - Target Collateral Ratio

## MCR - Maintenance Collateral Ratio

maintenance_collateral_ratio - Fed by the witneses

## MSSR - Maximum Short Squeeze Ratio

maximum_short_squeeze_ratio - Fed by the witness

## MSSP - Max Short Squeeze Price

`bitasset.current_feed.max_short_squeeze_price();`

## Shorter

Is the business of borrowing bitasset from the network with BTS as collateral and sell the asset in the market. Shorters are bitasset creators.

## Premium

## Discount

## Call Order

Borrow

## Limit Order

Sell

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
