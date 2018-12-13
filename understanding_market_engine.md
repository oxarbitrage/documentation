# Understanding the Bitshares Market Engine

## User Issued Assets

Bitshares is a token/asset factory. Once an account is created the user can inmediatly create and issue asset. Once an asset is created it can inmediatly trade with all the other assets in the bitshares blockchain unless specific whitelist/blacklist rules are configured. User Issued Assets(UIAs) trading price ...

## Market Pegged Assets

Users of the bitshares blockchain can also create Market Pegged Assets(MPAs) that can follow the value of a conventional asset like the USD or the SP500 . In this kind of asset the creator will whitelist one or several feed producers that will fed the system with external underlying asset prices. 

## Smartcoin

Smartcoin and MPA is the same.

## BitAsset

In bitshares the term smartcoin generally refers to specific MPAs that are owned by the committee and fed by the witnesses. Examples of this assets are USD, CNY, EUR and other system stable coins. They are also known by the "bit" prefix like bitUSD however the real name registered in the blockchain is just USD in that example.

## Global Settle - Black Swan

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

## Collateral Ratio

## MCR

maintenance_collateral_ratio - Fed by the witneses

## MSSR

maximum_short_squeeze_ratio - Fed by the witness

## MSSP

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

## Bitshares Market engine hardforks

## References

- http://docs.bitshares.org/bitshares/user/dex-margin-mechanics.html
- https://steemit.com/bitshares/@haruka/detailed-mechanism-of-global-settlement-black-swan-and-reviving-of-bitassets
- http://docs.bitshares.org/bitshares/user/dex-short.html
- http://www.bittwenty.com/bts_smartcoins.php
- http://bytemaster.github.io/article/2015/01/27/BitAssets-and-Black-Swan-Events/
