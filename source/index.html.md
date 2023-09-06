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

 - `UNDEFINED`: Undefined Order Status
 - `PENDING`: Pending Order
 - `OPEN`: Open Order
 - `CANCELLED`: Cancelled Order
 - `PARTIAL_CANCELLED`: Partially Cancelled Order
 - `PARTIAL_FILLED`: Partially Filled Order
 - `FILLED`: Filled Order
 - `EXPIRED`: Expired Order
 - `FAILED`: Failed Order

### Time in Force

 - `GOOD_TILL_CANCEL`: Good 'til Cancelled
 - `IMMEDIATE_OR_CANCEL`: Immediate or Cancel
 - `ALL_OR_NONE`: All or None
 - `FILL_OR_KILL`: Fill or Kill
 - `UNDEFINED_TIME_IN_FORCE`: Undefined Time in Force

### Side Effects

 - `NO_SIDE_EFFECT`: No Side Effect
 - `AUTO_BORROW`: Auto Borrow
 - `AUTO_REPLAY`: Auto Replay

### Stop Operators

 - `GTE`: Greater Than or Equal To
 - `LTE`: Less Than or Equal To

### Loan Types

 - `AUTO`: Auto Loan Type
 - `MANUAL`: Manual Loan Type

### Margin Directions

 - `ADD`: Add to Margin
 - `SUB`: Subtract from Margin

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

### Chart Intervals

 - `FIVE_MIN`: 5 minutes
 - `FIFTEEN_MINS`: 15 minutes
 - `HALF_HOUR`: 30 minutes
 - `HOUR`: 1 hour
 - `FOUR_HOURS`: 4 hours
 - `DAY`: 1 day

# *Public API*

## Currencies

`GET /api/v3/currencies`

```python
public_api.currencies()
```

> Response

```json
[
  {
    "sign": "Ƀ",
    "short_name": "BTC"
  }, ...
]
```

## Pairs

`GET /api/v3/pairs`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
currency1 | string  | filter by base currency
currency2 | integer | filter by quote currency

```python
public_api.pairs()
```

> Response

```json
[{
    "name": "BTC/USDT",
    "currency1": "BTC",
    "currency2": "USDT",
    "amount_precision": 8,
    "price_precision": 2,
    "maximum_order_size": 10.0,
    "minimum_order_size": 0.001,
    "minimum_order_value": 10.0
}, ...]
```

## Tickers

