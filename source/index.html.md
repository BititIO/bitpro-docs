---
title: API Reference

language_tabs:
  - shell
  - ruby

search: true
---

# Getting started

### Overview

Bitpro provide a simple and robust REST-ful API to integrate crypto-currency technologies.

The Bitpro api can perform the following operations:

* Create/Manage clients
* Buy for a client
* Witdrawal for a client
* Manage passed transactions


## Bitpro API endpoints

```shell
  curl -X GET https://sandbox.bitit.pro/api/v1/ping
```
```ruby
  HTTParty.get('https://sandbox.bitit.pro/api/v1/ping')
```

  > Example Response

```shell
  {
      "status": "ok",
      "environment": "sandbox"
  }
```
```ruby
  {
      "status" => "ok",
      "environment" => "sandbox"
  }
```

Bitpro as 2 separates environments available for development and production. For security reasons all the requests are made over HTTPS.


### Production Environment

The Bitpro production endpoint is live and used by partners.

* Production API: `https://bitit.pro/api/v1` **_(no ready yet)_**


### Test/Sandbox Environment

The Bitpro test environment is used by default in our examples. It is entirely separate from Bitpro production and there is no overlap in data and accounts. You will need to create a separate account at sandbox.bitit.pro.

* Test API: `https://sandbox.pro/api/v1`

