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

## Settelement price

Also known as feed price. 

## Collateral

## Collateral Ratio

## MCR

maintenance_collateral_ratio - Fed by the witneses

## MSSR

maximum_short_squeeze_ratio - Fed by the witness

## MSSP

## Premium

## Discount

## Call Order

## Limit Order

## Margin Call

- Feed price is below order call price.

## margin call territory

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
