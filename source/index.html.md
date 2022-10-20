---
title: ALP.COM API Reference

language_tabs:
  - python: Python
  - javascript: Node.js

toc_footers:
  - <a href='https://alp.com/en/register/'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---


# Introduction


Welcome to ALP.COM API docs!

Service ALP.COM provides open API for trading operations and broadcasting of all trading events.

# HTTP API (v1)


HTTP API endpoint available on [https://alp.com/api/v1/](https://alp.com/api/v1/).

Some methods requires authorization [read below](#authorization).

<aside class="success">
We have limit in 2 calls per second from single account to authorization required methods and 100 calls per secong from single IP address to public methods.
</aside>

All HTTP methods accept `JSON` formats of requests and responses if it not specified by headers.

## Authorization

> To generate auth headers, use this code:

```python
import hmac
from time import time
from urllib.parse import urlencode

def get_auth_headers(self, data):
        msg = 'your key' + urlencode(sorted(data.items(), key=lambda val: val[0]))

        sign = hmac.new('your keys secret'.encode(), msg.encode(), digestmod='sha256').hexdigest()

        return {
            'X-KEY': 'your key',
            'X-SIGN': sign,
            'X-NONCE': str(int(time() * 1000)),
        }
```

```javascript
const hmacSha256 = require('crypto-js/hmac-sha256'); // sha256 hash. or use you favorite u like
const request = require('request'); // for http requests. or use you favorite u like

const BASE_URL = 'https://alp.com/api/v1';
const API_KEY = '0000-0000...';
const SECRET = 'Z%2........';

//Serialize for singing only. Can be used in request body if u like urlencoded form format instead of json
function serializePayload(payload) {
  return Object
    .keys(payload) // get keys of payload object
    .sort() // sort keys
    .map((key) => key + "=" + encodeURIComponent(payload[key])) // each value should be url encoded. the most sensitive part for sign checking
    .join('&'); // to sting, separate with ampersand
}

// Generates auth headers
function getAuthHeaders(payload) {
  // get SHA256 of <API_KEY><sorted urlencoded payload string><SECRET>
  const sign = hmacSha256(API_KEY + serializePayload(payload), SECRET).toString();

  return {
    'X-KEY': API_KEY,
    'X-SIGN': sign,
    'X-NONCE': Date.now()
  };
}

function getWallets(callback) {
  payload = {};

  const options = {
    method: 'get',
    url: `${BASE_URL}/wallets/`,
    headers: getAuthHeaders(payload)
  };

  request(options, callback);
}

function createOrder(order, callback) {
  const options = {
    method: 'post',
    url: `${BASE_URL}/order/`,
    headers: getAuthHeaders(order),
    form: order, // API accepts urlencoded form or json. Use appropriate headers!
  };

  request(options, callback);
}

// test
getWallets((error, response, body) => {
  console.log('error', error);
  console.log('body', body);
});

const order = {
  type: 'buy',
  pair: 'BTC_USD',
  amount: '0.0001',
  price: '0.1'
};

createOrder(order, (error, response, body) => {
  // get json, etc
  console.log('error', error);
  console.log('body', body);
});
```

In order for access to private API methods, generate authorization keys [in profile settings](https://alp.com/en/profile/api).

All request to these methods must contain the following headers:

* X-KEY - your key.
* X-SIGN - query's POST data, sorted by keys and signed by your key's "secret" according to the HMAC-SHA256 method.
* X-NONCE - integer value, must be greater then nonce in previous api call.


# Currencies


## List all currencies


```python
import requests

response = requests.get('https://alp.com/api/v1/currencies/')
```

```javascript
var request = require('request');

request.get('https://alp.com/api/v1/currencies/', function (error, response, body) {
    // process response
});
```

> Sample output

```json
[
  {
    "sign": "Ƀ",
    "short_name": "BTC"
  },
  {
    "sign": "Ξ",
    "short_name": "ETH"
  }
]
```

Returns information about all available currencies.

### HTTP Request

`GET https://alp.com/api/v1/currencies/`


# Pairs

## List all pairs


```python
import requests

response = requests.get('https://alp.com/api/v1/pairs/')
```

```javascript
var request = require('request');

request.get('https://alp.com/api/v1/pairs/', function (error, response, body) {
    // process response
});
```

> Sample output

```json
[
  {
    "name": "BTC_USD",
    "currency1": "BTC",
    "currency2": "USD",
    "price_precision": 2,
    "amount_precision": 6,
    "maximum_order_size": 100000000.00000000,
    "minimum_order_size": 0.00000001,
    "liquidity_type": 10
   }
]
```

Returns information about all available pairs.

### HTTP Request

`GET https://alp.com/api/v1/pairs/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency1 | all | Filter by first currency.
currency2 | all | Filter by second currency.


# Tickers



```python
import requests

response = requests.get('https://alp.com/api/v1/ticker/')
```

```javascript
var request = require('request');

request.get('https://alp.com/api/v1/ticker/', function (error, response, body) {
    // process response
});
```

> Sample output

```json
[
  {
    "timestamp": 1642511387.55049,
    "pair": "ETH_BTC",
    "last": 0.076,
    "diff": 0,
    "vol": 0,
    "high": 0,
    "low": 0,
    "buy": 0.071346,
    "sell": 0.080317
  }
]
```

Used to fetch last price changes.

### HTTP Request

`GET https://alp.com/api/v1/ticker/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
pair | all | Filter by pair symbol(e.g. BTC_USD)

# Charts
```python
import requests

params = {
    'limit': 1
    }

response = requests.get('https://alp.com/api/charts/BTC_USD/60/chart', params = params)
```

```javascript
var request = require('request');

var params = {limit: 1};

request.get({
                url: 'https://alp.com/api/charts/BTC_USD/60/chart',
                qs: params
            }, function (error, response, body) {
                // process response
            }
);
```

> Sample output

```json
[
  {
    "volume": 0.262929,
    "high": 912.236,
    "low": 910.086,
    "close": 911.915,
    "time": 1485777600,
    "open": 910.424
   }
]
```

### Timeframes

Value | Description
--------- | -------
5 | 5 minutes
15 | 15 minutes
30 | 30 minutes
60 | 1 hour
240 | 4 hours
D | 1 day

### HTTP Request

`GET https://alp.com/api/charts/<pair_symbol>/<timeframe>/chart`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit | 720 | Limiting results
since | - | Since Timestamp
until | - | Until Timestamp

# Wallets


## List own wallets

```python
import requests

response = requests.get('https://alp.com/api/v1/wallets/', headers = get_auth_headers({}))
```


```javascript
var request = require('request');

request.get(
    'https://alp.com/api/v1/wallets/',
    {
        headers: getAuthHeaders({})
    },
    function (error, response, body) {
        // process response
    }
);
```

> Sample output

```json
[
  {
    "currency": "BTC",
    "balance": 0.00000000,
    "reserve": 0.00000000
   }
]
```

Returns information about all wallets of account.

### HTTP Request

`GET https://alp.com/api/v1/wallets/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency_id | all | Filter by currency.

<aside class="warning">
This method requires authorization.
</aside>


# Orders

## Order statuses

Value | Status name | Description
--------- |  ----------- | -----------
1 | Active | Order in queue for executing
2 | Canceled | Order not active, permanently
3 | Done | Order fully executed


## Get orderbook

```python
import requests

params = {
    limit_bids: 1,
    limit_asks: 1,
}

response = requests.get('https://alp.com/api/v1/orderbook/BTC_USD', params=params)
```

```javascript
var request = require('request');

var url = 'https://alp.com/api/v1/orderbook/BTC_USD';

var params = {
    limit_bids: 1,
    limit_asks: 1
};

request.get({url: url, qs: params}, function (error, response, body) {
        // process response
    }
);
```

> Sample output

```json
{
  "sell": [
    {
      "price": 911.519,
      "amount": 0.000446
     }
   ],
  "buy": [
    {
      "price": 911.122,
      "amount": 0.001233
    }
  ]
}
```

Get full orderbook of **active** orders

### HTTP Request

`GET https://alp.com/api/v1/orderbook/<pair_name>`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit_sell | all | Sell orders limit
limit_buy | all | Buy orders limit
group | 0 | If set 1, then order will grouped by price


## List own orders

```python
import requests

response = requests.get('https://alp.com/api/v1/orders/own/', headers = get_auth_headers({}))
```

```javascript
var request = require('request');

request.get('https://alp.com/api/v1/orders/own/', {headers: getAuthHeaders({})}, function (error, response, body) {
    // process response
});
```

> Sample output

```json
[
  {
    "amount": 0.500000000,
    "pair": "ETH_USD",
    "type": "buy",
    "status": 3,
    "price": 0.00113000,
    "id": 11249
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

List all orders created in your account. Can be filtered by query parameters.

### HTTP Request

`GET https://alp.com/api/v1/orders/own/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
type | all | Filter by orders type (sell or buy).
pair | none | Filter by pair.
status | all | Filter by status.
limit | 2000 | Limiting results.

## Retrieve single order


```python
import requests

oid = 53189

response = requests.get('https://alp.com/api/v1/order/{}/'.format(oid))
```

```javascript
var request = require('request');

var oid = 53189;

request.get('https://alp.com/api/v1/order/' + oid + '/', function (error, response, body) {
    // process response
});
```

> Sample output

```json
{
  "id": 11259,
  "date": 1478025717394,
  "pair": "BTC_USD",
  "type": "buy",
  "price": 870.69000000,
  "amount": 0.1250000,
  "amount_filled": "0.00419580",
  "amount_original": 0.0041958,
  "status": "1"
}
```

Get detailed information about order by its id

### HTTP Request

`GET https://alp.com/api/v1/order/<pk>/`


## Create order


```python
import requests

order = {
    type: 'buy',
    pair: 'BTC_USD',
    amount: '1.0',
    price: '870.69'
}

response = requests.post('https://alp.com/api/v1/order/', data = order, headers = get_auth_headers(order))
```

```javascript
var request = require('request');

var order = {
    type: 'buy',
    pair: 'BTC_USD',
    amount: '1.0',
    price: '870.69'
};

request.post({
                url: 'https://alp.com/api/v1/order/',
                form: order,
                headers: getAuthHeaders(order)
            }, function (error, response, body) {
                // process response
                // save order id, check if it's executed
            }
);
```

> Sample output

```json
{
  "success": true,
  "type": "buy",
  "date": 1483721079.51632,
  "oid": 11268,
  "price": 870.69000000,
  "amount": 0.00000000,
  "trades": [
    {
      "type": "sell",
      "price": 870.69000000,
      "o_id": 11266,
      "amount": 0.00010000,
      "tid": 6049
    }
  ]
}
```

<aside class="warning">
This method requires authorization.
</aside>

Create new order that will be automatically executed.

### HTTP Request

`POST https://alp.com/api/v1/order/`

### POST params

Parameter | Description
--------- | -----------
type | Type of order (`sell` or `buy`).
pair | Pair of order
amount | Amount of first currency of pair.
price | Price of order. This param have limited precision (See [pairs](#pairs)).


## Cancel order


```python
import requests

data = {
    order: 63568
}

response = requests.post('https://alp.com/api/v1/order-cancel/', data = data, headers = get_auth_headers(data))
```

```javascript
var request = require('request');

var data = {
    order: 63568
};

request.post({
        url: 'https://alp.com/api/v1/order-cancel/',
        form: data,
        headers: getAuthHeaders(data)
    }, function (error, response, body) {
        // process response
    }
);
```

> Sample output

```json
{
  "order": 63568
}
```

<aside class="warning">
This method requires authorization.
</aside>

Cancel your **active** order. If order not exists, it's done or cancelled error message will be returned.

### HTTP Request

`POST https://alp.com/api/v1/order-cancel/`

### POST params

Parameter | Description
--------- | -----------
order | ID of order to cancel


# Exchanges


## List all exchanges


```python
import requests

response = requests.get('https://alp.com/api/v1/exchanges/')
```

```javascript
var request = require('request');

request.get('https://alp.com/api/v1/exchanges/', function (error, response, body) {
    // process response
});
```

> Sample output

```json
[
  {
    "id": 6030,
    "account_id": 384360,
    "price": 839.36000000,
    "pair": "BTC_USD",
    "type": "sell",
    "timestamp": 1483705817.735508,
    "amount": 0.00281167
  }
]
```


### HTTP Request

`GET https://alp.com/api/v1/exchanges/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
pair | all | Filter by pair.
limit | 100 | Limiting results.
offset | 0 | Skip number of records.
ordering | -id | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting.


## List own exchanges

<aside class="warning">
This method requires authorization.
</aside>

List only own exchanges where your account was buyer or seller.

### HTTP Request

`GET https://alp.com/api/v1/exchanges/own/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
type | all | Filter by orders type (sell or buy).
pair | all | Filter by pair.
limit | 100 | Limiting results.
offset | 0 | Skip number of records.
ordering | -id | Ordering parameter. `id` - ascending sorting, `-id` - descending sorting.


# Deposits


## List own deposits

```python
import requests

response = requests.get('https://alp.com/api/v1/deposits/', headers = get_auth_headers({}))
```

```javascript
var request = require('request');

request.get({
                url: 'https://alp.com/api/v1/deposits/',
                headers: getAuthHeaders({})
            }, function (error, response, body) {
                // process response
            }
);
```

```json
[
  {
    "timestamp": 1485363039.18359,
    "id": 317,
    "currency": "BTC",
    "amount": 530.00000000
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

List your deposits

### HTTP Request

`GET https://alp.com/api/v1/deposits/`


# Withdraws

## Withdraw statuses

Value | Status name | Description
--------- | ----------- | -----------
10 | New | Withdraw created, verification need
20 | Verified | Withdraw verified, waiting for approving
30 | Approved | Approved by moderator
40 | Refused | Refused by moderator. See your email for more details
50 | Canceled | Cancelled by user

## List own made withdraws

```python
import requests

response = requests.get('https://alp.com/api/v1/withdraws/', headers: get_auth_headers({}))
```

```javascript
var request = require('request');

request.get({
        url: 'https://alp.com/api/v1/withdraws/',
        headers: getAuthHeaders({})
    }, function (error, response, body) {
        // process response
    }
);
```

> Sample output

```json
[
  {
    "id": 403,
    "timestamp": 1485363466.868539,
    "currency": "BTC",
    "amount": 0.53000000,
    "status": 20
  }
]
```

<aside class="warning">
This method requires authorization.
</aside>

Get list of your withdraws

### HTTP Request

`GET https://alp.com/api/v1/withdraws/`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
currency_id | all | Filter currency.
status | all | Filter by status.

# API v3

# *Private API*

## Auth
Authentication keys must be provided for each endpoint call.
You need to include valid `api-secret` and `api-key` you have generated for your account.
> **Warning**
>
> functionality are in progress

## Fee Info

`GET /api/v3/feeinfo`

Response

`200` -
```json
[{
  "name": "account name",
  "account_id": 25,
  "is_active": true,
  "currency": "USDT", #may be empty
  "pair": "BTC_USDT", #may be empty
  "taker_fee_rate": 0.02,
  "maker_fee_rate": 0.01,
  "fee_currency": "USDT",
  "priority": 1
}...]
```
`401`

## Orders

### GET

`GET /api/v3/orders`

Query Parameters

| Parameter  | Type    | Description|
|------------|---------| -----------|
| pair       | string  | pair symbol like `BTC_USDT`|
| open_only  | bool    | if set and true returns open order only|
| side       | string  | if set must be rather `BUY` or `SELL`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
|  end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 76575,
  "account_id": 25,
  "pair": "BTC_USDT",
  "wallet_type": "SPOT",
  "side": "BUY",
  "type": "LIMIT",
  "tif": "GTC",
  "price": 20450.05,
  "stop_price": 0, #used if type is stop
  "amount": 0.05, #amount in base currency
  "amount_filled": 0,
  "amount_cancelled": 0,
  "status": "OPEN",
  "date": "2022-10-06T08:28:06.801064-04:00",
  "updated_at": "2022-10-06T08:28:06.801064-04:00",
  "open_at": "",
  "expire_after": "",
  "done_at": ""
}...]
```
`400`,
`401`

### POST

`POST /api/v3/orders`

Query Body

```json
{
  "account_id": 25,
  "wallet_type": "SPOT",
  "pair": "BTC_USDT",
  "order_side": "SELL",
  "order_type": "LIMIT",
  "tif": "GTC",
  "expire_after": "", #may be empty
  "side_effect": "", #may be empty or AUTO_BORROW, AUTO_REPAY
  "limit_price": 25750.45,
  "base_amount": 0.1,  # required for limit types
  "quote_amount": 0, # allowed for market types only
  "stop_price": 0, # for stop order only
  "stop_operator": "", # for stop order only, may be GTE or LTE
  "client_order_id": "jkndijejnmjn2554562",
  "valid_till": "" #if set matching engine should start processing before
}
```

Response

`201` -
```
34
```
(order_id)

`400`, `401`

### DELETE

`DELETE /api/v3/orders`

Query Body

```json
{
  "order_ids": [1, 5, 8],
  "all": false,
  "pair": ""
}
```
> **Note**
>
> rather `order_ids` or `all`=`true` must be specified. If `all`=`true`, `pair` can be specified to cancel all orders for one specific pair

Response

`202`,
`204` (no orders to cancels by specified params),
`400`, `401`

## Accounts

### GET

`GET /api/v3/accounts`

Query Parameters

Parameter | Type    | Description
--------- |---------| -----------
include_subaccounts | bool | if set true, all subaccounts for account are returned as well

Response

`200` -
```json
[{
  "id": 25,
  "name": "account name",
  "email": "test_user_email@gmail.com",
  "id_parent": 0, #if it is subbaccount, parent id will be set
}...]
```
`401`

## Account Trades

### GET

`GET /api/v3/accounts/trades`

Query Parameters


Parameter | Type    |   Description
--------- |---------| ----------------------
pair | string  | pair symbol like `BTC_USDT`
side | string  | if set must be rather `BUY` or `SELL`
start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`
end_time | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "date": "2022-10-06T08:28:06.801064-04:00",
  "status": "FILLED", #status for account order set after trade
  "side": "BUY",
  "type": "LIMIT",
  "price": 23400.34,
  "amount": 0.05,
  "amount_unfilled": 0,
  "fee_rate": 0.001,
  "fee_amount": 1.170017,
  "is_maker": true,
  "account_id": 25,
  "currency": "USDT", #may be empty
  "pair": "BTC_USDT",
  "fee_currency": "USDT",
  "wallet_type": "SPOT"
}...]
```
`400`, `401`



## Account Balances

### GET

`GET /api/v3/accounts/balances`

Response

`200` -
```json
[{
  "account_id": 25,
  "balance": 2000.0,
  "reserve": 500.0, #locked in orders
  "currency": "USDT",
  "pair": "BTC_USDT", #may be empty
  "wallet_type": "SPOT",
  "last_update_at": "2022-10-06T08:28:06.801064-04:00"
}...]
```
`401`


## Wallet Activity

### GET

`GET /api/v3/wallets/activity`

Query Parameters

| Parameter  | Type    |   Description|
|------------|---------| ----------------------|
| currency   | string  | currency symbol like `BTC`|
| side       | string  | if set must be rather `BUY` or `SELL`|
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00`|
|  end_time  | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`|

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 12,
  "account_id": 25,
  "currency": "USDT",
  "pair": "USDT_BTC",
  "wallet_id": 3,
  "date": "2022-10-06T08:28:06.801064-04:00",
  "type": "MOTION_TYPE_ORDER",
  "balance": 10000.0,
  "balance_delta": 1500.0,
  "reserve": 500.0,
  "reserve_delta": -1500.0,
  "balance_end": 9500.0,
  "reserve_end": 500.0,
  "wallet_type": "SPOT"
}...]
```
`400`, `401`



## Margin

### GET

`GET /api/v3/margin`

Query Parameters

| Parameter  | Type | Description|
|------------|--| -----------|
| isolated_margin | bool |  if set true isolated margin positions are returned |
| cross_margin   | bool |  if set true cross margin positions are returned |

> **Note**
>
> rather `isolated_margin` or `cross_margin` must be set true

Response

`200` -
```json
[{
  "id": 38237,
  "account_id": 25,
  "wallet_type": "ISOLATED_MARGIN",
  "interest": 0.003,
  "currecny": "ETH",
  "borrowed": 0.3,
  "pair": "BTC_ETH", #used for ISOLATED_MARGIN only
}...]
```
`400`,
`401`

### DELETE

`DELETE /api/v3/margin`

Query Body

```json
{
  "pair": "BTC_USDT",
  "wallet_type": "ISOLATED_MARGIN"
}
```

> **Note**
>
> rather `ISOLATED_MARGIN` or `CROSS_MARGIN` must be set to `wallet_type`


Response

`202`, `400`, `401`


## Margin Borrow

### GET

`GET /api/v3/margin/borrow`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| open_only  | bool    | if set and true returns open order only                     |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 3636,
  "date": "2022-10-06T08:28:06.801064-04:00",
  "type": "LOAN_MANUAL",
  "wallet_type": "ISOLATED_MARGIN",
  "opened_at": "2022-10-06T08:28:06.801064-04:00",
  "closed_at": "", #set on borrow closing
  "borrowed": 0.03,
  "repaid": 0,
  "interest_rate": 0.0001,
  "interest_accrued": 0.0002,
  "interest_paid": 0.0002,
  "account_id": 25,
  "currency": "ETH",
  "pair": "BTC_ETH", # empty for CROSS_MARGIN
  "last_interest_at": "2022-10-06T10:28:06.801064-04:00"
}...]
```
`400`,
`401`

### POST

`POST /api/v3/margin/borrow`

Query Body

```json
{
  "currency": "ETH",
  "pair": "", #may be empty for CROSS_MARGIN
  "wallet_type": "CROSS_MARGIN",
  "type": "LOAN_MANUAL", #or LOAN_AUTO
  "borrow": 0.3
}
```

Response

`200`, `400`, `401`

## Margin Transfer

### GET

`GET /api/v3/margin/transfer`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 30930,
  "account_id": 25,
  "wallet_type": "ISOLATED_MARGIN",
  "date": "2022-10-06T08:28:06.801064-04:00",
  "direction": "ADD", #ADD or SUB
  "currency": "ETH",
  "pair": "ETH_BTC",
  "amount": 0.02,
  "note": "test isolated margin transfer"
}...]
```
`400`,
`401`

### POST

`POST /api/v3/margin/transfer`

Query Body

```json
{
  "wallet_type": "CROSS_MARGIN",
  "direction": "SUB", #ADD or SUB
  "currency": "ETH",
  "amount": 0.02,
  "pair": "", #must be set for ISOLATED_MARGIN
  "note": "test transfer note"
}
```

Response

`200`, `400`, `401`

## Margin Repay

### GET

`GET /api/v3/margin/repay`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 30930,
  "account_id": 25,
  "wallet_type": "ISOLATED_MARGIN",
  "date": "2022-10-06T08:28:06.801064-04:00",
  "type": "LOAN_MANUAL", #LOAN_MANUAL or LOAN_AUTO
  "currency": "ETH",
  "pair": "ETH_BTC", #empty for CROSS_MARGIN
  "principal_amount": 0.02,
  "interest_amount": 0.0005,
  "borrow_id": 283
}...]
```
`400`,
`401`

### POST

`POST /api/v3/margin/repay`

Query Body

```json
{
  "wallet_type": "CROSS_MARGIN",
  "type": "LOAN_MANUAL",
  "currency": "ETH",
  "amount": 0.02,
  "pair": "" #must be set for ISOLATED_MARGIN
}
```

Response

`200`, `400`, `401`


## Margin Interest

### GET

`GET /api/v3/margin/interest`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
```json
[{
  "id": 32864,
  "account_id": 25,
  "wallet_type": "ISOLATED_MARGIN",
  "date": "2022-10-06T08:28:06.801064-04:00",
  "currency": "ETH",
  "pair": "ETH_BTC", #empty for CROSS_MARGIN
  "interest_rate": 0.00023,
  "borrow_id": 234,
  "interest_accrued": 0.0023,
  "interest_paid": 0.0046
}...]
```
`400`,
`401`



## Margin Liquidation

### GET

`GET /api/v3/margin/liquidation`

Query Parameters

| Parameter  | Type    | Description                                                 |
|------------|---------|-------------------------------------------------------------|
| pair       | string  | pair symbol like `BTC_USDT`                                 |
| currency   | string  | currency symbol like `BTC`                                  |
| open_only  | bool    | if set and true returns open order only                     |
| start_time | string  | start time filter (>=), format: `2006-01-02T15:04:05Z07:00` |
| end_time   | string  | start time filter (<), format: `2006-01-02T15:04:05Z07:00`  |

> **Note**
>
> `start_time` and `end_time` interval must not exceed 30 days. If one is not set 30 days interval from set value is applied. If both are not set, default time interval is 30 days back from today.

Response

`200` -
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
`400`,
`401`


## Wallet Addresses

### POST

`POST /api/v3/wallets/addresses`

> Comming soon

## Wallet Withdraw

### POST

`POST /api/v3/wallets/withdraw`

> Comming soon