This environment is connected to the Bitcoin TestNet which you can use [Blockr](http://tbtc.blockr.io/) to navigate. To get some test coins, try a [faucet](http://tpfaucet.appspot.com/) or talk to us.


## Error Handling

All errors follow general REST principles. Included in the body of any error response (e.g. non-200 status code) will be an error object of the form:

> Example json error

```shell
{
    "errors": "Not found"
}
```
```ruby
{
    "errors" => "Not found"
}
```

Parameter | Value
--------- | -----
errors|The detailed description of the errors

# Authentication

Bitpro's authenticaction is via the combination of the `Authorization`, `Content-MD5` and `Date` in order to provide a strong authentication without the possibility of replicater requests.

> Api/Secret

```shell
  API_KEY="M3175465"
  SECRET_KEY="TkLPjz2dzzyoMZdkJqM1qLusyTizYPl0IeX7vHi+17+mlYEjyfnLIgqopAQ8Bnk2QnwcSYa4VuXY384EnGi3tA=="
```
```ruby
  api_key = "M3175465"
  secret_key = "TkLPjz2dzzyoMZdkJqM1qLusyTizYPl0IeX7vHi+17+mlYEjyfnLIgqopAQ8Bnk2QnwcSYa4VuXY384EnGi3tA=="
```

A Merchant will need an `access_key` and a `secret_key`, thoses key are supplied by Bitit, if you need to generate a keypair, please [reach us](http://support.bitit.gift/hc/en-us/requests/new).

Bitpro API use [HMAC encryption](https://github.com/diafygi/webcrypto-examples#hmac) as Authorization on all our endpoints.

> Date

```shell
  DATE=`env TZ=GMT date '+%a, %d %b %Y %H:%M:%S %Z'`
  #Mon, 10 Apr 2017 04:43:12 GMT
```

```ruby
  date = Time.now.utc.httpdate
  #Mon, 10 Apr 2017 04:43:12 GMT
```

1. The `Date` header is always on GMT timezone with the following format `Mon, 10 Apr 2017 04:43:12 GMT`

> Content-MD5

```shell
  PARAMS="address=2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr&amount=0.12"

  CONTENT_MD5=`echo -n $PARAMS | openssl md5 -binary | base64`
  #FBgiRcA+lHYYgLkAc2JX5g==
```
```ruby
  params = {
    address: '2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr',
    amount: 0.12
  }

  encoded_params = URI.encode_www_form(params)
  #"address=2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr&amount=0.12"

  content_md5 = Digest::MD5.base64digest(encoded_params)
  #FBgiRcA+lHYYgLkAc2JX5g==
```

2. The `Content-MD5` header is a MD5 base64 digest of the url encoded params

> Signature

```shell
  METHOD="POST"
  URI="/api/v1/clients/BT-C962387/withdrawal"

  CANONICAL_STRING="$METHOD,$CONTENT_MD5,$URI,$DATE"
  #POST,FBgiRcA+lHYYgLkAc2JX5g==,/api/v1/clients/BT-C962387/withdrawal,Mon, 10 Apr 2017 04:43:12 GMT

  SIGNATURE=`echo -n $CANONICAL_STRING | openssl sha1 -hmac $SECRET_KEY -binary | base64`
  #W81e6C9NeGYqau1RWhyGtw8YfoY=
```
```ruby
  method = "POST"
  uri = "/api/v1/clients/BT-C962387/withdrawal"

  canonical_string = [method, content_md5, uri, date].join(',')
  #"POST,FBgiRcA+lHYYgLkAc2JX5g==,/api/v1/clients/BT-C962387/withdrawal,Mon, 10 Apr 2017 04:43:12 GMT"

  digest = OpenSSL::Digest.new('sha1')
  signature = Base64.strict_encode64(OpenSSL::HMAC.digest(digest, secret_key, canonical_string))
  #W81e6C9NeGYqau1RWhyGtw8YfoY=
```

3. The 'Authorization' is a string with the format ```Bitpro api_key:signature```

The signature is a base64 HMAC-SHA-1 of a canonical string the following format ```method,content_md5,uri,timestamp```

* the `method` in capitalized letters (GET, POST)
* the previously calculated `content_md5`
* the `uri` of the request
* the previously calculated `date` as a string

all separeted with commas (,)


# Rates

Rate are the current price of BTC in USD

```bash
  curl -X GET https://sandbox.bitit.pro/api/v1/rates
```

> Response

```bash
  {
    "time_expires": 1491824457.468,
    "buy_price": "1225.10",
    "asset": "XBT"
  }
```

### HTTP Request

`GET /api/v1/rates`

### Response

Returns the current price for BTC in USD.

Parameter | Type | Descritption
--------- | ---- | ------------
time_expires|Time|Timestamp before expiration of the current price
buy_price|Number|Price in USD for 1 unit
asset|String|Name of the asset


# Clients

  Clients are the customers of the Merchant (you).
  Each client have his own unique id (UID) and complementary informations like balances or bitcoin address.

## Create a new client

Create a new client

```shell
  curl -X POST \
    -d "$PARAMS" \
    -H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
    -H "Content-MD5: $CONTENT_MD5" \
    -H "Date: $DATE" \
    "https://sandbox.bitit.pro/api/v1/clients"
```
```ruby
  headers = {
        'Authorization' => "Bitpro #{api_key}:#{signature}",
        'Content-MD5' => content_md5,
        'Date' => date
      }

      HTTParty.post('https://sandbox.bitit.pro/api/v1/clients', body: {}, headers: headers)
    end
```
> Response

```shell
  {
    "uid": "BT-C798980",
    "bitcoin_address": "2NCJCpBM7ET6oExzzKfWBnFPq8h2Ni2k7Hk",
    "balance": "0.0",
    "unconfirmed_balance": "0.0"
  }
```
```ruby
  {
    "uid" => "BT-C798980",
    "bitcoin_address" => "2NCJCpBM7ET6oExzzKfWBnFPq8h2Ni2k7Hk",
    "balance" => "0.0",
    "unconfirmed_balance" => "0.0"
  }
```

### HTTP Request

`POST /api/v1/clients`

### BODY Parameters

None.

### Response

Returns a Client Model object.

Parameter | Type | Descritption
--------- | ---- | ------------
uid|String|Unique client id
bitcoin_address|String|Client public bitcoin deposit address
balance|Number|Confirmed bitcoin balance
unconfirmed_balance|Number|Unconfirmed bitcoin balance (waiting for validations)


## Get client

Retrieve an existing client

```shell
  curl -X GET \
    -d "$PARAMS" \
    -H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
    -H "Content-MD5: $CONTENT_MD5" \
    -H "Date: $DATE" \
    "https://sandbox.bitit.pro/api/v1/clients/BT-C798980"
```
```ruby
  headers = {
        'Authorization' => "Bitpro #{api_key}:#{signature}",
        'Content-MD5' => content_md5,
        'Date' => date
      }

      HTTParty.get('https://sandbox.bitit.pro/api/v1/clients/BT-C798980', body: {}, headers: headers)
    end
```
> Response

```shell
  {
    "uid": "BT-C798980",
    "bitcoin_address": "2NCJCpBM7ET6oExzzKfWBnFPq8h2Ni2k7Hk",
    "balance": "0.0",
    "unconfirmed_balance": "0.0"
  }
```
```ruby
  {
    "uid" => "BT-C798980",
    "bitcoin_address" => "2NCJCpBM7ET6oExzzKfWBnFPq8h2Ni2k7Hk",
    "balance" => "0.0",
    "unconfirmed_balance" => "0.0"
  }
```

### HTTP Request

`GET /api/v1/clients/:uid`

### URL Parameters

None.

### Response

Returns a Client Model object.

Parameter | Type | Descritption
--------- | ---- | ------------
uid|String|Unique client id
bitcoin_address|String|Client public bitcoin deposit address
balance|Number|Confirmed bitcoin balance
unconfirmed_balance|Number|Unconfirmed bitcoin balance (waiting for validations)


# Account Operations

Each client can do different account operationts like buy, deposit, withdrawal. Each action requieres specific parameters.

## Buy

Buy a USD (United State Dollar) of bitcoin at the current rate

```shell
  curl -X POST \
    -d "$PARAMS" \
    -H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
    -H "Content-MD5: $CONTENT_MD5" \
    -H "Date: $DATE" \
    "https://sandbox.bitit.pro/api/v1/clients/BT-C798980/buy"
```

> Response

```shell
  {
    "tid": "22000700",
    "amount": "1.22779733",
    "created_at": "2017-04-10T08:52:08Z"
  }
```

### HTTP Request

`POST /api/v1/clients/:uid/buy`

### URL Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
uid|String|YES|UID of the Client

### BODY Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
traded_fiat|Number|YES|Amount of BTC to buy in USD

### Response

Returns informations relative to the trade.

Parameter | Type | Descritption
--------- | ---- | ------------
tid|String|Unique transaction id
amount|Number|Amount of BTC bought
created_at|Date|Date of the trade

### Errors

Response | Description
--------- | -----
Not found|The uid of the client dosen't exist for this merchant
Traded fiat must be greater than 0|Traded fiat amount missing

## Withdrawal

Client can withdrawal an amount of bitcoin to an external bitcoin address

```shell
  curl -X POST \
    -d "$PARAMS" \
    -H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
    -H "Content-MD5: $CONTENT_MD5" \
    -H "Date: $DATE" \
    "https://sandbox.bitit.pro/api/v1/clients/BT-C798980/withdrawal"
```

> Response

```shell
  {
    "tid": "39486536",
    "amount": "0.01",
    "created_at": "2017-04-10T10:14:12Z",
    "tx_id": "7518e03ffd987865aa6c7e456e3796900f7ee8a7cafe57c24108aba16d19d0d4",
    "address": "2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr",
    "state": "processed"
    }
```

### HTTP Request

`POST /api/v1/clients/:uid/withdrawal`

### URL Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
uid|String|YES|UID of the Client

### BODY Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
address|String|YES|A valid external bitcoin address
amount|Number|YES|Amount of BTC to withdrawal

### Response

Returns informations relative to the withdrawal.

Parameter | Type | Descritption
--------- | ---- | ------------
tid|String|Unique transaction id
amount|Number|Amount of BTC withdrawal
created_at|Date|Date of the withdrawal
tx_id|String|Bitcoin transaction id
address|String|Bitcoin destination address
state|String|State of the withdrawal (pending|accepted)

### Errors

Response | Description
--------- | -----
Not found|The uid of the client dosen't exist for this merchant
Amount is greater than your available balance|Amount the client try to withdrawal is greater than the confirmed balance
Address can't be blank|Bitcoin destination address is missing
Address is not a valid bitcoin address|Bitcoin destination address is wrong
Amount should not be smaller than 0.05 BTC|Amount is too small


## Get transactions

All transactions of a client per page of 25

```bash
curl -X GET \
-d "$PARAMS" \
-H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
-H "Content-MD5: $CONTENT_MD5" \
-H "Date: $DATE" \
"https://sandbox.bitit.pro/api/v1/clients/BT-C798980/transactions"
```

> Response

```bash
[
  {
    "tid": "39486536",
    "type": "bitcoin_withdrawal",
    "amount": "0.01",
    "created_at": "2017-04-10T10:14:12Z",
    "tx_id": "7518e03ffd987865aa6c7e456e3796900f7ee8a7cafe57c24108aba16d19d0d4",
    "address": "2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr",
    "state": "processed"
  },
  {
    "tid": "35728038",
    "type": "bitcoin_buy",
    "amount": "1.28949065",
    "created_at": "2017-04-06T06:21:52Z"
  },
  {
    "tid": "3501712",
    "type": "bitcoin_deposit",
    "amount": "0.1",
    "created_at": "2017-04-06T06:16:19Z",
    "tx_id": "6086da4618f9cdd1e79e83696a13d5f90b8e1f207d7d74997a8c98b3669e43b0",
    "tx_confirmations":6
  },
  #....
]
```

### HTTP Request

`GET /api/v1/clients/:uid/transactions`

### URL Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
uid|String|YES|UID of the Client
page|Number|No|Page number (default: 1)


### Response

Returns a list of Transction Model objects.

Parameter | Type | Descritption
--------- | ---- | ------------
tid|String|Unique transaction id
type|String|type of the transaction (bitcoin_withdrawal|bitcoin_buy|bitcoin_deposit)
amount|Number|Amount of the transaction (Can be negative)
created_at|Date|Date of the withdrawal
tx_id|String|Bitcoin transaction id
tx_confirmations|Number|Number of confirmations for a Deposit
address|String|Bitcoin destination address
state|String|State of the withdrawal (pending|accepted)


## Get transaction

Get only one specific transaction

```bash
curl -X GET \
-d "$PARAMS" \
-H "Authorization: Bitpro $API_KEY:$SIGNATURE" \
-H "Content-MD5: $CONTENT_MD5" \
-H "Date: $DATE" \
"https://sandbox.bitit.pro/api/v1/clients/BT-C499471/transactions/39486536"
```

> Response

```bash
{
    "tid": "39486536",
    "type": "bitcoin_withdrawal",
    "amount": "0.01",
    "created_at": "2017-04-10T10:14:12Z",
    "tx_id": "7518e03ffd987865aa6c7e456e3796900f7ee8a7cafe57c24108aba16d19d0d4",
    "address": "2MzCEdXEzBaxCJQAq8igw4pUP1NSGAgzsUr",
    "state": "processed"
  }
```

### HTTP Request

`GET /api/v1/clients/:uid/transactions/:tid`

### URL Parameters

Parameter | Type | Required | Descritption
--------- | ---- | -------- | ------------
uid|String|YES|UID of the Client
tid|String|YES|TID of the Transaction


### Response

Returns a Transaction Model object.

Parameter | Type | Descritption
--------- | ---- | ------------
tid|String|Unique transaction id
type|String|type of the transaction (bitcoin_withdrawal|bitcoin_buy|bitcoin_deposit)
amount|Number|Amount of the transaction (Can be negative)
created_at|Date|Date of the withdrawal
tx_id|String|Bitcoin transaction id
tx_confirmations|Number|Number of confirmations for a Deposit
address|String|Bitcoin destination address
state|String|State of the withdrawal (pending|accepted)
