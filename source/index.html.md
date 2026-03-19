---
title: ALP.COM API Reference

language_tabs:
  - json: JSON
  - python: Python

toc_footers:
  - <a href='https://alp.com/en/register/'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to ALP.COM API v3 docs!

Service ALP.COM provides open API for trading operations and broadcasting of all trading events.

HTTP API endpoint available on [https://alp.com/api/v3/](https://alp.com/api/v3/).

For users currently using API v1, we strongly recommend transitioning to API v3, as API v1 will soon be deprecated and no longer available.

Available clients:
- Python: [https://pypi.org/project/alpcom-api/](https://pypi.org/project/alpcom-api/)

```python
# pip3 install alpcom-api

from alpcom_api import factories, cache

public_api = factories.get_public_api()
private_api = factories.get_private_api(
    key='** Your private_api Key **',
    secret='** Your private_api Secret **',
    # Optionally use cache to store temp token
    # otherwise it will be generated every time an instance of HTTPClient gets created
    # cache=cache.FileCache('/path/to/token.txt')
)
```

# Constants

### Wallet Types

- `SPOT`: Spot Wallet
- `MARGIN_CROSS`: Cross Margin Wallet
- `MARGIN_ISOLATED`: Isolated Margin Wallet
- `FUNDING`: Funding Wallet

### Order Sides

- `BUY`: Buy Order
- `SELL`: Sell Order

### Order Types

- `MARKET`: Market Order
- `LIMIT`: Limit Order
- `STOP_LIMIT`: Stop-Limit Order
- `STOP_MARKET`: Stop-Market Order

### Order Status

- `PENDING`: Pending Order
- `OPEN`: Open Order
- `FILLED`: Filled Order
- `CANCELLED`: Cancelled Order
- `PARTIAL_FILLED`: Partially Filled Order
- `PARTIAL_CANCELED`: Partially Cancelled Order
- `FAILED`: Failed Order

### Order Time in Force

- `GTC`: Good 'til Cancelled
- `IOC`: Immediate or Cancel
- `FOK`: Fill or Kill

### Order Stop Operators

- `GTE`: Greater Than or Equal To
- `LTE`: Less Than or Equal To

### Chart Intervals

- `1`: 1 minute
- `5`: 5 minutes
- `15`: 15 minutes
- `30`: 30 minutes
- `60`: 1 hour
- `240`: 4 hours
- `1D`: 1 day

### Deposit Status

- `PENDING`: Pending Deposit
- `MODERATION`: In Moderation
- `CONFIRMED`: Confirmed Deposit
- `REJECTED`: Rejected Deposit

### Withdrawal Status

- `NEW`: New Withdrawal
- `CONFIRMED`: Confirmed Withdrawal
- `VERIFIED`: Verified Withdrawal
- `MODERATION`: Withdrawal in Moderation
- `APPROVED`: Approved Withdrawal
- `SUSPENDED`: Suspended Withdrawal
- `DONE`: Completed Withdrawal
- `REFUSED`: Refused Withdrawal
- `CANCELED`: Cancelled Withdrawal

# Error Format

All error responses use the following format:

```json
{
    "code": "ERROR_CODE",
    "message": "Human-readable error description"
}
```

# *Public API*

## Server Time

`GET /api/v3/time`

> Response

```json
{
    "serverTime": 1630862400
}
```

Returns the current server Unix timestamp.


## Currency Rates

`GET /api/v3/currency-rates`

Returns valuation rates for all currencies as a map of currency code to rate string.

> Response

```json
{
    "BTC": "45000.00000000",
    "ETH": "1632.69000000",
    "USDT": "1.00000000"
}
```


## Currencies

`GET /api/v3/currencies`

```python
public_api.currencies()
```

> Response

```json
[
  {
    "short_name": "BTC",
    "sign": "BTC"
  }, ...
]
```


## Pairs

`GET /api/v3/pairs`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
currency1 | string  | filter by base currency
currency2 | string  | filter by quote currency

```python
public_api.pairs()
```

> Response

```json
[{
    "name": "BTC_USDT",
    "currency1": "BTC",
    "currency2": "USDT",
    "price_precision": 2,
    "amount_precision": 8,
    "minimum_order_size": 0.001,
    "maximum_order_size": 10.0,
    "minimum_order_value": 10.0
}, ...]
```


## Tickers

`GET /api/v3/ticker`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`

```python
public_api.ticker()
```

> Response

```json
[{
    "pair": "BTC_USDT",
    "last": 45000.0,
    "open": 44990.0,
    "vol": 1000.0,
    "buy": 44990.0,
    "sell": 45010.0,
    "high": 45500.0,
    "low": 44500.0,
    "diff": 10.0,
    "diff_percent": 0.02,
    "price_precision": 2,
    "amount_precision": 8,
    "timestamp": 1630862400.0
}, ...]
```


## Orderbook

`GET /api/v3/orderbook`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`, **required**
limit_buy | integer  | limit of buy levels
limit_sell | integer  | limit of sell levels

```python
public_api.orderbook('BTC_USDT')
```

> Response

```json
{
    "buy": [
        {
            "amount": 1.0,
            "price": 45000.0
        },
        {
            "amount": 2.0,
            "price": 44990.0
        }
    ],
    "sell": [
        {
            "amount": 1.0,
            "price": 45010.0
        },
        {
            "amount": 2.0,
            "price": 45020.0
        }
    ]
}
```

## Charts

`GET /api/v3/charts`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`, **required**
interval | string  | chart interval (`1`, `5`, `15`, `30`, `60`, `240`, `1D`), **required**
since | integer | Unix timestamp, filter from
until | integer | Unix timestamp, filter to
limit | integer | limit of candles (default 500, max 500)

```python
import time
from alpcom_api import dto

candle_list = public_api.charts(
    pair='BTC_USDT',
    interval=dto.ChartInterval.DAY,
    since=int(time.time()) - 60 * 60 * 24 * 10  # last 10 days
)
```

> Response

```json
[{
    "time": 1630862400,
    "open": 45000.0,
    "close": 45100.0,
    "low": 44900.0,
    "high": 45200.0,
    "volume": 1000.0,
    "quoteVolume": 45050000.0,
    "tradesCount": 150,
    "takerBuyBaseVolume": 600.0,
    "takerBuyQuoteVolume": 27030000.0
}, ...]
```

## Trades

`GET /api/v3/trades`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`, **required**
limit | integer | limit of records

```python
last_trades = public_api.trades(pair='BTC_USDT', limit=10)
```

> Response

```json
[{
    "id": 1,
    "price": 45000.0,
    "amount": 2.5,
    "timestamp": 1630862400.0,
    "pair": "BTC_USDT",
    "type": "buy"
}, ...]
```

# *Private API*

## Authorization

To access private API, clients must use JWT tokens for authentication. Tokens must be included in the Authorization header as follows: `Authorization: Bearer {token}`


### Token Expiration
Clients should check the key `exp` in the JWT token payload, which contains a timestamp indicating the token's expiration time. Tokens are valid until this time.

### Obtaining JWT Tokens
JWT token generates by API Key and API Secret.
The API Key pair can be generated in user profile, by link `https://alp.com/en/profile/api`
To generate it, make a POST request to the following endpoint: `https://alp.com/api/v3/auth` with body
```json
{
  "api_key": "<Your API Key>",
  "api_secret": "<Your API Secret>"
}
```

> Response (JWT token string)

```json
"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### POST/DELETE Method Signature (X-SIGN Header)
For POST and DELETE requests to trading endpoints, clients must include a request signature in the `X-SIGN` header.
Calculate this signature as the HMAC SHA-256 hash of the request body using the `api_secret` obtained earlier.


## Accounts

`GET /api/v3/accounts/accounts`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
include_subaccounts | bool | if set true, all subaccounts for account are returned as well

```python
private_api.accounts().accounts(include_subaccounts=True)
```

> Response

```json
[{
    "id": 123,
    "email": "example@email.com",
    "name": "John Doe"
}]
```

where:
`parent_id` - present only for sub-accounts, indicates the ID of the master account


## Account Balances

`GET /api/v3/accounts/balances`

```python
private_api.accounts().balances()
```

> Response

```json
[{
    "balance": 1000.0,
    "reserve": 500.0,
    "currency": "USDT",
    "wallet_type": "SPOT",
    "last_update_at": "2023-09-04T12:00:00Z",
    "account_id": 12345
}
...]
```
where:
`reserve` - balance in open orders, pending withdraws, etc.


## Fee Info

`GET /api/v3/accounts/feeinfo`

```python
private_api.accounts().fees()
```

> Response

```json
[{
    "account_id": 12345,
    "active_once": ["2023-09-04T12:00:00Z", "2023-09-05T12:00:00Z"],
    "taker_fee_rate": 0.002,
    "maker_fee_rate": 0.0015,
    "fee_currency": "USD",
    "priority": 1,
    "currency": null,
    "pair": null,
    "is_active": true
}...]
```

Field | Type | Description
----- |------| -----------
account_id | integer | account ID
active_once | array | `[valid_from, valid_to]` as ISO8601 strings, null if unbounded
taker_fee_rate | float | taker fee rate
maker_fee_rate | float | maker fee rate
fee_currency | string | currency code for fee payment, null if default
priority | integer | rule priority
currency | integer | currency ID filter, null if applies to all
pair | integer | pair ID filter, null if applies to all
is_active | boolean | whether the rule is active


## Orders

`GET /api/v3/accounts/order`

Returns orders for the authenticated account (up to 100 most recent).

```python
private_api.accounts().orders()
```

> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "pair": "BTC_USDT",
    "amount": 1.5,
    "price": 45000.0,
    "side": "BID",
    "status": "FILLED",
    "type": "LIMIT",
    "wallet_type": "SPOT",
    "amount_canceled": 0.0,
    "amount_filled": 1.5,
    "date": "2023-09-04T12:00:00Z",
    "open_at": "2023-09-04T11:59:00Z",
    "tif": "GTC",
    "updated_at": "2023-09-04T12:01:00Z",
    "done_at": null,
    "expire_after": null,
    "stop_price": null
} ...]
```

Field | Type | Description
----- |------| -----------
id | integer | order ID
account_id | integer | account ID
pair | string | pair symbol
amount | float | base amount
price | float | limit price (0 for market orders)
side | string | `BID` or `ASK`
status | string | `PENDING`, `OPEN`, `FILLED`, `CANCELLED`, `PARTIAL_FILLED`, `PARTIAL_CANCELED`, `FAILED`
type | string | `MARKET`, `LIMIT`, `STOP_LIMIT`, `STOP_MARKET`
wallet_type | string | `SPOT`, `CROSS_MARGIN`, `ISOLATED_MARGIN`
amount_canceled | float | cancelled amount
amount_filled | float | filled amount
date | string | creation timestamp (ISO8601), null if not set
open_at | string | when the order became active (ISO8601), null if not set
tif | string | time in force: `GTC`, `IOC`, `FOK`
updated_at | string | last update timestamp (ISO8601), null if not set
done_at | string | completion timestamp (ISO8601), null if not set
expire_after | string | expiration timestamp (ISO8601), null if not set
stop_price | float | stop trigger price, null if not a stop order


## Account Trades

`GET /api/v3/accounts/trades`

Query Parameters

Parameter | Type    |   Description
--------- |---------| ----------------------
pair | string  | pair symbol like `BTC_USDT`
side | string  | `BUY` or `SELL`
start_time | integer | Unix timestamp, filter from (>=)
end_time | integer | Unix timestamp, filter to (<)
limit | integer | limit of records (default 100, max 100)

```python
private_api.accounts().trades('ETH_USDT')
```

> Response

```json
[{
    "id": 293107798,
    "pair": "BTC_USDT",
    "side": "BUY",
    "price": 45000.0,
    "amount": 1.0,
    "fee": 0.005,
    "timestamp": 1630862400.0,
    "order_id": 123456
}, ...]
```

Field | Type | Description
----- |------| -----------
id | integer | match/trade ID
pair | string | pair symbol
side | string | `BUY` or `SELL`
price | float | trade price
amount | float | trade amount
fee | float | fee amount charged
timestamp | float | Unix timestamp (with microsecond precision)
order_id | integer | the order ID that was filled


## Operation History

`GET /api/v3/accounts/activity`

Query Parameters

| Parameter | Type    |   Description|
|-----------|---------| ----------------------|
| limit  | integer  | limit of records (default 100, max 1000)|
| offset | integer  | offset for pagination (default 0)|

```python
private_api.accounts().history()
```

> Response

```json
[{
    "balance": 500.0,
    "reserve": 0,
    "currency": "USDT",
    "wallet_type": "SPOT",
    "last_update_at": "2023-09-04T12:00:00Z",
    "account_id": 0,
    "op_type": 1
}...]
```

Field | Type | Description
----- |------| -----------
balance | float | operation amount
reserve | float | reserve change
currency | string | currency code
wallet_type | string | wallet type (`SPOT`, `CROSS_MARGIN`, `ISOLATED_MARGIN`)
last_update_at | string | operation date
account_id | integer | account ID
op_type | integer | operation type code


## Place Order

`POST /api/v3/trading/order`

Body Parameters

| Attribute       | Type     | Limit    | Stop Limit | Stop Market | Market   | Description                                      |
|-----------------|----------|----------|------------|-------------|----------|--------------------------------------------------|
| pair            | string   | required | required   | required    | required | The trading pair symbol, e.g. `BTC_USDT`.        |
| order_type      | string   | required | required   | required    | required | `MARKET`, `LIMIT`, `STOP_LIMIT`, or `STOP_MARKET`. |
| order_side      | string   | required | required   | required    | required | `BUY` or `SELL` (also accepts `BID` / `ASK`).    |
| wallet_type     | string   | required | required   | required    | required | `SPOT`, `MARGIN_CROSS`, `MARGIN_ISOLATED`, or `FUNDING`. |
| base_amount     | string   | required | required   | required    | optional | Base amount as decimal string, e.g. `"1.5"`.     |
| quote_amount    | string   |          |            |             | optional | Quote amount as decimal string (market orders).   |
| limit_price     | string   | required | required   |             |          | Limit price as decimal string.                    |
| stop_price      | string   |          | required   | required    |          | Stop trigger price as decimal string.             |
| stop_operator   | string   |          | required   | required    |          | `GTE` or `LTE`.                                   |
| client_order_id | string   | optional | optional   | optional    | optional | Client-specified order ID.                        |
| post_only       | boolean  | optional |            |             |          | If true, order will only be placed as maker.      |

> **Note**: For market BUY orders, `quote_amount` is required. For market SELL orders, `base_amount` is required.

> Limit Order Request

```json
{
    "pair": "BTC_USDT",
    "order_side": "BUY",
    "wallet_type": "SPOT",
    "base_amount": "1.0",
    "limit_price": "45000.00",
    "order_type": "LIMIT"
}
```

```python
from alpcom_api import dto

req_limit = dto.LimitOrderRequest(
    pair='ETH_USDT',
    order_side=dto.OrderSide.SELL,
    base_amount=0.05,
    limit_price=1800,
)

private_api.trading().place_order(req_limit)
```

> Market Order Request

```json
{
    "pair": "BTC_USDT",
    "order_side": "BUY",
    "wallet_type": "SPOT",
    "quote_amount": "100.00",
    "order_type": "MARKET"
}
```

```python
from alpcom_api import dto

req_market = dto.MarketOrderRequest(
    pair='ETH_USDT',
    order_side=dto.OrderSide.SELL,
    base_amount=0.1 # base_amount or quote_amount
)

private_api.trading().place_order(req_market)
```

> Stop Limit Order Request

```json
{
    "pair": "BTC_USDT",
    "order_side": "BUY",
    "wallet_type": "SPOT",
    "base_amount": "1.0",
    "limit_price": "45000.00",
    "stop_price": "44000.00",
    "stop_operator": "GTE",
    "order_type": "STOP_LIMIT"
}
```

```python
from alpcom_api import dto

req_stop_limit = dto.StopLimitOrderRequest(
    pair='ETH_USDT',
    order_side=dto.OrderSide.SELL,
    stop_price=1800,
    stop_operator=dto.StopOperator.GTE,
    base_amount=0.05,
    limit_price=1700,
)

private_api.trading().place_order(req_stop_limit)
```

> Response (array of created order IDs)

```json
[34]
```

> Error Response

```json
{
    "code": "1003",
    "message": "invalid order amount"
}
```


## Cancel Order

`DELETE /api/v3/trading/order`

Body Parameters

| Parameter | Type    | Description                                                 |
|-----------|---------|-------------------------------------------------------------|
| order_ids | array   | An array of order IDs, e.g., `[1, 5, 8]`                   |
| all       | boolean | Set `true` to cancel all orders                             |
| pair      | string  | Cancel all orders for a specific pair, e.g. `BTC_USDT`      |

> **Note**: Exactly one of `order_ids`, `all`, or `pair` must be specified.

> Cancel by order IDs

```json
{
  "order_ids": [1, 5, 8]
}
```

> Response (cancel by order IDs) - map of order ID to status

```json
{
  "1": "CANCELLED",
  "5": "CANCELLED",
  "8": "order not found"
}
```

> Cancel all / cancel by pair

```json
{
  "all": true
}
```

> Response: HTTP 202 Accepted (no body)

```python
private_api.trading().cancel_order(12312332)
private_api.trading().cancel_orders([12312332, 12312334, 12312338])
private_api.trading().cancel_orders_of_pair('ETH_USDT')
```


# *WebSocket API*

The WebSocket service by Alp Exchange is a real-time communication platform for traders.
It offers public channels for accessing market data like prices and order book updates, as well as private (authenticated)
channels for personal fund and order change notifications.
This service ensures quick access to trading information, secure authentication, efficient order management,
and an enhanced user experience, catering to the needs of both public and private traders.
The service is available on [wss://www.alp.com/ws](wss://www.alp.com/ws).


```python
import asyncio
from pprint import pprint

from alpcom_api import ws, clients, cache
from alpcom_api.dto import ws as dto


class MyHandler(ws.Handler):
    def on_ticker(self, ticker: dto.Ticker):
        pprint(ticker)

    def on_trade(self, trade: dto.Trade):
        pprint(trade)

    def on_rate(self, rate: dto.Rate):
        pprint(rate)

    def on_diff(self, diff: dto.Diff):
        pprint(diff)

    def on_depth(self, depth: dto.Depth):
        pprint(depth)

    def on_wallet(self, wallet: dto.Wallet):
        pprint(wallet)

    def on_order(self, order: dto.Order):
        pprint(order)

async def main():
    async with ws.Client(handler=MyHandler()) as client:
        await client.subscribe(ws.tps.tickers_all)
        await client.receive_messages()


asyncio.get_event_loop().run_until_complete(main())
```


## Ping

1. **Ping Message:** The server initiates the process by sending a "ping" message, string `1` to the client at regular intervals, every 20 seconds by default.
2. **Pong Message:** Upon receiving the ping message, the client responds with a "pong" message, string `2` to acknowledge that it's still active and responsive.
3. **Connection Monitoring:** The server keeps track of the time since the last ping message was sent. If it doesn't receive a corresponding pong message within a set timeframe, 30 seconds by default, it interprets this as an unresponsive client.
4. **Connection Closure:** In the event that the server doesn't receive a pong message within the specified time frame (30 seconds by default), it takes action to close the connection. This action can include terminating the connection to free up server resources and maintain system efficiency.


## Topic Subscription

Message to subscribe on topics:
`["subscribe", "topic1", "topic2", ...]`

Message to unsubscribe:
`["unsubscribe", "topic1", "topic2", ...]`

The server will respond on subscription or unsubscription with next messages
`["subscribe",1586857703.901509,"topic1","topic1"]`
`["unsubscribe",1586857703.901509,"topic1","topic1"]`

In case of any error, the server will send next message:
`["error",1586857703.901509,"Topic does not exist"]`


## Welcome
The welcome message gets sent when a connection is established. This message provides important initialization information for the client to ensure a smooth and responsive communication experience.

Message Fields
1. **Type (Index 0)**: The type of the message is set to "welcome," indicating that it's a welcome message.
2. **Time (Index 1)**: A floating-point number representing the timestamp of the message.
3. **PingInterval (Index 2)**: An integer representing the ping interval in nanoseconds. This specifies how often the client should send ping messages to the server to maintain the connection.
4. **PingTimeout (Index 3)**: An integer representing the ping timeout in nanoseconds. If the server does not receive a ping response from the client within this timeout period, it may consider the connection as unresponsive and take appropriate action.
5. **ClientID (Index 4)**: A unique identifier (UUID) for the client, allowing the server to distinguish between different clients.

> Event

```json
["welcome", 1694002700, 30000000000, 65000000000, "5c00d9c2-607e-4c7f-927b-b204b29e387b"]
```


## Ticker

Topics: `ticker.*`, `ticker.XXX_YYY`

Fields:

- `Type`: The type of the message is set to "tk," indicating that it's a ticker message.
- `Timestamp`: The Unix timestamp of the ticker data.
- `Ticker 1`: Ticker array.
- `Ticker 2`. Ticker array.
- `Ticker n`. Ticker array.


Each ticker array includes following fields:

- Trading Pair Symbol
- Close Price
- Base Volume
- Quote Volume
- Change Percent
- High Price
- Low Price
- Bid Price
- Ask Price

> Example

```json
[
  "tk",
  1694003337,
  ["XRP_USDT", "0.50057000", "2036106.67737645", "1024194.48541761", "-0.28287417", "0.50677000", "0.49872000", "0.50057000", "0.50117000"],
  ["BTC_USDT", "25725.53000000", "6074.90520117", "156489295.23543299", "0.09369121", "25889.00000000", "25629.43000000", "25717.69000000", "25747.69000000"],
  ["LTC_USDT", "63.06000000", "19800.19751964", "1248404.62483421", "0.39802579", "63.63000000", "62.60000000", "63.03000000", "63.06000000"],
  ["DOGE_USDT", "0.06389400", "26360317.66603325", "1686644.45381961", "0.03131164", "0.06466200", "0.06354300", "0.06384708", "0.06396200"],
  ["ETH_USDT", "1632.69000000", "7379.92830702", "12057736.03072529", "0.10791388", "1648.02000000", "1625.16000000", "1632.69000000", "1635.28000000"],
  ["TRX_USDT", "0.07827864", "17930244.99331608", "1392304.80058763", "1.44319315", "0.07857449", "0.07710881", "0.07832800", "0.07842434"],
  ["USDC_USDT", "1.00130000", "2135117.29179469", "2136330.76840695", "0.08996401", "1.00190090", "0.99929970", "1.00160060", "1.00140000"]
]
```

```python
from alpcom_api import ws
await client.subscribe(ws.tps.tickers_all)
await client.subscribe(ws.tps.tickers_of('BTC_USDT'))
```


## Trade

Topics: `trade.*`, `trade.XXX_YYY`

Fields:

- `Type`: The type of the message is set to "t," indicating that it's a trade event.
- `Timestamp`: The Unix timestamp of the trade event.
- `Trade ID`: A unique identifier for the trade event.
- `Trading Pair Symbol`: The symbol representing the trading pair associated with the trade.
- `Trade Amount`: The amount of the trade, converted to a string.
- `Trade Price`: The price of the trade, converted to a string.
- `Trade Direction`: The direction of the trade, either "buy" or "sell.

> Example

```json
[
  "t",
  1694003848,
  293107798,
  "BTC_USDT",
  "0.20285052",
  "25705.02000000",
  "buy"
]
```

```python
from alpcom_api import ws
await client.subscribe(ws.tps.trades_all)
await client.subscribe(ws.tps.trades_of('BTC_USDT'))
```


## Orderbook Diff

Topics: `diff.XXX_YYY`

Fields:

- `Type`: The type of the message is set to "d," indicating that it's a market depth difference event.
- `Timestamp`: The Unix timestamp of the market depth difference data.
- `Payload`: The payload is an array containing market depth difference information.

Payload includes the following fields:
- `Symbol`: The trading symbol associated with the market depth.
- `Asks`: A two-dimensional array representing the updated ask side of the market depth. Each ask is an array with the price and quantity difference.
- `Bids`: A two-dimensional array representing the updated bid side of the market depth. Each bid is an array with the price and quantity difference.

> Example

```json
[
  "d",
  1694004500,
  "BTC_USDT",
  [
    [["101.00", "5.0"], ["102.00", "3.0"]],
    [["99.00", "2.0"], ["98.50", "1.0"]]
  ]
]
```


```python
from alpcom_api import ws
await client.subscribe(ws.tps.diff_of('BTC_USDT'))
```


## Market Depth

Topics: `market_depth.XXX_YYY`

Fields:

- `Type`: The type of the message is set to "p," indicating that it's a market depth event.
- `Timestamp`: The Unix timestamp of the market depth data.
- `Payload`: Market depth object.

Market depth object includes the following fields:

- `Asks`: A two-dimensional array representing the ask side of the market depth. Each ask is an array with the price and quantity.
- `Bids`: A two-dimensional array representing the bid side of the market depth. Each bid is an array with the price and quantity.
- `Symbol`: The trading symbol associated with the market depth.
- `TotalAsks`: The total quantity of asks.
- `TotalBids`: The total quantity of bids.

> Example

```json
[
  "p",
  1694178331,
  {
    "Asks": [
      [
        "26100.11000000",
        "0.30200000"
      ],
      [
        "26149.68000000",
        "0.51800000"
      ]
    ],
    "Bids": [
      [
        "25768.09000000",
        "0.05900000"
      ],
      [
        "25748.38000000",
        "0.46300000"
      ]
    ],
    "Symbol": "BTC_USDT",
    "TotalAsks": "148953.50292677",
    "TotalBids": "148953.97125366"
  }
]
```

```python
from alpcom_api import ws
await client.subscribe(ws.tps.depth_of('BTC_USDT'))
```

## Auth

You can use JWT-token, mentioned above, that is used for access to private API to authorize within web socket connection.
To make it send next message:
`["auth", "Generated JWT Token"]`

```python
cli = clients.ALPAuthClient(
    key='**API_KEY**',
    secret='**API_KEY**',
)
await client.auth(cli)
```


## Account Order Update

After the authorization, you will receive updates of your orders on trades.

Fields:

- `Type`: The type of the message is set to "o" indicating that it's an order event.
- `Timestamp`: The Unix timestamp of the message.
- `Order Id`: The unique identifier for the order.
- `Symbol`: The trading symbol associated with the order.
- `Side`: The side of the order (e.g., 'buy' or 'sell').
- `Order Type`: The type of order (e.g., 'limit' or 'market').
- `Base Amount`: The quantity of the base currency in the order.
- `Limit Price`: The specified price for a limit order.
- `Amount Unfilled`: The quantity of the order that remains unfilled.
- `Amount Filled`: The quantity of the order that has been filled.
- `Amount Cancelled`: The quantity of the order that has been cancelled.
- `Value Filled`: The total value of the order that has been filled.
- `Price Avg`: The average price at which the order has been filled.
- `Done At`: The timestamp indicating when the order was completed.
- `Status`: The current status of the order (e.g., 'open', 'completed', 'cancelled').
- `Quote Amount`: The amount in the quote currency.
- `Wallet Type`: The type of wallet associated with the order (e.g., 'spot', 'margin').
- `Stop Price`: The specified stop price for a stop order.
- `Stop Operator`: The operator for the stop price (e.g., 'greater_than', 'less_than').


```json
[
  "o",
  1694178331,
  12345,
  "BTC_UST",
  "ASK",
  "LIMIT",
  "0.5",
  "48000.00",
  "0.2",
  "0.3",
  "0.0",
  "14400.00",
  "48000.00",
  1631386242.567,
  "FILLED",
  "14400.00",
  "SPOT",
  "0.00",
  ""
]
```

## Account Wallet Update

After the authorization, you will receive updates of your balances after each change.

Fields:

- `Type`: The type of the message is set to "w" indicating that it's a wallet update event.
- `Timestamp`: The Unix timestamp of the message.
- `Code`: Represents the unique code associated with the item.
- `Wallet Type`: Indicates the type of wallet  (e.g., 'spot', 'margin').
- `Symbol`: Denotes the symbol of margin isolated wallet.
- `Balance`: Reflects the current balance of the wallet.
- `Reserve`: Specifies the reserved amount for the wallet.

```json
[
  "w",
  1694178331,
  "USDT",
  "SPOT",
  "",
  "5.0",
  "1.0"
]
```
