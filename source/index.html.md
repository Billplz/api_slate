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
  -d name="Sara" \
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
  "name": "SARA",
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

1. Merchant creates bill through API with a valid bank code
1. Merchant redirects payer from merchant's page to bill URL (returned from #1) with extra parameter, **auto_submit=true**
1. Payer lands at selected payment gateway to pay
1. Payer redirected from payment gateway to `redirect_url` / receipt page (refer to [API#flow](#api-flow))

###### INVALID DIRECT PAYMENT GATEWAY PAYMENT FLOW

1. Merchant creates bill through API with invalid / inactive bank code
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
  So, do not worry if your internal Payment Gateways are not always up to date.
</aside>

# V3

This version is not in active development state. No new feature will be introduced in this version.

## Collections

Collections are where all of your [Bills](#bills) are belongs to. Collections can be useful to categorize your bill payment. As an example, you may use Collection to separate a Tuition Fee Collection for September and November collection.

### Create a Collection

Billplz API now supports the creation of collection with a split rule feature. The response will contain the collection’s ID that is needed in Bill API, split rule info and fields.

<aside class="notice">
  You can choose to upload a logo to display on your bill template. It will use your company logo otherwise.
</aside>

> Example request:

```shell
# Creates a collection
curl https://www.billplz.com/api/v3/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -F title="My First API Collection"
```

> Response: 

```json
{
  "id": "inbmmepb",
  "title": "My First API Collection",
  "logo": 
  {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_payment": 
  {
    "email": null,
    "fixed_cut": null,
    "variable_cut": null,
    "split_header": false
  }
}
```

> Example request with optional arguments:

```shell
curl https://www.billplz.com/api/v3/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -F title="My First API Collection" \
  -F logo=@/Users/Billplz/Documents/uploadPhoto.png \
  -F split_payment[email]="verified@account.com" \
  -F split_payment[fixed_cut]=100 \
  -F split_payment[split_header]=true
```

> Response:

```json
{
  "id": "inbmmepb",
  "title": "My First API Collection",
  "logo":
  {
    "thumb_url": "https://sample.net/assets/uploadPhoto.png",
    "avatar_url": "https://sample.net/assets/uploadPhoto.png"
  },
  "split_payment": 
  {
    "email": "verified@account.com",
    "fixed_cut": 100,
    "variable_cut": null,
    "split_header": true
  }
}
```

###### HTTP Request

`POST https://www.billplz.com/api/v3/collections`

###### REQUIRED ARGUMENTS

| Parameter | Description |
| --- | --- |
| title | The collection title. Will be displayed on bill template. String format.|

###### OPTIONAL ARGUMENTS

| Parameter | Description |
| --- | --- |
| logo | This image will be resized to avatar (40x40) and thumb (180x180) dimensions. Whitelisted formats are `jpg`, `jpeg`, `gif` and `png`. |
| split_payment[email] | The email address of the split rule's recipient. (The account must be a verified account.)|
| split_payment[fixed_cut] | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00) <br>This field is required if `split_payment[variable_cut]` is not present.|
| split_payment[variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present.|
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| id | ID that represents a collection. |
| title | The collection's title in string format. |
| logo[thumb_url] | The thumb dimension's (180x180) URL. |
| logo[avatar_url] | The avatar dimension's (40x40) URL. |
| split_payment[email] | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#get-fpx-banks](#get-fpx-banks). |
| split_payment[fixed_cut] | The 1st recipient's fixed cut in smallest and positive currency unit. |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format. |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true. |

<aside class="notice">
  API <code>V4</code> now supports Split Rule for 2 recipients. Please refer <a href="#v4-create-a-collection">API#v4-create-a-collection</a>.
</aside>

### Get a Collection

Use this API to query your collection record.

> Example request:

```shell
# Get a collection
curl https://www.billplz.com/api/v3/collections/inbmmepb \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "id": "inbmmepb",
  "title": "My First API Collection",
  "logo": 
  {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_payment": 
  {
    "email": null,
    "fixed_cut": null,
    "variable_cut": null,
    "split_header": false
  },
  "status": "active"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | Collection ID returned in Collection object. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| id | ID that represents a collection. |
| title | The collection's title in string format. |
| logo[thumb_url] | The thumb dimension's (180x180) URL. |
| logo[avatar_url] | The avatar dimension's (40x40) URL. |
| split_payment[email] | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#get-fpx-banks](#get-fpx-banks). |
| split_payment[fixed_cut] | The 1st recipient's fixed cut in smallest and positive currency unit. |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format. |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true. |
| status | Collection's status, it is either active and inactive. |

### Get Collection Index

Use this API to retrieve your collections list. To utilise paging, append a page parameter to the URL e.g. ?page=1. If there are 15 records in the response, you will need to check if there is any more data by fetching the next page, e.g. ?page=2 and continuing this process until no more results are returned.

> Example request:

```shell
# Get collection index
curl https://www.billplz.com/api/v3/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "collections":
  [{
    "id": "inbmmepb",
    "title": "My First API Collection",
    "logo": 
    {
      "thumb_url": null,
      "avatar_url": null
    },
    "split_payment": 
    {
      "email": null,
      "fixed_cut": null,
      "variable_cut": null,
      "split_header": false
    },
    "status": "active"
  }],
  "page": 1
}
```

> Example request with optional arguments:

```shell
# Get collection index
curl https://www.billplz.com/api/v3/collections?page=2&status=active \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "collections":
  [{
    "id": "inbmmepb",
    "title": "My First API Collection",
    "logo": 
    {
      "thumb_url": null,
      "avatar_url": null
    },
    "split_payment": 
    {
      "email": null,
      "fixed_cut": null,
      "variable_cut": null,
      "split_header": false
    },
    "status": "active"
  }],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description |
| --- | --- |
| page | Up to 15 collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status | Parameter to filter collection's status, valid value are `active` and `inactive`. |

### Create an Open Collection

Billplz API now supports the creation of open collections (Payment Form) with a split rule feature. The response contains the collection’s attributes, including the payment form URL.

<aside class="notice">
  You can choose to upload a photo to display on your payment form. It uses your company logo otherwise.
</aside>

> Example request:

```shell
# Creates an open collection
curl https://www.billplz.com/api/v3/open_collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d title="My First API Open Collection" \
  -d description="Maecenas eu placerat ante." \
  -d amount=299
```

> Response:

```json
{
  "id": "0pp87t_6",
  "title": "MY FIRST API OPEN COLLECTION",
  "description": "Maecenas eu placerat ante.",
  "reference_1_label": null,
  "reference_2_label": null,
  "email_link": null,
  "amount": 299,
  "fixed_amount": true,
  "tax": null,
  "fixed_quantity": true,
  "payment_button": "pay",
  "photo":
  {
    "retina_url":  null,
    "avatar_url":  null
  },
  "split_payment": 
  {
    "email": null,
    "fixed_cut": null,
    "variable_cut": null,
    "split_header": false
  },
  "url": "https://www.billplz.com/0pp87t_6"
}
```

> Example request with optional arguments:

```shell
# Creates a collection
curl https://www.billplz.com/api/v3/open_collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -F title="My First API Open Collection" \
  -F description="Maecenas eu placerat ante." \
  -F fixed_amount=false \
  -F fixed_quantity=false \
  -F payment_button="buy" \
  -F reference_1_label="ID No" \
  -F reference_2_label="First Name" \
  -F email_link="http://www.test.com" \
  -F tax=1 \
  -F photo=@/Users/Billplz/Documents/uploadPhoto.png \
  -F split_payment[email]="verified@account.com" \
  -F split_payment[variable_cut]=20 \
  -F split_payment[split_header]=true
```

> Response:

```json
{
  "id": "0pp87t_6",
  "title": "MY FIRST API OPEN COLLECTION",
  "description": "Maecenas eu placerat ante.",
  "reference_1_label": "ID No",
  "reference_2_label": "First Name",
  "email_link": "http://www.test.com",
  "amount": null,
  "fixed_amount": false,
  "tax": 1,
  "fixed_quantity": false,
  "payment_button": "buy",
  "photo":
  {
    "retina_url":  "https://sample.net/assets/uploadPhoto.png",
    "avatar_url":  "https://sample.net/assets/uploadPhoto.png"
  },
  "split_payment": 
  {
    "email": "verified@account.com",
    "fixed_cut": null,
    "variable_cut": 20,
    "split_header": true
  },
  "url": "https://www.billplz.com/0pp87t_6"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v3/open_collections`

###### REQUIRED ARGUMENTS

| Parameter | Description |
| --- | --- |
| title | The collection title. It's showing up on the payment form. String format. (Max of 50 characters) |
| description | The collection description. Will be displayed on payment form. String format. (Max of 200 characters) |
|amount | A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00) <br> Required if fixed_amount is true; Ignored if fixed_amount is false|

###### OPTIONAL ARGUMENTS

| Parameter | Description |
| --- | --- |
| fixed_amount | Boolean value. Set this to false for Open Amount. Default value is `true` |
| fixed_quantity | Boolean value. Set this to false for Open Quantity. Default value is `true` |
| payment_button | Payment button's text. Available options are `buy` and `pay`. Default value is `pay` |
| reference_1_label | Label #1 to reconcile payments (Max of 20 characters). <br>Default value is `Reference 1`. |
| reference_2_label | Label #2 to reconcile payments. (Max of 20 characters). <br>Default value is `Reference 2`. |
| email_link | A URL that email to customer after payment is successful. |
| tax | Tax rate in positive integer format. |
| photo | This image will be resized to retina (Yx960) and avatar (180x180) dimensions. Whitelisted formats are jpg, jpeg, gif and png.
| split_payment[email] | The email address of the split rule's recipient. (The account must be a verified account.) |
| split_payment[fixed_cut] | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00). <br> This field is required if `split_payment[variable_cut]` is not present |
| split_payment[variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present|
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| id | The collection ID. |
| title | The collection's title. |
| description | The collection description. |
| reference_1_label | Label #1 to reconcile payments. |
| reference_2_label | Label #2 to reconcile payments. |
| email_link | A URL that email to customer after payment is successful.|
| amount | The collection's fixed amount to create bill in the smallest currency unit (cents). |
| fixed_amount | Boolean value. It returns to `false` if Open Amount.|
| tax | Tax rate in positive integer format. |
| fixed_quantity | Boolean value. It returns `false` if Open Quantity.|
| payment_button | Payment button's text.|
| photo[retina_url] | The retina dimension's (960x960) URL.|
| photo[avatar_url] | The avatar dimension's (180x180) URL.|
| split_payment[email] | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#v4-create-an-open-collection](#v4-create-an-open-collection).|
| split_payment[fixed_cut] | The 1st recipient's fixed cut in smallest and positive currency unit (cents).|
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format.|
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`. |
| url | URL to the collection.|

<aside class="notice">
  API <code>V4</code> now supports Split Rule for 2 recipients. Please refer <a href="#v4-create-an-open-collection">API#v4-create-an-open-collection</a>.
</aside>

### Get an Open Collection

Use this API to query your open collection record.

> Example request:

```shell
# Get an open collection
curl https://www.billplz.com/api/v3/open_collections/0pp87t_6 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "id": "0pp87t_6",
  "title": "MY FIRST API OPEN COLLECTION",
  "description": "Maecenas eu placerat ante.",
  "reference_1_label": null,
  "reference_2_label": null,
  "email_link": null,
  "amount": 299,
  "fixed_amount": true,
  "tax": null,
  "fixed_quantity": true,
  "payment_button": "pay",
  "photo":
  {
    "retina_url":  null,
    "avatar_url":  null
  },
  "split_payment": 
  {
    "email": null,
    "fixed_cut": null,
    "variable_cut": null,
    "split_header": false
  },
  "url": "https://www.billplz.com/0pp87t_6",
  "status": "active"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/open_collections/{COLLECTION_ID}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | Collection ID returned in Collection object. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| id | The collection ID. |
| title | The collection's title. |
| description | The collection description. |
| reference_1_label | Label #1 to reconcile payments. |
| reference_2_label | Label #2 to reconcile payments. |
| email_link | A URL that email to customer after payment is successful.
| amount | The collection's fixed amount to create bill in the smallest currency unit (cents). |
| fixed_amount | Boolean value. It returns to `false` if Open Amount.|
| tax | Tax rate in positive integer format.|
| fixed_quantity | Boolean value. It returns `false` if Open Quantity. |
| payment_button | Payment button's text. |
| photo[retina_url] | The retina dimension's (960x960) URL. |
| photo[avatar_url] | The avatar dimension's (180x180) URL.|
| split_payment[email] | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#v4-create-an-open-collection](). |
| split_payment[fixed_cut] | The 1st recipient's fixed cut in smallest and positive currency unit.|
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format. |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`. |
| url | URL to the collection. |
| status | Collection's status, it is either `active` or `inactive`. |

### Get an Open Collection Index

Use this API to retrieve your open collections list. To utilise paging, append a page parameter to the URL e.g. ?page=1. If there are 15 records in the response you will need to check if there is any more data by fetching the next page e.g. ?page=2 and continuing this process until no more results are returned.

> Example request:

```shell
# Get an open collection index
curl https://www.billplz.com/api/v3/open_collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "open_collections":
  [{
    "id": "0pp87t_6",
    "title": "MY FIRST API OPEN COLLECTION",
    "description": "Maecenas eu placerat ante.",
    "reference_1_label": "ID No",
    "reference_2_label": "First Name",
    "email_link": "http://www.test.com",
    "amount": null,
    "fixed_amount": false,
    "tax": 1,
    "fixed_quantity": false,
    "payment_button": "buy",
    "photo":
    {
      "retina_url":  "https://sample.net/assets/uploadPhoto.png",
      "avatar_url":  "https://sample.net/assets/uploadPhoto.png"
    },
    "split_payment": 
    {
      "email": "verified@account.com",
      "fixed_cut": null,
      "variable_cut": 20,
      "split_header": false
    },
    "url": "https://www.billplz.com/0pp87t_6",
    "status": "active"
  }],
  "page": 1
}
```

> Example request with optional arguments:

```shell
# Get an open collection index
curl https://www.billplz.com/api/v3/open_collections?page=2&status=active \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "open_collections":
  [{
    "id": "0pp87t_6",
    "title": "MY FIRST API OPEN COLLECTION",
    "description": "Maecenas eu placerat ante. Fusce ut neque justo, et aliquet enim. In hac habitasse platea dictumst.",
    "reference_1_label": "ID No",
    "reference_2_label": "First Name",
    "email_link": "http://www.test.com",
    "amount": null,
    "fixed_amount": false,
    "tax": 1,
    "fixed_quantity": false,
    "payment_button": "buy",
    "photo":
    {
      "retina_url":  "https://sample.net/assets/uploadPhoto.png",
      "avatar_url":  "https://sample.net/assets/uploadPhoto.png"
    },
    "split_payment": 
    {
      "email": "verified@account.com",
      "fixed_cut": null,
      "variable_cut": 20,
      "split_header": false
    },
    "url": "https://www.billplz.com/0pp87t_6",
    "status": "active"
  }],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/open_collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description |