`GET /api/v3/ticker`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`
pair_id | integer | id of pair

```python
public_api.ticker()
```

> Response

```json
[{
    "pair": "BTC/USD",
    "last": 45000.0,
    "vol": 1000.0,
    "buy": 44990.0,
    "sell": 45010.0,
    "high": 45500.0,
    "low": 44500.0,
    "diff": 10.0,
    "timestamp": 1630862400.0
}, ...]
```

## Orderbook

`GET /api/v3/order_book`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`
group | integer | if set `1` - orders will be grouped by price
limit_buy | integer  | limit of buy orders
limit_sell | integer  | limit of sell orders

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
interval | string  | time interval, **required**
since | integer | timestamp, filter from
until | integer | timestamp, filter to 
limit | integer | timestamp, limit of candles

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
    "open": 45000.0,
    "close": 45100.0,
    "high": 45200.0,
    "low": 44900.0,
    "volume": 1000.0,
    "time": 1630862400.0
}, ...]
```

## Trades

`GET /api/v3/trades`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
pair | string  | pair symbol like `BTC_USDT`
limit | integer | limit of records
offset | integer | offset of records

```python
last_trades = public_api.trades(pair='BTC_USDT', limit=10)
```

> Response

```json
[{
    "id": 1,
    "account_id": 101,
    "pair": "BTC/USDT",
    "amount": 2.5,
    "price": 45000.0,
    "timestamp": 1630862400.0,
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

### POST Method Signature (X-SIGN Header)
For POST requests, clients must include a request signature in the `X-SIGN` header. 
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
    "name": "John Doe",
    "parent_id": null
}]
```

where:
`parent_id` - for sub accounts it indicates id of master account


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

## Orders

`GET /api/v3/accounts/order`

Query Parameters

| Parameter | Type    | Description|
|-----------|---------| -----------|
| pair      | string  | pair symbol like `BTC_USDT`|
| open_only | bool    | if set and true returns open order only|
| side      | string  | if set must be rather `BUY` or `SELL`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
| end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|

```python
private_api.accounts().orders('ETH_USDT', open_only=True)
```

> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "pair": "BTC/USDT",
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


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.


## Account Trades

`GET /api/v3/accounts/trades`

Query Parameters


Parameter | Type    |   Description
--------- |---------| ----------------------
pair | string  | pair symbol like `BTC_USDT`
side | string  | if set must be rather `BUY` or `SELL`
start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`
end_time | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`

```python
private_api.accounts().trades('ETH_USDT')
```

> Response

```json
[{
    "account_id": 12345,
    "pair": "BTC/USD",
    "amount": 1.0,
    "amount_unfilled": 0.5,
    "price": 45000.0,
    "currency": "USD",
    "status": "FILLED",
    "date": "2023-09-04T12:00:00Z",
    "fee_amount": 0.005,
    "fee_currency": "USD",
    "fee_rate": 0.005,
    "is_maker": true,
    "type": "LIMIT",
    "wallet_type": "SPOT"
}, ...]
```

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.



## Operation History

`GET /api/v3/accounts/activity`

Query Parameters

| Parameter | Type    |   Description|
|-----------|---------| ----------------------|
| currency  | string  | currency symbol like `BTC`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
| end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|

```python
private_api.accounts().history('USDT')
```

> Response

```json
[{
    "id": 1,
    "account_id": 12345,
    "wallet_id": 67890,
    "type": "DEPOSIT",
    "balance": 1000.0,
    "balance_delta": 500.0,
    "balance_end": 1500.0,
    "reserve": 100.0,
    "reserve_delta": 50.0,
    "reserve_end": 150.0,
    "currency": "USDT",
    "wallet_type": "SPOT",
    "date": "2023-09-04T12:00:00Z"
}...]
```


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.



## Deposit Methods

`GET /api/v3/deposit/methods`

Query Parameters

| Parameter | Type    | Description                            |
|-----------|---------|----------------------------------------|
| currency  | string  | currency symbol like `BTC` (required!) |

```python
private_api.deposits().methods(currency='USDT')
```

> Response

```json
[{
    "network": "Example Network",
    "currency": "USDT",
    "attributes": [
        {
            "key": "address",
            "value": "0x1234567890abcdef"
        },
        {
            "key": "memo",
            "value": "123456"
        }
    ],
    "confirmations": 6,
    "min_deposit": 10.0,
    "max_deposit": 1000.0,
    "is_enabled": true
}, ...]
```

## Deposit History

`GET /api/v3/deposit`

Query Parameters

| Parameter | Type    |   Description|
|-----------|---------| ----------------------|
| currency  | string  | currency symbol like `BTC`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
| end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|


```python
private_api.deposits().history(currency='USDT')
```

> Response

```json
[{
    "id": 123,
    "amount": 100.0,
    "currency": "BTC",
    "status": "PENDING",
    "tx_hash": "0xabc123def456",
    "date": "2023-09-04T12:00:00Z"
}, ...]
```

## Withdraw Methods

`GET /api/v3/withdraw/methods`

Query Parameters

| Parameter | Type    | Description                 | Required   |
|-----------|---------|-----------------------------|------------|
| currency  | string  | currency symbol like `BTC`  | true       |


```python
private_api.withdraws().methods(currency='USDT')
```

> Response

```json
[{
    "id": 123,
    "network": "ETH",
    "currency": "USD",
    "attributes": [
        {
            "key": "address",
            "value": "0x1234567890abcdef"
        },
        {
            "key": "memo",
            "value": "123456"
        }
    ],
    "fee_rate": 0.002,
    "fee_amount": 5.0,
    "fee_currency": "USD",
    "min_withdraw": 10.0,
    "max_withdraw": 1000.0,
    "is_enabled": true
}
, ...]
```

> **Note**
>
> This method is disabled by default, set "Can Withdraw" during creation of an API-Key to use it.


## Withdraw Request

`POST /api/v3/withdraw`

Body Parameters

| Parameter        | Type    | Description                                             | Required |
|------------------|---------|---------------------------------------------------------|----------|
| amount           | number  | The amount, e.g., 50.0                                  | true     |
| method           | number  | The method identifier, e.g., 123                        | true     |
| attributes       | object  | Attributes for payment: address, memo, credit card, etc | true     |
| client_order_id  | string  | Client order ID, e.g., "withdraw-1"                     | false    |



```python
from alpcom_api import dto

