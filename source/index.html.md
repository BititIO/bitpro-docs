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

* Please list operations here
* Operation1
* Operation 2


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

all separated with commas (,)

# Merchant dashboard

Merchant dashboard is available on the following url: [https://bitit.pro/dashboard](https://bitit.pro/dashboard).
You will be able to look at your customers transactions, manage your merchant settings (webhook urls, domains authorization, custom logo, custom color) and manage your team.

<aside class="notice">
  Webhook urls need to be configured in order to notify your system with transaction updates.
</aside>

<aside class="notice">
  Merchant integration can be customized with a logo or a color, this parameters are available in your dashboard settings page.
</aside>

# Payment integration

Instructions about the integration of Bitit Pro cryptocurrencies payment solution into a merchant checkout page.

## Prerequisites

Before starting your should have a Bitit pro merchant account with a valid `merchant_id`.

## Using Widget

This "in-page" integration of the payment widget rely on iframe technology and allow you to accept payments directly into your page, without the hassle of redirecting your customer on an external service.

### HTML integration
```html
<div>
  <button id="bitit-iframe-trigger" 
          data-merchant-id=":merchant-id" 
          data-external-order-id=":external-order-id"
          data-product-price=":item-price" 
          data-buyer-email=":customer-email" 
          data-lang=":customer-language">
    Pay with Bitcoin
  </button>
</div>
<script src="https://bitit.pro/libs/iframeBuilderV1.min.js"></script>
```

* Button tag can be replaced by any html element and styled as you wish using class or inline css
* Trigger element should be wrapped by another HTML element
* Trigger element should have the following id: `bitit-iframe-trigger`
* Trigger element should have the following mandatory data attribute: `merchant-id`
* Trigger element can have the following optionals data attributes: `external-order-id` `product-price` `buyer-email` `lang`
* If you would like to offer the possibility to close/remove the iframe, you can add another trigger element with the following id: `bitit-iframe-destroy`

Attribute | Type | Description
--------- | ---- | -----------
merchant-id|String|Unique merchant id (36 chars long) -- mandatory
product-price|Float|Item price -- optional
external-order-id|String|Custom merchant order id (max 200 chars long) -- optional
buyer-email|String|Customer email address -- optional
lang|String|Customer language (only `fr` and `en` supported) -- optional

<aside class="warning">
  Iframe integration requires that you set all the domains using the iframe in your merchant settings. Settings are available on your merchant dashboard.
</aside>

## Using direct link

This integration will redirect your customer on an external payment page.

### HTML integration
```html
<a href="https://bitit.pro/app/checkout/:merchant-id?fiat_price=:product-price&buyer_email=:buyer-email&lang=:lang&external_order_id=:external-order-id" target="_blank" rel="noopener noreferrer external">
  Pay with Bitcoin
</a>
```

* Url params should be url-encoded
* Only `merchant-id` is mandatory
* Link can be styled as you wish using class or inline css