| --- | --- |
| page | Up to 15 open collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status | Parameter to filter open collection's status, valid value are `active` and `inactive`. |

### Deactivate a Collection

Both **Collection** and **Open Collection** can be deactivated via this API. The API will respond with error messages if the collection cannot be deactivated.

<aside class="notice">
  It will return <code>message: {COLLECTION_ID} cannot be deactivated.</code> if you trying to deactivate an inactive collection.
</aside>

> Example request:

```shell
# Deactivate a collection or open collection
curl -X POST \
https://www.billplz.com/api/v3/collections/qag4fe_o6/deactivate \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/deactivate`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | Collection ID returned in Collection object. |

### Activate a Collection

Both **Collection** and **Open Collection** can be activated via this API. The API will respond with error messages if the collection cannot be activated.

<aside class="notice">
  It will return <code>message: {COLLECTION_ID} cannot be activated.</code> if you trying to deactivate an active collection.
</aside>

> Example request:

```shell
# Activate a collection or open collection
curl -X POST \
https://www.billplz.com/api/v3/collections/qag4fe_o6/activate \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/activate`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | Collection ID returned in Collection object. |

## Bills

Bills need to be created inside a [collection](#collection). It must be either `Collection` or `Open Collection`. However, only `Collection` can be used to create a bill via API and `Open Collection` cannot be used to create a bill via API. 

### Create a Bill

To create a bill, you would need the collection’s ID. Each bill must be created within a collection. To get your collection ID, visit the collection page on your Billplz account.

The bill's collection will be set to `active` automatically.

<aside class="notice">
  You can choose to redirect the customer to bill’s URL returned to you.
</aside>

> Example request:

```shell
# Creates a bill for RM 2.00
curl https://www.billplz.com/api/v3/bills \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d collection_id=inbmmepb \
  -d description="Maecenas eu placerat ante." \
  -d email=api@billplz.com \
  -d name="Sara" \
  -d amount=200 \
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
  "name": "Sara",
  "url": "https://www.billplz.com/bills/8X0Iyzaw",
  "reference_1_label": "Reference 1",
  "reference_1": null,
  "reference_2_label": "Reference 2",
  "reference_2": null,
  "redirect_url": null,
  "callback_url": "http://example.com/webhook/",
  "description": "Maecenas eu placerat ante."
}
```

> Example request with optional arguments:

```shell
# Creates a bill for RM 2.00
curl https://www.billplz.com/api/v3/bills \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d collection_id=inbmmepb \
  -d email=api@billplz.com \
  -d name="Sara" \
  -d amount=200 \
  -d callback_url="http://example.com/webhook/" \
  -d description="Maecenas eu placerat ante." \
  -d due_at="2020-12-31" \
  --data-urlencode mobile="+60112223333" \
  -d reference_1_label="First Name" \
  -d reference_2_label="Last Name" \
  -d reference_1="Sara" \
  -d reference_2="Dila" \
  -d deliver=false \
  -d redirect_url="http://example.com/redirect/"
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
  "due_at": "2020-12-31",
  "email" :"api@billplz.com",
  "mobile": "+60112223333",
  "name": "Sara",
  "url": "https://www.billplz.com/bills/8X0Iyzaw",
  "reference_1_label": "First Name",
  "reference_1": "Sara",
  "reference_2_label": "Last Name",
  "reference_2": "Dila",
  "redirect_url": "http://example.com/redirect/",
  "callback_url": "http://example.com/webhook/",
  "description": "Maecenas eu placerat ante."
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v3/bills`

###### REQUIRED ARGUMENTS

| Parameter | Description |
| --- | --- |
| collection_id | The collection ID. A string. |
| email | The email address of the bill’s recipient (Email is required if mobile is not present). |
| mobile | Recipient’s mobile number. Be sure that all mobile numbers include country code, area code and number without spaces or dashes. (e.g., `+60122345678` or `60122345678`). Use Google libphonenumber library to help. Mobile is required if email is not present. |
| name | Bill’s recipient name. Useful for identification on recipient part. (Max of 255 characters). |
| amount | A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). |
| callback_url | Web hook URL to be called after payment’s transaction completed. It will POST a Bill object. |
| description | The bill's description. Will be displayed on bill template. String format (Max of 200 characters). |

###### OPTIONAL ARGUMENTS

| Parameter | Description |
| --- | --- |
| due_at | Due date for the bill. The format `YYYY-MM-DD`, default value is today. Year range is **19xx** to **2xxx** |
| redirect_url | URL to redirect the customer after payment completed. It will do a GET to `redirect_url` together with bill’s status and ID.|
| deliver | Boolean value to set email and SMS (if mobile is present) delivery. Default value is `false`. ___SMS is subjected to charges depending on subscribed plan___. |
| reference_1_label | Label #1 to reconcile payments (Max of 20 characters). <br> Default value is `Reference 1`. |
| reference_1 | Value for `reference_1_label` (Max of 120 characters). |
| reference_2_label | Label #2 to reconcile payments (Max of 20 characters). <br> Default value is `Reference 2`. |
| reference_2 | Value for `reference_2_label` (Max of 120 characters). |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| id | Bill ID that represents a bill. |
| url | URL to the bill. |

<aside class="notice">
  SMS will be ignored if your account's credit balance is insufficient.
</aside>

<aside class="success">
  Sending bill via email is free!
</aside>

### Get a Bill

At any given time, you can request a bill to check on the status. It will return the bill object.

<aside class="warning">
  The usage of this API is not recommended as it may subject to our <a href="#rate-limit">Rate Limit</a> policy.
</aside>

<aside class="success">
  Use <a href="#x-signature-redirect">X Signature Redirect</a> and <a href="#x-signature-callback">X Signature Callback</a> to validate the response parameter.
</aside>

> Example request:

```shell
# Get a bill
curl https://www.billplz.com/api/v3/bills/8X0Iyzaw \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: 
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
  "due_at": "2020-12-31",
  "email" :"api@billplz.com",
  "mobile": "+60112223333",
  "name": "SARA",
  "url": "https://www.billplz.com/bills/8X0Iyzaw",
  "reference_1_label": "First Name",
  "reference_1": "Sara",
  "reference_2_label": "Last Name",
  "reference_2": "Dila",
  "redirect_url": "http://example.com/redirect/",
  "callback_url": "http://example.com/webhook/",
  "description": "Maecenas eu placerat ante."
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/bills/{BILL_ID}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| BILL_ID | Bill ID returned in Bill object. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| paid | Boolean value to tell if a bill has paid. It will return `false` for due bills; `true` for paid bills. |
| state | State that representing the bill's status, possible states are `due`, `paid` and `deleted`. |

<aside class="warning">
  ABUSING THIS API WILL RESULT TO YOUR IP ADDRESS WILL BE BLOCKED.
</aside>

### Delete a Bill

Only due bill can be deleted. Paid bill can’t be deleted. Deleting a bill is useful in a scenario where there’s a time limitation to payment.

Example usage would be to show a timer for customer to complete payment within 10 minutes with a grace period of 5 minutes. After 15 minutes of bill creation, get bill status and delete if bill is still due.

> Example Request:

```shell
# Delete a Bill
curl -X DELETE https://www.billplz.com/api/v3/bills/8X0Iyzaw \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`DELETE https://www.billplz.com/api/v3/bills/{BILL_ID}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| BILL_ID | Bill ID returned in Bill object. |

<aside class="notice">
  Deleted bill will always reappeared once the customer make payment. Attempting to delete paid bill will throw 422 http status code.
</aside>

## Registration Check

### By Bank Account Number

At any given time, you can request to check on a registration status by bank account number.

> Example Request:

```shell
curl https://www.billplz.com/api/v3/check/bank_account_number/1234567890 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "name": "verified"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/check/bank_account_number/{BANKcurl https://www.billplz.com/api/v3/check/bank_account_number/1234567890 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:_ACCOUNT_NUMBER}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| BANK_ACCOUNT_NUMBER | Bank account number to check. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| name | State that representing the bank account number's status, possible states are `verified`, `unverified` and `not found`. |

<aside class="notice">
   This registration check is only for checking the status for Company-Registered account. For Bank Direct Verification status, use <a href="#get-a-bank-account">Get a Bank Account</a> API instead.
</aside>

## Transactions

### Get Transaction Index

Use this API to retrieve your bill's transactions. To utilise paging, append a page parameter to the URL e.g. ?page=1. If there are 15 records in the response you will need to check if there is any more data by fetching the next page e.g. ?page=2 and continuing this process until no more results are returned.

> Example Request:

```shell
# Get transaction index
curl https://www.billplz.com/api/v3/bills/inbmmepb/transactions \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "bill_id": "inbmmepb",
  "transactions":
  [{
    "id": "60793D4707CD",
    "status": "completed",
    "completed_at": "2017-02-23T12:49:23.612+08:00",
    "payment_channel": "FPX"
  },{
    "id": "28F3D3194138",
    "status": "failed",
    "completed_at": null,
    "payment_channel": "FPX"
  }],
  "page": 1
}
```

> Example request with optional arguments:

```shell
curl https://www.billplz.com/api/v3/bills/inbmmepb/transactions?page=1&status=completed \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "bill_id": "inbmmepb",
  "transactions":
  [{
    "id": "60793D4707CD",
    "status": "completed",
    "completed_at": "2017-02-23T12:49:23.612+08:00",
    "payment_channel": "FPX"
  }],
  "page": 1
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/bills/{BILL_ID}/transactions`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| page | Up to 15 transactions will be returned in a single API call per specified page. Default to *1* if not present. |
| status | Parameter to filter transaction's status. <br>Valid values are `pending`, `completed` and `failed`. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| bill_id | ID that represent the bill. |
| id | ID that represent the transaction. |
| status | Status that representing the transaction's status, possible statuses are `pending`, `completed` and `failed`. |
| completed_at | Datetime format when the transaction is completed. ISO 8601 format is used. |
| payment_channel | Payment channel that the transaction is made.<br>Possible values are `FPX`, `CIMB`, `TWOCTWOP`, `OCBC`, `BOOST`, `SENANGPAY`, `ISUPAYPAL`, and `PAYPAL`. |

## Payment Methods

### Get Payment Method Index

The get payment methods index API allows to retrieve all available payment methods that you can enable / disable to a collection.

> Example Request:

```shell
# Get payment method index
curl https://www.billplz.com/api/v3/collections/0idsxnh5/payment_methods \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "payment_methods":
  [{
    "code": "paypal",
    "name": "PAYPAL",
    "active": true
  },{
    "code": "fpx",
    "name": "Online Banking",
    "active": false
  }]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | ID that represents the collection where the payment methods belong to. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| code | Unique payment method's specific code, in string value. |
| name | Payment method's general name, in string value. |
| active | The API will return payment method's status for a collection, in boolean value. It returns true if this payment method has enabled for the collection. |

### Update Payment Methods

Pass the payment method's code to the API to update your collection's payment methods.

Invalid payment method code will be ignored.

> Example Request:

```shell
# Update payment methods
curl -X PUT -d payment_methods[][code]='fpx' -d payment_methods[][code]='paypal' https://www.billplz.com/api/v3/collections/0idsxnh5/payment_methods \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "payment_methods":
  [{
    "code": "paypal",
    "name": "PAYPAL",
    "active": true
  },{
    "code": "fpx",
    "name": "Online Banking",
    "active": true
  }]
}
```

###### HTTP REQUEST

`PUT https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| COLLECTION_ID | ID that represents the collection where the payment methods belong to. |
| payment_methods | Array that contains all codes in hash format.  <br>`code` in hash represents the unique payment method's code that you would like to enable. <br>Do not pass the code if you would like to disable a payment method. <br>Ex, `"payment_methods"=>[{"code"=>"fpx"}]` would enable fpx (Online Banking) and disable paypal (PAYPAL). <br>Possible values are `billplz`, `twoctwop`, `fpx`, `boost`, `ocbc`, `senangpay`, and `isupaypal`.|

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| code | Unique payment method's specific code, in string value. |
| name | Payment method's general name, in string value. |
| active | Payment method current's status for the collection, in boolean value. It is true if this payment method has enabled for the collection. |

## Bank Account Direct Verification

### Get Bank Account Index

Query Billplz Bank Account Direct Verification Service by passing list of account numbers arguement. This API will only return latest, matched bank accounts.

> Example Request:

```shell
# Get bank account index
curl 'https://www.billplz.com/api/v3/bank_verification_services?account_numbers\[\]=1234567890&account_numbers\[\]=2234567890&account_numbers\[\]=3234567890' \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response: 