req = dto.WithdrawRequest(
    amount=10.0,
    method=2,
    attributes={
        'address': '<my_dest_addr>',
        'memo': '<extra memo>'
    },
    client_order_id='order_123'
)
private_api.withdraws().create(req)
```

> Query Body

```json
{
    "amount": 50.0,
    "method": 123,
    "attributes": {
        "address": "0xabcdef1234567890",
        "memo": "123456"
    },
    "client_order_id": "withdraw-1"
}
```

> **Note**
>
> This method is disabled by default, set "Can Withdraw" during creation of an API-Key to use it.


## Withdraw History

`GET /api/v3/withdraw`

Query Parameters

| Parameter | Type    |   Description|
|-----------|---------| ----------------------|
| currency  | string  | currency symbol like `BTC`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
| end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|

```python
private_api.withdraws().history(currency='USDT')
```

> Response

```json
[{
    "id": 123,
    "amount": 50.0,
    "fee_amount": 2.0,
    "currency": "BTC",
    "fee_currency": "BTC",
    "status": "COMPLETED",
    "tx_hash": "0xabcdef1234567890",
    "date": "2023-09-04T12:00:00Z"
}
, ...]
```

> **Note**
>
> This method is disabled by default, set "Can Withdraw" during creation of an API-Key to use it.


## Place Order

`POST /api/v3/trading/order`

Body Parameters

| Attribute       | Type     | Limit    | Stop Limit | Market   | Description                                      |
|-----------------|----------|----------|------------|----------|--------------------------------------------------|
| pair            | string   | required | required   | required | The trading pair.                                |
| order_type      | string   | required | required   | required | MARKET, LIMIT, STOP_LIMIT, or STOP_MARKET.       |
| order_side      | string   | required | required   | required | BID or ASC.                                      |
| wallet_type     | string   | required | required   | required | SPOT, CROSS_MARGIN, ISOLATED_MARGIN, or FUNDING. |
| client_order_id | string   | required | required   | required | The client order ID.                             |
| side_effect     | string   | required | required   | required | NO_SIDE_EFFECT, AUTO_BORROW, AUTO_REPLAY         |
| expire_after    | datetime | required | required   | required | The expiration date and time.                    |
| base_amount     | float    | required | required   | optional | The base amount of the order.                    |
| quote_amount    | float    |          |            | optional | The quote amount of the order.                   |
| limit_price     | float    | required | required   |          | The limit price for a order.                     |
| stop_price      | float    |          | required   |          | The stop price, required for STOP_LIMIT.         |
| stop_operator   | string   |          | required   |          | GTE, LTE, required for STOP_LIMIT.               |
| valid_till      | optional | optional | optional   | optional | The valid till date and time.                    |
| tif             | optional | optional | optional   | optional | GTC, IOC, AON, FOK                               |



> Limit Order Request

```json
{
    "pair": "BTC/USD",
    "order_side": "BID",
    "client_order_id": "1",
    "wallet_type": "SPOT",
    "side_effect": "NO_SIDE_EFFECT",
    "valid_till": null,
    "expire_after": null,
    "base_amount": 1.0,
    "limit_price": 45000.0,
    "order_type": "LIMIT",
    "tif": "GTC"
}
```

> Market Order Request

```json
{
    "pair": "BTC/USD",
    "order_side": "BID",
    "client_order_id": "1",
    "wallet_type": "SPOT",
    "side_effect": "NO_SIDE_EFFECT",
    "valid_till": null,
    "expire_after": null,
    "base_amount": null,
    "quote_amount": null,
    "order_type": "MARKET"
}
```

> Stop Limit Order Request

```json
{
    "pair": "BTC/USD",
    "order_side": "BID",
    "client_order_id": "1",
    "wallet_type": "SPOT",
    "side_effect": "NO_SIDE_EFFECT",
    "valid_till": null,
    "expire_after": null,
    "base_amount": 1.0,
    "limit_price": 45000.0,
    "stop_price": 44000.0,
    "stop_operator": "GTE",
    "order_type": "STOP_LIMIT",
    "tif": "GTC"
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

> Response, (Array of created order ids)

```json
[34]
```


## Cancel Order

`DELETE /api/v3/trading/order`

Body Parameters

| Parameter | Type    | Description                                                 |
|-----------|---------|-------------------------------------------------------------|
| order_ids | array   | An array of order IDs, e.g., [1, 5, 8]                      |
| all       | boolean | rather `order_ids` or `all`=`true` must be specified        |
| pair      | string  | can be specified to cancel all orders for one specific pair |


> Query Body

```json
{
  "order_ids": [1, 5, 8],
  "all": false,
  "pair": ""
}
```

```python
private_api.trading().cancel_order(12312332)
private_api.trading().cancel_orders([12312332, 12312334, 12312338])
private_api.trading().cancel_orders_of_pair('ETH_USDT')
```


## Margin Transfer

`POST /api/v3/margin/transfer`

Body Parameters

| Parameter   | Type    | Description                       | Required |
|-------------|---------|-----------------------------------|----------|
| account_id  | integer | The account ID, e.g., 12345       | true     |
| wallet_type | string  | The wallet type, e.g., "SPOT"     | true     |
| direction   | string  | "ADD" or "SUB"                    | true     |
| amount      | number  | The amount, e.g., 100.0           | true     |
| currency    | string  | The currency symbol, e.g., "USDT" | true     |
| pair        | null    | The trading pair, e.g., null      | false    |
| note        | null    | A note, e.g., null                | false    |


```python
from alpcom_api import dto

req = dto.MarginTransferRequest(
    account_id=384457,
    wallet_type=dto.WalletType.MARGIN_CROSS,
    direction=dto.MarginDirection.ADD,
    amount=1,
    currency='ETH',
)

operation_id = private_api.margin().transfer(req)
```

> Query Body

```json
{
    "account_id": 12345,
    "wallet_type": "SPOT",
    "direction": "ADD",
    "amount": 100.0,
    "currency": "USDT",
    "pair": null,
    "note": null
}
```

> Response:

```text
123 # transfer id
```

> **Note**
>
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.


## Margin Borrow

`POST /api/v3/margin/borrow`

Body Parameters

| Parameter   | Type    | Description                           | Required |
|-------------|---------|---------------------------------------|----------|
| account_id  | integer | The account ID, e.g., 12345           | true     |
| borrow      | number  | The amount to borrow, e.g., 100.0     | true     |
| currency    | string  | The currency symbol, e.g., "USD"      | true     |
| wallet_type | string  | The wallet type, e.g., "SPOT"         | true     |
| pair        | null    | The trading pair, e.g., null          | false    |
| type        | string  | The type of loan, e.g., "LOAN_MANUAL" | true     |


```python
from alpcom_api import dto

req = dto.BorrowRequest(
    account_id=384457,
    borrow=1,
    currency='ETH',
    wallet_type=dto.WalletType.MARGIN_CROSS,
)

operation_id = private_api.margin().borrow(req)
```


> Query Body

```json
{
    "account_id": 12345,
    "borrow": 100.0,
    "currency": "USD",
    "wallet_type": "SPOT",
    "pair": null,
    "type": "LOAN_MANUAL"
}
```

> Response:
```text
123 # borrow id
```

> **Note**
>
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.


## Margin Repay

`POST /api/v3/margin/repay`

Body Parameters

| Parameter   | Type    | Description                           | Required |
|-------------|---------|---------------------------------------|----------|
| account_id  | integer | The account ID, e.g., 12345           | true     |
| borrow      | number  | The amount to borrow, e.g., 100.0     | true     |
| currency    | string  | The currency symbol, e.g., "USD"      | true     |
| wallet_type | string  | The wallet type, e.g., "SPOT"         | true     |
| pair        | null    | The trading pair, e.g., null          | false    |
| type        | string  | The type of loan, e.g., "LOAN_MANUAL" | true     |


```python
from alpcom_api import dto

req = dto.RepayRequest(
    account_id=384457,
    amount=0.03,
    currency='ETH',
    wallet_type=dto.WalletType.MARGIN_CROSS,
)

operation_id = private_api.margin().repay(req)
```


> Query Body

```json
{
    "account_id": 12345,
    "amount": 100.0,
    "currency": "USD",
    "wallet_type": "SPOT",
    "pair": null,
    "type": "LOAN_MANUAL"
}
```

> Response:
```text
123 # repay id
```

> **Note**
>
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.


### Close Position

Body Parameters

| Parameter   | Type   | Description                              | Required |
|-------------|--------|------------------------------------------|----------|
| wallet_type | string | The wallet type, e.g., "ISOLATED_MARGIN" | true     |
| pair        | string | The trading pair, e.g., "BTC_USDT"       | false    |


`DELETE /api/v3/margin`

```python
private_api.margin().close_position(account_id=384457, wallet_type=dto.WalletType.MARGIN_CROSS)
```

> Query Body

```json
{
  "pair": "BTC_USDT",
  "wallet_type": "ISOLATED_MARGIN"
}
```

> **Note**
>
> rather `ISOLATED_MARGIN` or `CROSS_MARGIN` must be set to `wallet_type`
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Loans

`GET /api/v3/margin`

Query Parameters

| Parameter  | Type | Description|
|------------|--| -----------|
| isolated_margin | bool |  if set true isolated margin positions are returned |
| cross_margin   | bool |  if set true cross margin positions are returned |


```python
loans = private_api.margin().loans(cross_margin=True)
```


> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "borrowed": 1000.0,
    "interest": 50.0,
    "currency": "USD",
    "wallet_type": "SPOT"
}, ...]
```


> **Note**
>
> rather `isolated_margin` or `cross_margin` must be set true
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Transfer History

`GET /api/v3/margin/transfer`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| open_only  | bool    | if set and true returns open order only                     |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |


```python
private_api.margin().transfers('ETH', start_time=None, end_time=None)
```


> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "amount": 100.0,
    "currency": "USD",
    "date": "2023-09-04T12:00:00Z",
    "direction": "ADD",
    "wallet_type": "SPOT",
    "note": "Transfer from Spot to Margin",
    "pair": "BTC/USD"
}, ...]
```


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Borrow History

`GET /api/v3/margin/borrow`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| open_only  | bool    | if set and true returns open order only                     |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |


```python
private_api.margin().borrows('ETH', start_time=None, end_time=None)
```


> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "borrowed": 1000.0,
    "date": "2023-09-04T12:00:00Z",
    "interest_accrued": 50.0,
    "interest_paid": 10.0,
    "interest_rate": 0.05,
    "repaid": 0.0,
    "last_interest_at": null,
    "opened_at": "2023-09-04T11:00:00Z",
    "type": "AUTO",
    "wallet_type": "SPOT"
}, ...]
```


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Repay History

`GET /api/v3/margin/repay`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

```python
private_api.margin().repays('ETH', start_time=None, end_time=None)
```


> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "interest_amount": 10.0,
    "principal_amount": 100.0,
    "currency": "USDT",
    "date": "2023-09-04T12:00:00Z",
    "type": "AUTO",
    "wallet_type": "SPOT"
}
...]
```


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Interest History

`GET /api/v3/margin/interest`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |


```python
private_api.margin().interests('ETH', start_time=None, end_time=None)
```


> Response

```json
[{
    "id": 123,
    "account_id": 456,
    "borrow_id": 789,
    "currency": "USDT",
    "interest_accrued": 50.0,
    "interest_paid": 10.0,
    "interest_rate": 0.05,
    "wallet_type": "SPOT"
}, ...]
```


> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.



## Margin Liquidation History

`GET /api/v3/margin/liquidation`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| open_only  | bool    | if set and true returns open order only                     |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |


```python
private_api.margin().liquidations('ETH', start_time=None, end_time=None)
```


> Response

```json
[{
  "id": 293878,
  "account_id": 25,
  "created_at": "2022-10-06T08:28:06.801064-04:00",
  "wallet_type": "ISOLATED_MARGIN",
  "margin_level": 0.002,
  "closed_at": "",
  "amount": 0.015,
  "fund_amount": 0.03,
  "currency": "ETH",
  "order_closed": [1, 3],
  "order_created": [5],
  "borrow_closed": [22, 41],
  "repay_created": [54]
}...]
```

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.
> This method is disabled by default, set "Margin" flag during creation of an API-Key to use it.
