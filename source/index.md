---
title: HoneyLedger API Reference

toc_footers:
  - <a href='http://stage.honeyledger.com/'>HoneyLedger Staging Site</a>

includes:
  - errors

search: true
---

# Introduction

<aside class="notice">
The HoneyLedger API is still under active development and may change often. We highly recommend signing up to the <a href='http://eepurl.com/bIMS_f'>HoneyLedger API Developers Email List</a> and receive emails when we make changes to the API.
<br><br>This document will always have the most up-to-date documentation on our API.
</aside>

The HoneyLedger API is intended for third-party developers to build their own notification and payment systems on top of HoneyLedger.

The typical use case is for a third-party application to create and track a payment on behalf of one of their users.

# API Keys

The HoneyLedger API is currently open to interested developers, but there isn't a formal signup process yet. If you do not currently have a HoneyLedger `app_id` and `app_secret`, email us at developers@honeyledger.com and we can get you started.

# Application Flow

Using the HoneyLedger API to build and track payments is a simple process:

1. Create a HoneyLedger Transaction ID (`honeyledger_txid`) using the `/create` API call.
2. Redirect your user to the appropriate HoneyLedger Payment Page with their `honeyledger_txid` for checkout.
3. Receive a webhook notification when the payment has been posted and verify it with a `/status` API call.

All of the HoneyLedger API data is associated with a `honeyledger_txid`, which is a unique transaction ID associated with your `app_id`.

For most users, you will most likely be creating a payment form on your website or app, gathering the required information on that page, then creating (and storing) the `honeyledger_txid` request on your side before redirecting the user to checkout with HoneyLedger.

# Data Formats

The HoneyLedger API will always return data in JSON format.

Data should be passed to the HoneyLedger API in JSON format, using an HTTP POST request with `Content-Type: application/json` in the request header.

# Staging & Production APIs

When developing your application, use the staging environment. Staging is code-identical to production, except that no real money is transferred, and "payments" are processed near-instantly.

**STAGING URLs**:

* API: `https://api-stage.honeyledger.com/v1/{api_call}`
* Payment Pages: `https://pay-stage.honeyledger.com/{honeyledger_txid}`
* Website: `https://stage.honeyledger.com`

**PRODUCTION**:

* API: `https://api.honeyledger.com/v1/{api_call}`
* Checkout Pages: `https://pay.honeyledger.com/{honeyledger_txid}`
* Website: `https://honeyledger.com`

# API Calls

## /create

> Example JSON request:

```json
{
    "app_id":"your_app_id",
    "app_secret":"your_app_secret",
    "user_token":"FDB6D6750D5F1832A6452D86A4094F5D19B72F1A",
    "from_name":"Kappa",
    "app_fee":1.00,
    "amount":5.00
}
```

> Example request using curl:

```shell

curl -H "Content-Type: application/json" \
     -X POST \
     -d '{"app_id":"your_app_id","app_secret":"your_app_secret","user_token":"FDB6D6750D5F1832A6452D86A4094F5D19B72F1A","from_name":"Kappa","app_fee":1.00,"amount":5.00}' \
     http://api-stage.honeyledger.com/v1/create
```

> The above call will return JSON structured like this:


```json
{
  "honeyledger_txid": "C554001FDF0604EE",
  "expire_time": "1449446664"
}
```


The create API call generates a HoneyLedger Transaction ID for a 3rd party application. Transaction IDs are valid for 1 hour from generation, at which point they will be automatically set to expired.

### API Call

`/v1/create`

### API Arguments

Parameter | Required | Default | Type | Description
--------- | -------- | ------- | ---- | -----------
app\_id | **Yes** |  | int(10) | Your Application ID
app\_secret | **Yes** |  | string(255) | Your Application Secret
user\_token | **Yes** |  | string(40) | A token to represent the user that will be receiving a payment. This token is given to a user when they finish signing up to accept payments at HoneyLedger.
from\_name | No | Twitch username | string(255) | The name (or pseudonym) of the user who is trying to donate. Overrides their “actual” Twitch username for anonymous/pseudonymous donations
amount | **Yes** |  | decimal(14,2) | The amount a user is attempting to donate
app\_fee | No | 0.00 | decimal(4,2) | The percentage (1.00 = 1%) of the total amount to be collected by your application. NOTE: Your app\_fee cannot be higher than 15%.
webhook\_url | No | null | string(255) | The URL that will receive any webhook notifications when the payment is processed.
success\_url | No | null | string(255) | URL to redirect to upon successful payment
cancel\_url | No | null | string(255) | URL to redirect to upon canceling the payment

### API Response
Name | Type | Description
------------- | ---- | -----------
honeyledger\_txid | string(255) | HoneyLedger Transaction ID for this payment
expire_time | timestamp | Timestamp of when this transaction id expires (UTC). This is always 1 hour after the transaction request was created.

## /status

> Example JSON request:

```json
{
    "app_id":"your_app_id",
    "app_secret":"your_app_secret",
    "honeyledger_txid":"C554001FDF0604EE"
}
```

> Example request using curl:

```shell

curl -H "Content-Type: application/json" \
     -X POST \
     -d '{"app_id":"your_app_id","app_secret":"your_app_secret","honeyledger_txid":"C554001FDF0604EE"}' \
     http://api-stage.honeyledger.com/v1/create
```

> The above call returns JSON structured like this:

```json
{
  "honeyledger_txid": "C554001FDF0604EE",
  "from_name": "Kappa",
  "user_token": "FDB6D6750D5F1832A6452D86A4094F5D19B72F1A",
  "amount": "5.00",
  "state": "new",
  "app_fee": "1.00",
  "expire_time": "1449446664"
}
```

The status API call will look up the current state of a HoneyLedger Transaction ID (`honeyledger_txid`) associated with your `app_id` and `app_secret`.

If using webhooks to track payment status, always use this call after receiving a notification from HoneyLedger.

### API Call

`/v1/status`

### API Arguments

Parameter | Required | Default | Type | Description
--------- | -------- | ------- | ---- | -----------
app\_id | **Yes** |  | int(10) | Your Application ID
app\_secret | **Yes** |  | string(255) | Your Application Secret
honeyledger\_txid | **Yes** | string(255) | HoneyLedger transaction ID you want to look up the status of.

### API Response
Response Name | Type | Description
------------- | ---- | -----------
honeyledger\_txid | string(255) | HoneyLedger transaction ID for this payment
from\_name | string(255) | The name (or pseudonym) of the user who is trying to donate.
user\_token | string(40) | A token to represent the user that will be receiving a payment. 
amount | decimal(14,2) | The amount of this donation
state | string(255) | The state the transaction is in. For simplicity, may be one of `new` `expired` `pending` `failed` `completed`
expire_time | timestamp | Timestamp of when this transaction id expires (UTC). This is always 1 hour after the transaction request was created.

# Webhooks

> An example Webhook callback:

```json
{
    "honeyledger_txid":"C554001FDF0604EE"
}
```

When building an application for production use, webhooks are the quickest way to track the status of a payment.

Setting the `webhook_url` during the `/create` API call will trigger a callback to that URL when the status of the `honeyledger_txid` changes.

By default, webhook callbacks are only sent over when the status of a `honeyledger_txid` has changed to `completed` -- meaning the transaction has been successfully paid. However, we can turn on more notifications if you need them.

