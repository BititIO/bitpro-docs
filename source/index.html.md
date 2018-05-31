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

Before starting you'll need to request an account. You'll receive an email to chose you password.

- Login to https://bitit.pro with you email and password
- Configure your application on settings


## Bitpro API endpoints

Bitpro as 2 separates environments available for test and production.
Both environments use HTTPS to encrypt requests.


### Production Environment

The Bitpro production endpoint is live and used by partners.

* Production API: `https://bitit.pro` **_(no ready yet)_**


### Sandbox Environment

The Bitpro sandbox environment is entirely separated from Bitpro production and there is no overlap in data and accounts. You will need to request a sandbox account.

* Test API: `https://sandbox.bitit.pro`

This environment is connected to the Bitcoin TestNet which you can use [Blockr](http://tbtc.blockr.io/) to navigate. To get some test coins, try a [faucet](http://tpfaucet.appspot.com/) or talk to us.

# Merchant dashboard

Merchant dashboard is available on the following url: [https://bitit.pro](https://bitit.pro).
You will be able to look at your customers transactions, manage your merchant settings (webhook urls, domains authorization, custom logo, custom color) and manage your team.

## Settings

<aside class="notice">
Merchant ID will be requested each time you generate a payment link or query the API. It's your unique merchant ID.
</aside>

<aside class="notice">
  Webhook urls need to be configured in order to notify your system with transaction updates.
  You'll find more informations below
</aside>

<aside class="notice">
  Merchant integration can be customized with a logo or a color, this parameters are available in your dashboard settings page.
</aside>

<aside class="notice">
  Iframe allowed domains have to be whitelisted
</aside>

# Authentication

Bitpro's authentication is implemented on the [HTTP HMAC Spec](https://github.com/acquia/http-hmac-spec)

## Simple shell example

```shell
#!/bin/bash

PRIVATE_KEY=$YOUR_PRIVATE_KEY
ID=$YOUR_MERCHANT_ID

HOST=${YOUR_HOST:-sandbox.bitit.pro}
PATH_INFO=${YOUR_PATH:-/mapi/charges}
QUERY_STRING=${YOUR_QUERY_STRING:-page=1}

NONCE=$(uuidgen | tr 'ABCDEF' 'abcdef')
REALM='Bitpro'
TIME=$(date +%s)

STRING_TO_SIGN=\
"GET
$HOST
$PATH_INFO
$QUERY_STRING
id=${ID}&nonce=${NONCE}&realm=${REALM}&version=2.0
$TIME"

HMAC_KEY=$(echo -n "$PRIVATE_KEY" | openssl base64 -d -A | hexdump -v -e '1/1 "%02x"')
SIGNATURE=$(echo -n "$STRING_TO_SIGN" | openssl dgst -sha256 -mac HMAC -macopt hexkey:$HMAC_KEY -binary | openssl base64 -A)

AUTHORIZATION="Authorization: acquia-http-hmac realm=\"${REALM}\",id=\"${ID}\",nonce=\"${NONCE}\",version=\"2.0\",headers=\"\",signature=\"${SIGNATURE}\""

curl "https://${HOST}${PATH_INFO}?${QUERY_STRING}" \
  -H "$AUTHORIZATION" \
  -H 'Accept: application/vnd.bitpro-mapi-20180503+json' \
  -H "X-Authorization-timestamp: ${TIME}"
```

# Payment integration

Instructions about the integration of Bitpro cryptocurrencies payment solution into a merchant checkout page.

## Prerequisites

Before starting your should have a Bitpro merchant account with a valid `merchant ID`.
You'll find it on [https://bitit.pro/dashboard/merchant](https://bitit.pro/dashboard/merchant)

## Using direct link

This integration will redirect your customer on an external payment page. It can be used as an iframe after whitelisting the authorized domains on your dashboard.

### HTML integration
```html
<a href="https://bitit.pro/app/checkout/:merchant-id?fiat_price_cents=:fiat-price-cents&buyer_email=:buyer-email&lang=:lang&external_order_id=:external-order-id" target="_blank" rel="noopener noreferrer external">
  Pay with Bitcoin
</a>
```

* Url params should be url-encoded
* Only `merchant-id` is mandatory
* Link can be styled as you wish using class or inline css

Attribute | Type | Description| Mandatory
--------- | ---- | -----------|----------
merchant-id|String|Unique merchant id (36 chars long)|mandatory
buyer-email|String|Customer email address|optional
fiat-price-cents|Integer|Product price in cents|optional
fiat-currency|String|Currency ISO code (only `eur` supported)| optional
external-order-id|String|Custom merchant order id (max 200 chars long)|optional
lang|String|Customer language (only `fr` and `en` supported)|optional

## Using Widget

This "in-page" integration of the payment widget rely on iframe technology and allow you to accept payments directly into your page, without the hassle of redirecting your customer on an external service.

### HTML integration
```html
<div>
  <button id="bitit-iframe-trigger"
          data-merchant-id=":merchant-id"
          data-external-order-id=":external-order-id"
          data-fiat-price-cents=":fiat-price-cents"
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
* Trigger element can have the following optionals data attributes: `external-order-id` `fiat-price-cents` `buyer-email` `lang`, `fiat-currency`
* If you would like to offer the possibility to close/remove the iframe, you can add another trigger element with the following id: `bitit-iframe-destroy`

Attribute | Type | Description| Mandatory
--------- | ---- | -----------|----------
merchant-id|String|Unique merchant id (36 chars long)|mandatory
buyer-email|String|Customer email address|optional
fiat-price-cents|Integer|Product price in cents|optional
fiat-currency|String|Currency ISO code (only `eur` supported)| optional
external-order-id|String|Custom merchant order id (max 200 chars long)|optional
lang|String|Customer language (only `fr` and `en` supported)|optional

<aside class="warning">
  Iframe integration requires that you set all the domains using the iframe in your merchant settings. Settings are available on your merchant dashboard.
</aside>

## Webhooks
During a payment life there is different states and you can configure a webhook for each state.
On you dashboard settings you can add as many webhooks as you want by selecting a state and adding the url you want to receive the webhook on.

Different states are:
- `charge:seen` when the cryptocurrency transaction is first seen on the blockchain
- `charge:timedout` when the time expired before the transaction is seen
- `charge:confirmed` when the payment is fully confirmed
- `charge:failed` when the payment failed (wrong amount, errors ...)

```
{
  "delivery_attempt": 1,
  "event": {
              "id": "67c0e853-bb77-46ce-ae20-09011299cb22",
              "type": "charge:seen",
              "occured_at" :"2018-05-10T12:22:35.550Z",
              "data": {
                        "fiat_price":42.0,
                        "expires_at":"2018-05-10T12:32:35Z",
                        "created_at":"2018-05-10T12:12:35Z",
                        "addresses":[
                                      {
                                        "address":"1BTCADDR58375dace00e01c55b5d5a29c2"
                                      }
                                    ],
                        "payments":[
                                      {
                                        "txid":"5ec68db848026b167ceb584f8e0b466601663a9de739951f79cb964aea06fe3e",
                                        "value":"0.494093233",
                                        "state":"confirmed"
                                      }
                        ]
                      }
            }
}
```