```json
{
  "bank_verification_services":
  [{
    "name": "sara",
    "id_no": "820909101001",
    "acc_no": "1234567890",
    "code": "MBBEMYKL",
    "organization": false,
    "authorization_date": "2015-12-03",
    "status": "pending",
    "processed_at": null,
    "rejected_desc": null
  },{
    "name": "dila",
    "id_no": "820909101002",
    "acc_no": "2234567890",
    "code": "MBBEMYKL",
    "organization": true,
    "authorization_date": "2015-12-03",
    "status": "verified",
    "processed_at": "2015-12-05",
    "rejected_desc": null
  }]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/bank_verification_services?account_numbers=[]`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| account_numbers | Up to 10 bank accounts found will be returned in a single API call. In array format. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| name | Bank account holder's name, in string value. |
| id_no | Bank account's IC Number/SSM Registration Number, in string value. |
| acc_no | Bank account number, in string value. |
| code | Bank Code that represents bank, in string value. |
| organization | Boolean value that tell if the bank account is registered as organization. |
| authorization_date | Request date for authorization, in format `YYYY-MM-DD`. Example, `2012-12-30`. |
| status | Status that representing the authorization status, possible statuses are `pending`, `verified` and `rejected`. |
| processed_at | Date format when the authorization is performed. In format `YYYY-MM-DD`. Example, `2012-12-30`. It returns `null` if authorization has not been made. |
| reject_desc | Reason why the authorization was rejected, in string value. |

### Get a Bank Account

Query Billplz Bank Account Direct Verification Service by passing single account number arguement. This API will only return latest, single matched bank account.

> Example Request:

```shell
# Get a bank account
curl https://www.billplz.com/api/v3/bank_verification_services/1234567890 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response: 

```json
{
  "name": "sara",
  "id_no": "820909101001",
  "acc_no": "1234567890",
  "code": "MBBEMYKL",
  "organization": false,
  "authorization_date": "2015-12-03",
  "status": "pending",
  "processed_at": null,
  "rejected_desc": null
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/bank_verification_services/{BANK_ACCOUNT_NUMBER}`

###### URL PARAMETER

| Parameter | Description |
| --- | --- |
| BANK_ACCOUNT_NUMBER | Bank account number to query. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| name | Bank account holder's name, in string value. |
| id_no | Bank account's IC Number/SSM Registration Number, in string value. |
| acc_no | Bank account number, in string value. |
| code | Bank Code that represents bank, in string value. |
| organization | Boolean value that tell if the bank account is registered as organization. |
| authorization_date | Request date for authorization, in format `YYYY-MM-DD`. Example: `2012-12-30`. |
| status | Status that representing the authorization status, possible statuses are `pending`, `verified` and `rejected`. |
| processed_at | Date format when the authorization is performed. In format `YYYY-MM-DD`. Example, `2012-12-30` It returns `null` if authorization has not been made. |
| reject_desc | Reason why the authorization was rejected, in string value. |

### Create a Bank Account

Request Bank Account Direct Verification Service by creating bank records through this API. You need to wait for 3 working days after this request for the account to be verified.

<aside class="notice">
  Create a Bank account may subject to charges depending on your subscribed plan.
</aside>

> Example Request:

```shell
# Create a bank account
curl https://www.billplz.com/api/v3/bank_verification_services \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d name="Insan Jaya" \
  -d id_no="91234567890" \
  -d acc_no="999988887777" \
  -d code="MBBEMYKL" \
  -d organization=true
```

> Response:

```json
{
  "name": "Insan Jaya",
  "id_no": "91234567890",
  "acc_no": "999988887777",
  "code": "MBBEMYKL",
  "organization": true,
  "authorization_date": "2017-07-03",
  "status": "pending",
  "processed_at": null,
  "rejected_desc": null
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v3/bank_verification_services`

###### REQUIRED ARGUMENTS

| Parameter | Description |
| --- | --- |
| name | Bank account holder's name, in string value. |
| id_no | Bank account's IC Number/SSM Registration Number, in string value. |
| acc_no | Bank account number, in string value. |
| code | Bank Code that represents bank, in string value and case sensitive. <br>Please refer to the Bank Code table below. |
| organization | Boolean value that tell if the bank account is registered as organization. Default to `false` if this parameter is missing. |

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| name | Bank account holder's name, in string value. |
| id_no | Bank account's IC Number/SSM Registration Number, in string value. |
| acc_no | Bank account number, in string value. |
| code | Bank Code that represents bank, in string value. |
| organization | Boolean value that tell if the bank account is registered as organization. |
| authorization_date | Request date for authorization, in format `YYYY-MM-DD`. Example: `2012-12-30` |
| status | Status that representing the authorization status, possible statuses are `pending`, `verified` and `rejected`. |
| processed_at | Date format when the authorization is performed. In format `YYYY-MM-DD`. Example: `2012-12-30`. It returns `null` if authorization has not been made. |
| reject_desc | Reason why the authorization was rejected, in string value. |

#### Bank Code Table

| Bank | Code |
| --- | --- |
| Affin Bank Berhad | PHBMMYKL |
| AGROBANK / BANK PERTANIAN MALAYSIA BERHAD | BPMBMYKL |
| Alliance Bank Malaysia Berhad | MFBBMYKL |
| AL RAJHI BANKING & INVESTMENT CORPORATION (MALAYSIA) BERHAD | RJHIMYKL |
| AmBank (M) Berhad | ARBKMYKL |
| Bank Islam Malaysia Berhad | BIMBMYKL |
| Bank Kerjasama Rakyat Malaysia Berhad | BKRMMYKL |
| Bank Muamalat (Malaysia) Berhad | BMMBMYKL |
| Bank Simpanan Nasional Berhad | BSNAMYK1 |
| CIMB Bank Berhad | CIBBMYKL |
| Citibank Berhad | CITIMYKL |
| Hong Leong Bank Berhad | HLBBMYKL |
| HSBC Bank Malaysia Berhad | HBMBMYKL |
| Kuwait Finance House | KFHOMYKL |
| Maybank / Malayan Banking Berhad | MBBEMYKL |
| OCBC Bank (Malaysia) Berhad | OCBCMYKL |
| Public Bank Berhad | PBBEMYKL |
| RHB Bank Berhad | RHBBMYKL |
| Standard Chartered Bank (Malaysia) Berhad | SCBLMYKX |
| United Overseas Bank (Malaysia) Berhad | UOVBMYKL |

### Get FPX Banks

Use this API to get a list of bank codes that need for setting reference_1 in <a href="#bypass">API#bypass-billplz-bill-page</a>.

This API only return list of bank codes for online banking, if you looking for a complete payment gateways' bank code, please use <a href="#get-payment-gateways">API#get-payment-gateways</a> instead.

<aside class="success">
  Pull the latest bank list on <strong>hourly</strong> basis.
</aside>

> Example Request:

```shell
# Get FPX Banks
curl https://www.billplz.com/api/v3/fpx_banks \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "banks":
  [{
    "name": "MBU0227",
    "active": true
    },{
    "name": "OCBC0229",
    "active": false
    },{
    "name": "MB2U0227",
    "active": true
  }]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/fpx_banks`

###### RESPONSE PARAMETER

| Parameter | Description |
| --- | --- |
| name | This is the bank code that need to set to `reference_1`. Case sensitive. |
| active | `true` or `false` boolean that represents bank's availability. If an inactive bank was set to `reference_1`, the payment process will show Billplz page for payer to choose another bank from the list. |

#### Fpx Bank Abbreviations

| Bank Code | Bank Name |
| --- | --- |
| ABMB0212 | Alliance Bank |
| ABB0233 | Affin Bank |
| AMBB0209 | AmBank |
| BCBB0235 | CIMB Clicks |
| BIMB0340 | Bank Islam |
| BKRM0602 | Bank Rakyat |
| BMMB0341 | Bank Muamalat |
| BSN0601 | BSN |
| CIT0217 | Citibank Berhad |
| HLB0224 | Hong Leong Bank |
| HSBC0223 | HSBC Bank |
| KFH0346 | Kuwait Finance House |
| MB2U0227 | Maybank2u |
| MBB0227 | Maybank2E |
| MBB0228 | Maybank2E |
| OCBC0229 | OCBC Bank |
| PBB0233 | Public Bank |
| RHB0218 | RHB Now |
| SCB0216 | Standard Chartered |
| UOB0226 | UOB Bank |
| TEST0001\* | Test 0001 |
| TEST0002\* | Test 0002 |
| TEST0003\* | Test 0003 |
| TEST0004\* | Test 0004 |
| TEST0021\* | Test 0021 |
| TEST0022\* | Test 0022 |
| TEST0023\* | Test 0023 |

\* Only applicable in staging environment.


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