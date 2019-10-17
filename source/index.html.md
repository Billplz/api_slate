---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - php

toc_footers:
  - <a href='https://www.billplz.com'>Sign Up to Start</a>
  - <a href='https://www.billplz-sandbox.com'>Sign Up to Sandbox</a>
  - <a href='https://www.fb.com/groups/billplzdevjam'>Billplz Developer Community</a>

includes:
  - errors
  - changelogs

search: true
---

# Introduction

> API Endpoint:

```
https://www.billplz.com/api/
```

Welcome to the Billplz API! You can use our API to access Billplz API endpoints to start sending bills and collecting payment.

Our API is organized around REST. JSON will be returned in all responses from the API, including errors. The API will accept both application/x-www-form-urlencoded and application/json.

`V4` API is in active development state where all new features introduced will be added in this version.

<aside class="notice">
  THE MANAGEMENT CANNOT ACCEPT RESPONSIBILITY FOR LOSS OF MONEY DUE TO IMPROPER USE OF BILLPLZ API.
</aside>

# Sandbox Mode

> API Endpoint:

```
https://www.billplz-sandbox.com/api/
```

The Billplz Sandbox mirrors the features found on the real production server, while some of the features do not apply to the Sandbox.

You should test your integrations and know they will behave the same on the production server as they do in the Sandbox environment.

<aside class="notice">
  SANDBOX AND REAL PRODUCTION ARE SEPARATE ACCOUNTS. PLEASE USE YOUR PRODUCTION ACCOUNT IN PRODUCTION ENDPOINT; SANDBOX ACCOUNT IN SANDBOX ENDPOINT.
</aside>

# API Flow

In Billplz, you are interacting with Bill and Collection.

A Collection is a set of Bills. The relationship is, Collection has many Bills. You can choose to create a Collection either from API or within your Billplz's dashboard.

An example of Collection would be something like Tuition Fee for June 2015, Donation for Earthquake, Ticket Payment, and many more.

A Bill represents the promise made to you by your customer. It's an invoice for your customer. A Bill must belong to a Collection.

To start using the API, you would have to create a Collection. Then the payment flow will kick in as per below:

###### NORMAL COMPLETION FLOW
1. Customer visits your site.
1. Customer chooses to make payment.
1. Your site creates a Bill via API call.
1. Billplz API returns Bill's URL.
1. Your site redirects the customer to Bill's URL.
1. The customer makes payment via payment option of choice.
1. Billplz sends a server-side update \*\***at best effort** (Payment Completion) to your site upon payment failure or success. (Basic Callback URL / X Signature Callback URL depending on your configuration) ***[your backend server should capture the transaction update at this point]*** *refer to [API#X Signature Callback Url](#x-signature-callback_url)*.
1. Billplz redirects (Payment Completion) the customer back to your site if `redirect_url` is not empty (Basic Redirect URL / X Signature Redirect URL depending on your configuration)
***[your server should capture the transaction update at this point and give your user an instant reflection on the page loaded]*** *refer to [API#X Signature Redirect Url](#x-signature-redirect_url)*
or,
The customer will see Billplz receipt if `redirect_url` is not present.

###### COMPLETION FLOW WITHOUT REDIRECT_URL
1. Customer visits your site.
1. Customer chooses to make payment.
1. Your site creates a Bill via API call.
1. Billplz API returns Bill's URL.
1. Your site redirects the customer to Bill's URL.
1. The customer makes payment via payment option of choice.
1. Billplz sends a server-side update \*\*at best effort (Payment Completion) to your site upon payment failure or success. (Basic Callback URL / X Signature Callback URL depending on your configuration)
***[your backend server should capture the transaction update at this point]*** *refer to [API#X Signature Callback Url](#x-signature-callback_url)*.
Billplz does not redirects (Payment Completion) the customer back to your site, `redirect_url` (due to many possibilities, app hang, browser closed, disconnected, etc)

<aside class="warning">
  Integration with <code>callback_url</code> <strong>is compulsory</strong>.
</aside>
<aside class="success">
  Use <code>redirect_url</code> for a faster status update and better user experience.
</aside>
<aside class="notice">
  The order of <code>redirect_url</code> and <code>callback_url</code> APIs execution is not in a fixed order. Your system should not rely on the sequence and must be able to prevent duplicate updates.
</aside>

# Authentication

> To test authentication, use this code:

```shell
# With shell, you can just pass the correct HTTP Basic Auth credential with each request.
curl https://www.billplz.com/api/v4/webhook_rank \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:

# Alternatively, you may use header with base64 encoded API Secret Key
curl https://www.billplz.com/api/v4/webhook_rank \
  -H "Authorization: NzNlYjU3ZjAtN2Q0ZS00MmI5LWE1NDQtYWVhYzZlNGIwZjgxOg=="

```

```php
<?php
$host = 'https://www.billplz.com/api/v4/webhook_rank';
$api_key = '73eb57f0-7d4e-42b9-a544-aeac6e4b0f81';
$process = curl_init($host);
curl_setopt($process, CURLOPT_USERPWD, $api_key . ":"); 
$return = curl_exec($process);
echo $return;
```
> If authenticated, you should not see `Unauthorized` response type.

> curl uses the -u flag to pass basic auth credentials (adding a colon after your API key will prevent it from asking you for a password).

You authenticate to the Billplz API by providing your API Secret Keys in the request. You can get your API keys from your account’s settings page.

Authentication to the API occurs via HTTP Basic Auth. Provide your API key as the basic auth username. You do not need to provide a password.

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. You must authenticate all requests.

# Integration

> GitHub:

```
https://github.com/billplz
```

> Knowledgebase:

```
https://help.billplz.com/hc/en-us/articles/360015833913
```

Billplz has been integrated with many systems available in the market and always working to integrate with more platforms.

For a self-hosted system, you may download a production-ready module to integrate with your e-commerce system. Please refer to our [GitHub](https://github.com/billplz) or [Knowledgebase](https://help.billplz.com/hc/en-us/articles/360015833913-List-of-system-with-production-ready-integration) for the list of the available module.

## Xero

Billplz integration with XERO has been deprecated and pending removal.

## OAuth 2.0

Billplz provides a solution for a platform owner to connect merchants with Billplz easily. With OAuth integration, a merchant can easily integrate with Billplz by Signing In using their Billplz account. That's means; they don't have to manually copy `API Secret Key`, `Collection ID`, and `X Signature Key` to your platform.

<aside class="notice">
  OAuth 2.0 integration is an exclusive feature offered to Billplz's partner. Get in touch with our team for further details. 
</aside>

# Direct Payment Gateway

## Bypass Billplz Bill Page

> Example request:

```shell
# Creates a bill for RM 2.00 with reference_1_label and reference_1
curl https://www.billplz.com/api/v3/bills \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d collection_id="inbmmepb" \
  -d description="Maecenas eu placerat ante." \
  -d email="api@billplz.com" \
  -d name="Michael API V3" \
  -d amount="200" \
  -d reference_1_label="Bank Code" \
  -d reference_1="BP-FKR01" \
  -d callback_url="http://example.com/webhook/"
```

> Response:

```json
{
  "id": "8X0Iyzaw",
  "collection_id": "inbmmepb",
  "paid": false,
  "state": "due",
  "amount": 200 ,
  "paid_amount": 0,
  "due_at": "2015-3-9",
  "email" :"api@billplz.com",
  "mobile": null,
  "name": "MICHAEL API V3",
  "url": "https://www.billplz.com/bills/8X0Iyzaw",
  "reference_1_label": "Bank Code",
  "reference_1": "BP-FKR01",
  "reference_2_label": null,
  "reference_2": null,
  "redirect_url": null,
  "callback_url": "http://example.com/webhook/",
  "description": "Maecenas eu placerat ante.",
  "paid_at": null
}
```

Use this feature if you would like to bypass Billplz bill page, and direct payers straight from bill URL to the selected payment gateway seamlessly.

1. Create bills through Billplz API with a present of `reference_1_label` and `reference_1`.
  - Always set `reference_1_label` as **Bank Code**
  - Always set a bank code to `reference_1`. Please refer to section [API#get-payment-gateways](#get-payment-gateways) for more details on how to get the bank codes.
1. Always append parameter, auto_submit=true to the bill's URL return from [API#create-a-bill](#create-a-bill).
  - Example: [https://www.billplz.com/bills/abcdef]() becomes;
  - [https://www.billplz.com/bills/abcdef?auto_submit=true]()

Direct Payment Gateway feature will fallback to Billplz bill page for invalid `reference_1`, `reference_1_label`, or the selected payment gateway is not enabled or active.

###### STANDARD PAYMENT FLOW

1. Merchant creates bill through API.
1. Merchant redirects payer from merchant's page to bill URL (returned from #1).
1. Payer lands at Billplz page to select payment option.
1. Payer redirected from Billplz to selected payment gateway to pay.
1. Payer redirected from payment gateway to `redirect_url` / receipt page (refer to [API#flow](#api-flow)).

###### DIRECT PAYMENT GATEWAY PAYMENT FLOW

1. Merchant creates bill thru API with a valid bank code
1. Merchant redirects payer from merchant's page to bill URL (returned from #1) with extra parameter, **auto_submit=true**
1. Payer lands at selected payment gateway to pay
1. Payer redirected from payment gateway to `redirect_url` / receipt page (refer to [API#flow](#api-flow))

###### INVALID DIRECT PAYMENT GATEWAY PAYMENT FLOW

1. Merchant creates bill thru API with invalid / inactive bank code
1. Merchant redirects payer from merchant's page to bill URL (returned from #1) with extra parameter, auto_submit=true
1. Payer lands at Billplz page to payment option
1. Payer redirected from Billplz to selected payment gateway to pay
1. Payer redirected from payment gateway to redirect_url / receipt page (refer to [API#flow](#api-flow))

<aside class="warning">
  Bypass bill page using parameter <code>?auto_submit</code> with value <code>fpx</code> or <code>paypal</code> has been removed.
</aside>
<aside class="notice">
  If an inactive or invalid payment gateway for whatever reasons was submitted, the payment process will land at Billplz page for payer to re-choose a payment option again. (refer to <a href="#invalid-direct">INVALID DIRECT PAYMENT GATEWAY PAYMENT FLOW</a>).
  <br><br>
  So, do not worry if your internal Payment Gateways is not always up to date.
</aside>

# V3

This version is not in active development state. No new feature will be introduced in this version.

## Collections

Collections is where all of your Bills will be belongs to. Collections can be useful to categorize your bill payment. As an example, you may use Collection to separate Tuition Fee Collection for September and November collection.

### Create a Collection


# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

###### HTTP Request

`GET http://example.com/api/kittens`

###### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

###### HTTP Request

`GET http://example.com/kittens/<ID>`

###### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

###### HTTP Request

`DELETE http://example.com/kittens/<ID>`

###### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete