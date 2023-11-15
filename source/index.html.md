---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.billplz.com'>Billplz</a>
  - <a href='https://www.billplz-sandbox.com'>Billplz Sandbox</a>
  - <a href='https://www.facebook.com/groups/billplzdevjam'>Developer Community</a>
  - <a href='https:///help.billplz.com'>Frequently Asked Questions</a>

includes:
  - errors

search: true
---

# Introduction

> API Endpoint:

```
https://www.billplz.com/api/
```

Welcome to the Billplz API! You can use our API to access Billplz API endpoints to start sending bills and collecting payment.

Our API is organized around REST. JSON will be returned in all responses from the API, including errors. The API will accept both application/x-www-form-urlencoded and application/json.

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
1. Billplz sends a server-side update to your site upon payment failure or success. (Basic Callback URL / X Signature Callback URL depending on your configuration) **_[your backend server should capture the transaction update at this point]_** _refer to [X Signature Callback Url](#payment-completion-x-signature-callback-url)_.
1. Billplz redirects (Payment Completion) the customer back to your site if `redirect_url` is not empty (Basic Redirect URL / X Signature Redirect URL depending on your configuration)
   **_[your server should capture the transaction update at this point and give your user an instant reflection on the page loaded]_** _refer to [X Signature Redirect Url](#payment-completion-x-signature-redirect-url)_
   or,
   The customer will see Billplz receipt if `redirect_url` is not present.

###### COMPLETION FLOW WITHOUT REDIRECT_URL

1. Customer visits your site.
1. Customer chooses to make payment.
1. Your site creates a Bill via API call.
1. Billplz API returns Bill's URL.
1. Your site redirects the customer to Bill's URL.
1. The customer makes payment via payment option of choice.
1. Billplz sends a server-side update to your site upon payment failure or success. (Basic Callback URL / X Signature Callback URL depending on your configuration)
   **_[your backend server should capture the transaction update at this point]_** _refer to [X Signature Callback Url](#payment-completion-x-signature-callback-url)_.
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
<aside class="notice">
  Billplz API only accept <strong>Malaysian Ringgit (MYR)</strong> currency and did not support currency conversion. You need to ensure the value passed is in the smallest currency unit.
</aside>

# Authentication

> To test authentication, use this code:

```shell
# With shell, you can just pass the correct HTTP Basic Auth credential with each request.
curl https://www.billplz.com/api/v4/webhook_rank \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:

# Alternatively, you may use header with base64 encoded API Secret Key
curl https://www.billplz.com/api/v4/webhook_rank \
  -H "Authorization: Basic NzNlYjU3ZjAtN2Q0ZS00MmI5LWE1NDQtYWVhYzZlNGIwZjgxOg=="

```

> If authenticated, you should not see `Unauthorized` response type.

> curl uses the -u flag to pass basic auth credentials (adding a colon after your API key will prevent it from asking you for a password).

You authenticate to the Billplz API by providing your API Secret Keys in the request. You can get your API keys from your account's settings page.

Authentication to the API occurs via HTTP Basic Auth. Provide your API key as the basic auth username. You do not need to provide a password.

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. You must authenticate all requests.

# Integration

> GitHub:

```
https://github.com/billplz
```

> Knowledgebase:

```
https://help.billplz.com/article/53-list-of-system-with-production-ready-integration
```

Billplz has been integrated with many systems available in the market and always working to integrate with more platforms.

For a self-hosted system, you may download a production-ready module to integrate with your e-commerce system. Please refer to our [GitHub](https://github.com/billplz) or [Knowledgebase](https://help.billplz.com/article/53-list-of-system-with-production-ready-integration) for the list of the available module.

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
  "amount": 200,
  "paid_amount": 0,
  "due_at": "2015-3-9",
  "email": "api@billplz.com",
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
- Always set a bank code to `reference_1`. Please refer to section [Get Payment Gateways](#v4-get-payment-gateways) for more details on how to get the bank codes.

1. Always append parameter, auto_submit=true to the bill's URL return from API [Create a Bill](#v3-bills-create-a-bill).

- Example: [https://www.billplz.com/bills/abcdef]() becomes;
- [https://www.billplz.com/bills/abcdef?auto_submit=true]()

Direct Payment Gateway feature will fallback to Billplz bill page for invalid `reference_1`, `reference_1_label`, or the selected payment gateway is not enabled or active.

###### STANDARD PAYMENT FLOW

1. Merchant creates bill through API.
1. Merchant redirects payer from merchant's page to bill URL (returned from #1).
1. Payer lands at Billplz page to select payment option.
1. Payer redirected from Billplz to selected payment gateway to pay.
1. Payer redirected from payment gateway to `redirect_url` / receipt page (refer to [API Flow](#api-flow)).

###### DIRECT PAYMENT GATEWAY PAYMENT FLOW

1. Merchant creates bill through API with a valid bank code
1. Merchant redirects payer from merchant's page to bill URL (returned from #1) with extra parameter, **auto_submit=true**
1. Payer lands at selected payment gateway to pay
1. Payer redirected from payment gateway to `redirect_url` / receipt page (refer to [API Flow](#api-flow))

###### INVALID DIRECT PAYMENT GATEWAY PAYMENT FLOW

1. Merchant creates bill through API with invalid / inactive bank code
1. Merchant redirects payer from merchant's page to bill URL (returned from #1) with extra parameter, auto_submit=true
1. Payer lands at Billplz page to payment option
1. Payer redirected from Billplz to selected payment gateway to pay
1. Payer redirected from payment gateway to redirect_url / receipt page (refer to [API Flow](#api-flow))

<aside class="warning">
  Bypass bill page using parameter <code>?auto_submit</code> with value <code>fpx</code> or <code>paypal</code> has been removed.
</aside>
<aside class="notice">
  If an inactive or invalid payment gateway for whatever reasons was submitted, the payment process will land at Billplz page for payer to re-choose a payment option again. (refer to <a href="#direct-payment-gateway-bypass-billplz-bill-page-3d-secure-update-charge-card-invalid-direct-payment-gateway-payment-flow">INVALID DIRECT PAYMENT GATEWAY PAYMENT FLOW</a>).
  <br><br>
  So, do not worry if your internal Payment Gateways are not always up to date.
</aside>

# V3

This version is not in active development state. No new feature will be introduced in this version.

## Collections

Collections are where all of your [Bills](#v3-bills) are belongs to. Collections can be useful to categorize your bill payment. As an example, you may use Collection to separate a Tuition Fee Collection for September and November collection.

### Create a Collection

Billplz API now supports the creation of collection with a split rule feature. The response will contain the collection's ID that is needed in Bill API, split rule info and fields.

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
  "logo": {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_payment": {
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
  "logo": {
    "thumb_url": "https://sample.net/assets/uploadPhoto.png",
    "avatar_url": "https://sample.net/assets/uploadPhoto.png"
  },
  "split_payment": {
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

| Parameter | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| title     | The collection title. Will be displayed on bill template. String format. |

###### OPTIONAL ARGUMENTS

| Parameter                   | Description                                                                                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| logo                        | This image will be resized to avatar (40x40) and thumb (180x180) dimensions. Whitelisted formats are `jpg`, `jpeg`, `gif` and `png`.                                                         |
| split_payment[email]        | The email address of the split rule's recipient. (The account must be a verified account.)                                                                                                   |
| split_payment[fixed_cut]    | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00) <br>This field is required if `split_payment[variable_cut]` is not present. |
| split_payment[variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present.                                                |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true.                                                                          |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                                                                                                                      |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                          | ID that represents a collection.                                                                                                                                                                                                 |
| title                       | The collection's title in string format.                                                                                                                                                                                         |
| logo[thumb_url]             | The thumb dimension's (180x180) URL.                                                                                                                                                                                             |
| logo[avatar_url]            | The avatar dimension's (40x40) URL.                                                                                                                                                                                              |
| split_payment[email]        | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to V4 [Create a Collection](#v4-collections-create-a-collection). |
| split_payment[fixed_cut]    | The 1st recipient's fixed cut in smallest and positive currency unit.                                                                                                                                                            |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format.                                                                                                                                                                   |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true.                                                                                                              |

<aside class="notice">
  API <code>V4</code> now supports Split Rule for 2 recipients. Please refer <a href="#v4-collections-create-a-collection">API#v4-create-a-collection</a>.
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
  "logo": {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_payment": {
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

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                                                                                                                      |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                          | ID that represents a collection.                                                                                                                                                                                                 |
| title                       | The collection's title in string format.                                                                                                                                                                                         |
| logo[thumb_url]             | The thumb dimension's (180x180) URL.                                                                                                                                                                                             |
| logo[avatar_url]            | The avatar dimension's (40x40) URL.                                                                                                                                                                                              |
| split_payment[email]        | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to V4 [Create a Collection](#v4-collections-create-a-collection). |
| split_payment[fixed_cut]    | The 1st recipient's fixed cut in smallest and positive currency unit.                                                                                                                                                            |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format.                                                                                                                                                                   |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true.                                                                                                              |
| status                      | Collection's status, it is either active and inactive.                                                                                                                                                                           |

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
  "collections": [
    {
      "id": "inbmmepb",
      "title": "My First API Collection",
      "logo": {
        "thumb_url": null,
        "avatar_url": null
      },
      "split_payment": {
        "email": null,
        "fixed_cut": null,
        "variable_cut": null,
        "split_header": false
      },
      "status": "active"
    }
  ],
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
  "collections": [
    {
      "id": "inbmmepb",
      "title": "My First API Collection",
      "logo": {
        "thumb_url": null,
        "avatar_url": null
      },
      "split_payment": {
        "email": null,
        "fixed_cut": null,
        "variable_cut": null,
        "split_header": false
      },
      "status": "active"
    }
  ],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description                                                                                                     |
| --------- | --------------------------------------------------------------------------------------------------------------- |
| page      | Up to 15 collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status    | Parameter to filter collection's status, valid value are `active` and `inactive`.                               |

### Create an Open Collection

Billplz API now supports the creation of open collections (Payment Form) with a split rule feature. The response contains the collection's attributes, including the payment form URL.

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
  "photo": {
    "retina_url": null,
    "avatar_url": null
  },
  "split_payment": {
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
  "photo": {
    "retina_url": "https://sample.net/assets/uploadPhoto.png",
    "avatar_url": "https://sample.net/assets/uploadPhoto.png"
  },
  "split_payment": {
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

| Parameter   | Description                                                                                                                                                |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title       | The collection title. It's showing up on the payment form. String format. (Max of 50 characters)                                                           |
| description | The collection description. Will be displayed on payment form. String format. (Max of 200 characters)                                                      |
| amount      | A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00) <br> Required if fixed_amount is true; Ignored if fixed_amount is false |

###### OPTIONAL ARGUMENTS

| Parameter                   | Description                                                                                                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fixed_amount                | Boolean value. Set this to false for Open Amount. Default value is `true`                                                                                                                     |
| fixed_quantity              | Boolean value. Set this to false for Open Quantity. Default value is `true`                                                                                                                   |
| payment_button              | Payment button's text. Available options are `buy` and `pay`. Default value is `pay`                                                                                                          |
| reference_1_label           | Label #1 to reconcile payments (Max of 20 characters). <br>Default value is `Reference 1`.                                                                                                    |
| reference_2_label           | Label #2 to reconcile payments. (Max of 20 characters). <br>Default value is `Reference 2`.                                                                                                   |
| email_link                  | A URL that email to customer after payment is successful.                                                                                                                                     |
| tax                         | Tax rate in positive integer format.                                                                                                                                                          |
| photo                       | This image will be resized to retina (Yx960) and avatar (180x180) dimensions. Whitelisted formats are jpg, jpeg, gif and png.                                                                 |
| split_payment[email]        | The email address of the split rule's recipient. (The account must be a verified account.)                                                                                                    |
| split_payment[fixed_cut]    | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00). <br> This field is required if `split_payment[variable_cut]` is not present |
| split_payment[variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present                                                  |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to true.                                                                           |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id                          | The collection ID.                                                                                                                                                                                                                               |
| title                       | The collection's title.                                                                                                                                                                                                                          |
| description                 | The collection description.                                                                                                                                                                                                                      |
| reference_1_label           | Label #1 to reconcile payments.                                                                                                                                                                                                                  |
| reference_2_label           | Label #2 to reconcile payments.                                                                                                                                                                                                                  |
| email_link                  | A URL that email to customer after payment is successful.                                                                                                                                                                                        |
| amount                      | The collection's fixed amount to create bill in the smallest currency unit (cents).                                                                                                                                                              |
| fixed_amount                | Boolean value. It returns to `false` if Open Amount.                                                                                                                                                                                             |
| tax                         | Tax rate in positive integer format.                                                                                                                                                                                                             |
| fixed_quantity              | Boolean value. It returns `false` if Open Quantity.                                                                                                                                                                                              |
| payment_button              | Payment button's text.                                                                                                                                                                                                                           |
| photo[retina_url]           | The retina dimension's (960x960) URL.                                                                                                                                                                                                            |
| photo[avatar_url]           | The avatar dimension's (180x180) URL.                                                                                                                                                                                                            |
| split_payment[email]        | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#v4-create-an-open-collection](#v4-collections-create-an-open-collection). |
| split_payment[fixed_cut]    | The 1st recipient's fixed cut in smallest and positive currency unit (cents).                                                                                                                                                                    |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format.                                                                                                                                                                                   |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                            |
| url                         | URL to the collection.                                                                                                                                                                                                                           |

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
  "photo": {
    "retina_url": null,
    "avatar_url": null
  },
  "split_payment": {
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

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id                          | The collection ID.                                                                                                                                                                                                                               |
| title                       | The collection's title.                                                                                                                                                                                                                          |
| description                 | The collection description.                                                                                                                                                                                                                      |
| reference_1_label           | Label #1 to reconcile payments.                                                                                                                                                                                                                  |
| reference_2_label           | Label #2 to reconcile payments.                                                                                                                                                                                                                  |
| email_link                  | A URL that email to customer after payment is successful.                                                                                                                                                                                        |
| amount                      | The collection's fixed amount to create bill in the smallest currency unit (cents).                                                                                                                                                              |
| fixed_amount                | Boolean value. It returns to `false` if Open Amount.                                                                                                                                                                                             |
| tax                         | Tax rate in positive integer format.                                                                                                                                                                                                             |
| fixed_quantity              | Boolean value. It returns `false` if Open Quantity.                                                                                                                                                                                              |
| payment_button              | Payment button's text.                                                                                                                                                                                                                           |
| photo[retina_url]           | The retina dimension's (960x960) URL.                                                                                                                                                                                                            |
| photo[avatar_url]           | The avatar dimension's (180x180) URL.                                                                                                                                                                                                            |
| split_payment[email]        | The 1st recipient's email. It only returns the 1st recipient eventhough there is multiple recipients being set. If you wish to have 2 recipients, please refer to [API#v4-create-an-open-collection](#v4-collections-create-an-open-collection). |
| split_payment[fixed_cut]    | The 1st recipient's fixed cut in smallest and positive currency unit.                                                                                                                                                                            |
| split_payment[variable_cut] | The 1st recipient's percentage cut in positive integer format.                                                                                                                                                                                   |
| split_payment[split_header] | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                            |
| url                         | URL to the collection.                                                                                                                                                                                                                           |
| status                      | Collection's status, it is either `active` or `inactive`.                                                                                                                                                                                        |

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
  "open_collections": [
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
      "photo": {
        "retina_url": "https://sample.net/assets/uploadPhoto.png",
        "avatar_url": "https://sample.net/assets/uploadPhoto.png"
      },
      "split_payment": {
        "email": "verified@account.com",
        "fixed_cut": null,
        "variable_cut": 20,
        "split_header": false
      },
      "url": "https://www.billplz.com/0pp87t_6",
      "status": "active"
    }
  ],
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
  "open_collections": [
    {
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
      "photo": {
        "retina_url": "https://sample.net/assets/uploadPhoto.png",
        "avatar_url": "https://sample.net/assets/uploadPhoto.png"
      },
      "split_payment": {
        "email": "verified@account.com",
        "fixed_cut": null,
        "variable_cut": 20,
        "split_header": false
      },
      "url": "https://www.billplz.com/0pp87t_6",
      "status": "active"
    }
  ],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/open_collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description                                                                                                          |
| --------- | -------------------------------------------------------------------------------------------------------------------- |
| page      | Up to 15 open collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status    | Parameter to filter open collection's status, valid value are `active` and `inactive`.                               |

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

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
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

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

## Bills

Bills need to be created inside a [collection](#v4-collections). It must be either `Collection` or `Open Collection`. However, only `Collection` can be used to create a bill via API and `Open Collection` cannot be used to create a bill via API.

### Create a Bill

To create a bill, you would need the collection's ID. Each bill must be created within a collection. To get your collection ID, visit the collection page on your Billplz account.

The bill's collection will be set to `active` automatically.

<aside class="notice">
  You can choose to redirect the customer to bill's URL returned to you.
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
  "amount": 200,
  "paid_amount": 0,
  "due_at": "2015-3-9",
  "email": "api@billplz.com",
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
  "amount": 200,
  "paid_amount": 0,
  "due_at": "2020-12-31",
  "email": "api@billplz.com",
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

| Parameter     | Description                                                                                                                                                                                                                                                     |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| collection_id | The collection ID. A string.                                                                                                                                                                                                                                    |
| email         | The email address of the bill's recipient (Email is required if mobile is not present).                                                                                                                                                                         |
| mobile        | Recipient's mobile number. Be sure that all mobile numbers include country code, area code and number without spaces or dashes. (e.g., `+60122345678` or `60122345678`). Use Google libphonenumber library to help. Mobile is required if email is not present. |
| name          | Bill's recipient name. Useful for identification on recipient part. (Max of 255 characters).                                                                                                                                                                    |
| amount        | A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00).                                                                                                                                                                             |
| callback_url  | Web hook URL to be called after payment's transaction completed. It will POST a Bill object.                                                                                                                                                                    |
| description   | The bill's description. Will be displayed on bill template. String format (Max of 200 characters).                                                                                                                                                              |

###### OPTIONAL ARGUMENTS

| Parameter         | Description                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| due_at            | Due date for the bill. The format `YYYY-MM-DD`, default value is today. Year range is **19xx** to **2xxx**. Please note that due_at value does not affect the bill's payability and is only for informational reference.                                                  |
| redirect_url      | URL to redirect the customer after payment completed. It will do a GET to `redirect_url` together with bill's status and ID.                                  |
| deliver           | Boolean value to set email and SMS (if mobile is present) delivery. Default value is `false`. **_SMS is subjected to charges depending on subscribed plan_**. |
| reference_1_label | Label #1 to reconcile payments (Max of 20 characters). <br> Default value is `Reference 1`.                                                                   |
| reference_1       | Value for `reference_1_label` (Max of 120 characters).                                                                                                        |
| reference_2_label | Label #2 to reconcile payments (Max of 20 characters). <br> Default value is `Reference 2`.                                                                   |
| reference_2       | Value for `reference_2_label` (Max of 120 characters).                                                                                                        |

###### RESPONSE PARAMETER

| Parameter | Description                     |
| --------- | ------------------------------- |
| id        | Bill ID that represents a bill. |
| url       | URL to the bill.                |

<aside class="notice">
  All bills will default to expiry 30 days after creation, if you would like to expire the bill before the default date please inspect the<a href="#v3-bills-delete-a-bill">bill deactivation api</a>
</aside>

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
  Use <a href="#payment-completion-x-signature-redirect-url">X Signature Redirect</a> and <a href="#payment-completion-x-signature-callback-url">X Signature Callback</a> to validate the response parameter.
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
  "amount": 200,
  "paid_amount": 0,
  "due_at": "2020-12-31",
  "email": "api@billplz.com",
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

| Parameter | Description                      |
| --------- | -------------------------------- |
| BILL_ID   | Bill ID returned in Bill object. |

###### RESPONSE PARAMETER

| Parameter | Description                                                                                            |
| --------- | ------------------------------------------------------------------------------------------------------ |
| paid      | Boolean value to tell if a bill has paid. It will return `false` for due bills; `true` for paid bills. |
| state     | State that representing the bill's status, possible states are `due`, `paid` and `deleted`.            |

<aside class="warning">
  ABUSING THIS API WILL RESULT TO YOUR IP ADDRESS WILL BE BLOCKED.
</aside>

### Delete a Bill

Only due bill can be deleted. Paid bill can't be deleted. Deleting a bill is useful in a scenario where there's a time limitation to payment.

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

| Parameter | Description                      |
| --------- | -------------------------------- |
| BILL_ID   | Bill ID returned in Bill object. |

<aside class="notice">
  Deleted bill will always reappeared once the customer make payment. Attempting to delete paid bill will throw 422 http status code.
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
  "transactions": [
    {
      "id": "60793D4707CD",
      "status": "completed",
      "completed_at": "2017-02-23T12:49:23.612+08:00",
      "payment_channel": "FPX"
    },
    {
      "id": "28F3D3194138",
      "status": "failed",
      "completed_at": null,
      "payment_channel": "FPX"
    }
  ],
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
  "transactions": [
    {
      "id": "60793D4707CD",
      "status": "completed",
      "completed_at": "2017-02-23T12:49:23.612+08:00",
      "payment_channel": "FPX"
    }
  ],
  "page": 1
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/bills/{BILL_ID}/transactions`

###### URL PARAMETER

| Parameter | Description                                                                                                    |
| --------- | -------------------------------------------------------------------------------------------------------------- |
| page      | Up to 15 transactions will be returned in a single API call per specified page. Default to _1_ if not present. |
| status    | Parameter to filter transaction's status. <br>Valid values are `pending`, `completed` and `failed`.            |

###### RESPONSE PARAMETER

| Parameter       | Description                                                                                                                                                                                                                                                                                          |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bill_id         | ID that represent the bill.                                                                                                                                                                                                                                                                          |
| id              | ID that represent the transaction.                                                                                                                                                                                                                                                                   |
| status          | Status that representing the transaction's status, possible statuses are `pending`, `completed` and `failed`.                                                                                                                                                                                        |
| completed_at    | Datetime format when the transaction is completed. ISO 8601 format is used.                                                                                                                                                                                                                          |
| payment_channel | Payment channel that the transaction is made.<br>Possible values are `AMEXMBB`, `BANKISLAM`, `BILLPLZ`, `BOOST`, `TOUCHNGO`, `EBPGMBB`, `FPX`, `FPXB2B1`, `ISUPAYPAL`, `MPGS`, `OCBC`, `PAYDEE`, `RAZERPAYWALLET`, `SECUREACCEPTANCE`, `SENANGPAY`, `TWOCTWOP`, `TWOCTWOPIPP`, and `TWOCTWOPWALLET`. |

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
  "payment_methods": [
    {
      "code": "paypal",
      "name": "PAYPAL",
      "active": true
    },
    {
      "code": "fpx",
      "name": "Online Banking",
      "active": false
    }
  ]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods`

###### URL PARAMETER

| Parameter     | Description                                                            |
| ------------- | ---------------------------------------------------------------------- |
| COLLECTION_ID | ID that represents the collection where the payment methods belong to. |

###### RESPONSE PARAMETER

| Parameter | Description                                                                                                                                            |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| code      | Unique payment method's specific code, in string value.                                                                                                |
| name      | Payment method's general name, in string value.                                                                                                        |
| active    | The API will return payment method's status for a collection, in boolean value. It returns true if this payment method has enabled for the collection. |

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
  "payment_methods": [
    {
      "code": "paypal",
      "name": "PAYPAL",
      "active": true
    },
    {
      "code": "fpx",
      "name": "Online Banking",
      "active": true
    }
  ]
}
```

###### HTTP REQUEST

`PUT https://www.billplz.com/api/v3/collections/{COLLECTION_ID}/payment_methods`

###### URL PARAMETER

| Parameter       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| COLLECTION_ID   | ID that represents the collection where the payment methods belong to.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| payment_methods | Array that contains all codes in hash format. <br>`code` in hash represents the unique payment method's code that you would like to enable. <br>Do not pass the code if you would like to disable a payment method. <br>Ex, `"payment_methods"=>[{"code"=>"fpx"}]` would enable fpx (Online Banking) and disable paypal (PAYPAL). <br>Possible values are `amexmbb`, `bankislam`, `billplz`, `boost`, `touchngo`, `ebpgmbb`, `fpx`, `fpxb2b1`, `isupaypal`, `mpgs`, `ocbc`, `paydee`, `razerpaywallet`, `secureacceptance`, `senangpay`, `twoctwop`, `twoctwopipp`, and `twoctwopwallet`. |

###### RESPONSE PARAMETER

| Parameter | Description                                                                                                                             |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| code      | Unique payment method's specific code, in string value.                                                                                 |
| name      | Payment method's general name, in string value.                                                                                         |
| active    | Payment method current's status for the collection, in boolean value. It is true if this payment method has enabled for the collection. |

## Get FPX Banks

Use this API to get a list of bank codes that need for setting reference_1 in [API#bypass-billplz-bill-page](#direct-payment-gateway-bypass-billplz-bill-page)</a>.

This API only return list of bank codes for online banking, if you looking for a complete payment gateways' bank code, please use <a href="#v4-get-payment-gateways">API#get-payment-gateways</a> instead.

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
  "banks": [
    {
      "name": "MBU0227",
      "active": true
    },
    {
      "name": "OCBC0229",
      "active": false
    },
    {
      "name": "MB2U0227",
      "active": true
    }
  ]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v3/fpx_banks`

###### RESPONSE PARAMETER

| Parameter | Description                                                                                                                                                                                             |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name      | This is the bank code that need to set to `reference_1`. Case sensitive.                                                                                                                                |
| active    | `true` or `false` boolean that represents bank's availability. If an inactive bank was set to `reference_1`, the payment process will show Billplz page for payer to choose another bank from the list. |

#### Fpx Bank Abbreviations

| Bank Code  | Bank Name                   |
| ---------- | --------------------------- |
| ABMB0212   | allianceonline              |
| ABB0233    | affinOnline                 |
| ABB0234\*  | Affin Bank                  |
| AMBB0209   | AmOnline                    |
| BCBB0235   | CIMB Clicks                 |
| BIMB0340   | Bank Islam Internet Banking |
| BKRM0602   | i-Rakyat                    |
| BMMB0341   | i-Muamalat                  |
| BOCM01\*   | Bank of China               |
| BSN0601    | myBSN                       |
| CIT0219    | Citibank Online             |
| HLB0224    | HLB Connect                 |
| HSBC0223   | HSBC Online Banking         |
| KFH0346    | KFH Online                  |
| MB2U0227   | Maybank2u                   |
| MBB0227    | Maybank2E                   |
| MBB0228    | Maybank2E                   |
| OCBC0229   | OCBC Online Banking         |
| PBB0233    | PBe                         |
| RHB0218    | RHB Now                     |
| SCB0216    | SC Online Banking           |
| UOB0226    | UOB Internet Banking        |
| UOB0229\*  | UOB Bank                    |
| TEST0001\* | Test 0001                   |
| TEST0002\* | Test 0002                   |
| TEST0003\* | Test 0003                   |
| TEST0004\* | Test 0004                   |
| TEST0021\* | Test 0021                   |
| TEST0022\* | Test 0022                   |
| TEST0023\* | Test 0023                   |

\* Only applicable in staging environment.

# V4

This version is in active development state. New feature will be introduced in this version.

## Collections

Collections are where all of your [Bills](#v3-bills) are belongs to. Collections can be useful to categorize your bill payment. As an example, you may use Collection to separate a Tuition Fee Collection for September and November collection.

### Create a Collection

Billplz API now support creation of collection with 2 split rules feature, the response will contain the collection's ID that is needed in Bill API, split rules info and fields

<aside class="warning">
  Collection with custom logo support are removed starting in <code>V4</code>
</aside>

> Example Request:

```shell
# Creates a collection
curl https://www.billplz.com/api/v4/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -F title="My First V4 API Collection"
```

> Response:

```json
{
  "id": "inbmmepb",
  "title": "My First V4 API Collection",
  "logo": {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_header": false,
  "split_payments": []
}
```

> Example request with optional arguments:

```shell
curl https://www.billplz.com/api/v4/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -F title="My First V4 API Collection"\
  -F split_payments[][email]="verified@account.com" \
  -F split_payments[][fixed_cut]=100 \
  -F split_payments[][variable_cut]=2\
  -F split_payments[][stack_order]=0\
  -F split_payments[][email]="verified2@account.com" \
  -F split_payments[][fixed_cut]=200 \
  -F split_payments[][variable_cut]=3\
  -F split_payments[][stack_order]=1
```

> Response:

```json
{
  "id": "inbmmepb",
  "title": "My First V4 API Collection",
  "logo": {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_header": false,
  "split_payments": [
    {
      "email": "verified@account.com",
      "fixed_cut": 100,
      "variable_cut": 2,
      "stack_order": 0
    },
    {
      "email": "verified2@account.com",
      "fixed_cut": 200,
      "variable_cut": 3,
      "stack_order": 1
    }
  ]
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/collections`

###### REQUIRED ARGUMENTS

| Parameter | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| title     | The collection title. Will be displayed on bill template. String format. |

###### OPTIONAL ARGUMENTS

| Parameter                      | Description                                                                                                                                                                                                                                                                       |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| split_payments[][email]        | The email address of the split rule's recipient (The account must be a verified account).                                                                                                                                                                                         |
| split_payments[][fixed_cut]    | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00). <br>This field is required if `split_payment[variable_cut]` is not present.                                                                                     |
| split_payments[][variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present.                                                                                                                                     |
| split_payments[][stack_order]  | Integer format that defines the sequence of the split rule recipients. <br>This field is required and must be in correct order starts from 0 and increment by 1 subsequently if you want to set a split rule. <br>This input is crucial to determine a precise recipient's order. |
| split_header                   | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                                                             |

###### RESPONSE PARAMETER

| Parameter        | Description                                                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| id               | ID that represents a collection.                                                                                      |
| title            | The collection's title in string format.                                                                              |
| logo[thumb_url]  | The thumb dimension's (180x180) URL.                                                                                  |
| logo[avatar_url] | The avatar dimension's (40x40) URL.                                                                                   |
| split_header     | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`. |
| split_payments   | Array that contains all split rule recipients in hash format.                                                         |
| email            | in hash represents the recipient's email.                                                                             |
| fixed_cut        | in hash represents the recipient's fixed cut in smallest and positive currency unit.                                  |
| variable_cut     | The recipient's percentage cut in positive integer format.                                                            |
| stack_order      | The order of the recipient defined in split rules.                                                                    |

### Get a Collection

Use this API to query your collection record.

> Example request:

```shell
# Get a collection
curl https://www.billplz.com/api/v4/collections/inbmmepb \
-u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "id": "inbmmepb",
  "title": "My First API Collection",
  "logo": {
    "thumb_url": null,
    "avatar_url": null
  },
  "split_header": false,
  "split_payments": [
    {
      "email": "verified@account.com",
      "fixed_cut": 100,
      "variable_cut": 2,
      "stack_order": 0
    },
    {
      "email": "verified2@account.com",
      "fixed_cut": 200,
      "variable_cut": 3,
      "stack_order": 1
    }
  ],
  "status": "active"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/collections/{COLLECTION_ID}`

###### RESPONSE PARAMETER

| Parameter        | Description                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id               | ID that represents a collection.                                                                                                                                                                                                                                                                                                                                                 |
| title            | The collection's title in string format.                                                                                                                                                                                                                                                                                                                                         |
| logo[thumb_url]  | The thumb dimension's (180x180) URL.                                                                                                                                                                                                                                                                                                                                             |
| logo[avatar_url] | The avatar dimension's (40x40) URL.                                                                                                                                                                                                                                                                                                                                              |
| split_header     | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                                                                                                                                                            |
| split_payments   | Array that contains all split rule recipients in hash format. <br>`email` in hash represents the recipient's email. <br>`fixed_cut` in hash represents the recipient's fixed cut in smallest and positive currency unit. <br>`variable_cut` is the recipient's percentage cut in positive integer format.<br>`stack_order` is the order of the recipient defined in split rules. |
| status           | Collection's status, it is either `active` and `inactive`.                                                                                                                                                                                                                                                                                                                       |

<aside class="notice">
   It will return <code>404</code> status code, and a message of <code>RecordNotFound</code> if the collection you trying to query is not exists.
</aside>

### Get Collection Index

Use this API to retrieve your collections list. To utilise paging, append a page parameter to the URL e.g. ?page=1. If there are 15 records in the response you will need to check if there is any more data by fetching the next page e.g. ?page=2 and continuing this process until no more results are returned.

> Example request:

```shell
# Get collection index
curl https://www.billplz.com/api/v4/collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "collections": [
    {
      "id": "inbmmepb",
      "title": "My First API Collection",
      "logo": {
        "thumb_url": null,
        "avatar_url": null
      },
      "split_header": false,
      "split_payments": [],
      "status": "active"
    }
  ],
  "page": 1
}
```

> Example request with optional arguments:

```shell
# Get collection index
curl https://www.billplz.com/api/v4/collections?page=2&status=active \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "collections": [
    {
      "id": "inbmmepb",
      "title": "My First API Collection",
      "logo": {
        "thumb_url": null,
        "avatar_url": null
      },
      "split_header": false,
      "split_payments": [],
      "status": "active"
    }
  ],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description                                                                                                     |
| --------- | --------------------------------------------------------------------------------------------------------------- |
| page      | Up to 15 collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status    | Parameter to filter collection's status, valid value are `active` and `inactive`.                               |

### Create an Open Collection

Billplz API now support creation of open collections (Payment Form) with 2 split rule recipients feature, the response will contain the collection's attributes, including the payment form URL.

> Example request:

```shell
# Creates an open collection
curl https://www.billplz.com/api/v4/open_collections \
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
  "photo": {
    "retina_url": null,
    "avatar_url": null
  },
  "split_header": false,
  "split_payments": [],
  "url": "https://www.billplz.com/0pp87t_6",
  "redirect_uri": null
}
```

> Example request with optional arguments:

```shell
# Creates a collection
curl https://www.billplz.com/api/v4/open_collections \
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
  -F split_header=true \
  -F split_payments[][email]="verified@account.com" \
  -F split_payments[][variable_cut]=20 \
  -F split_payments[][stack_order]=0 \
  -F redirect_uri="http://www.test.com"
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
  "photo": {
    "retina_url": "https://sample.net/assets/uploadPhoto.png",
    "avatar_url": "https://sample.net/assets/uploadPhoto.png"
  },
  "split_header": true,
  "split_payments": [
    {
      "email": "verified@account.com",
      "fixed_cut": null,
      "variable_cut": 20,
      "stack_order": 0
    }
  ],
  "url": "https://www.billplz.com/0pp87t_6",
  "redirect_uri": "http://www.test.com"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/open_collections`

###### REQUIRED ARGUMENTS

| Parameter   | Description                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title       | The collection title. Will be displayed on payment form. String format (Max of 50 characters).                                                                      |
| description | The collection description. Will be displayed on payment form. String format (Max of 200 characters).                                                               |
| amount      | A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). <br>Required if `fixed_amount` is `true`; Ignored if `fixed_amount` is `false`. |

###### OPTIONAL ARGUMENTS

| Parameter                      | Description                                                                                                                                                                                                                                                                       |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| fixed_amount                   | Boolean value. Set `fixed_amount` to `false` for Open Amount. Default value is `true`.                                                                                                                                                                                            |
| fixed_quantity                 | Boolean value. Set `fixed_quantity` to `false` for Open Quantity. Default value is `true`.                                                                                                                                                                                        |
| payment_button                 | Payment button's text. Available options are `buy` and `pay`. Default value is `pay`.                                                                                                                                                                                             |
| reference_1_label              | Label #1 to reconcile payments (Max of 20 characters). <br>Default value is `Reference 1`.                                                                                                                                                                                        |
| reference_2_label              | Label #2 to reconcile payments (Max of 20 characters). <br>Default value is `Reference 2`.                                                                                                                                                                                        |
| email_link                     | A URL that email to customer after payment is successful.                                                                                                                                                                                                                         |
| tax                            | Tax rate in positive integer format.                                                                                                                                                                                                                                              |
| photo                          | This image will be resized to retina (960x960) and avatar (180x180) dimensions. Whitelisted formats are `jpg`, `jpeg`, `gif` and `png`.                                                                                                                                           |
| split_payments[][email]        | The email address of the split rule's recipient (The account must be a verified account).                                                                                                                                                                                         |
| split_payments[][fixed_cut]    | A positive integer in the smallest currency unit that is going in your account (e.g 100 cents to charge RM 1.00) <br>This field is required if `split_payment[variable_cut]` is not present.                                                                                      |
| split_payments[][variable_cut] | Percentage in positive integer format that is going in your account. <br>This field is required if `split_payment[fixed_cut]` is not present.                                                                                                                                     |
| split_payments[][stack_order]  | Integer format that defines the sequence of the split rule recipients. <br>This field is required and must be in correct order starts from 0 and increment by 1 subsequently if you want to set a split rule. <br>This input is crucial to determine a precise recipient's order. |
| split_header                   | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                                                             |
| redirect_uri                   | URL to redirect the customer after payment <i>(completed or failed)</i>. Billplz will do a GET to redirect_uri, with bill's `ID` appended to the URL <i>(additional `paid`, `paid_at` and `x_signature` if <a href="#x-signature">x_signature</a> is enabled)</i>.                |

###### RESPONSE PARAMETER

| Parameter         | Description                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                | The collection ID.                                                                                                                                                                                                                                                                                                                                                                |
| title             | The collection's title.                                                                                                                                                                                                                                                                                                                                                           |
| description       | The collection description.                                                                                                                                                                                                                                                                                                                                                       |
| reference_1_label | Label #1 to reconcile payments.                                                                                                                                                                                                                                                                                                                                                   |
| reference_2_label | Label #2 to reconcile payments.                                                                                                                                                                                                                                                                                                                                                   |
| email_link        | A URL that email to customer after payment is successful.                                                                                                                                                                                                                                                                                                                         |
| amount            | The collection's fixed amount to create bill in the smallest currency unit.                                                                                                                                                                                                                                                                                                       |
| fixed_amount      | Boolean value. It returns to `false` if Open Amount.                                                                                                                                                                                                                                                                                                                              |
| tax               | Tax rate in positive integer format.                                                                                                                                                                                                                                                                                                                                              |
| fixed_quantity    | Boolean value. It returns `false` if Open Quantity.                                                                                                                                                                                                                                                                                                                               |
| payment_button    | Payment button's text.                                                                                                                                                                                                                                                                                                                                                            |
| photo[retina_url] | The retina dimension's (960x960) URL.                                                                                                                                                                                                                                                                                                                                             |
| photo[avatar_url] | The avatar dimension's (180x180) URL.                                                                                                                                                                                                                                                                                                                                             |
| split_header      | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                                                                                                                                                             |
| split_payments    | Array that contains all split rule recipients in hash format. <br>`email` in hash represents the recipient's email. <br>`fixed_cut` in hash represents the recipient's fixed cut in smallest and positive currency unit. <br>`variable_cut` is the recipient's percentage cut in positive integer format. <br>`stack_order` is the order of the recipient defined in split rules. |
| url               | URL to the collection.                                                                                                                                                                                                                                                                                                                                                            |
| redirect_uri      | URL to redirect the customer after payment <i>(completed or failed)</i>. Billplz will do a GET to redirect_uri, with bill's `ID` appended to the URL <i>(additional `paid`, `paid_at` and `x_signature` if <a href="#x-signature">x_signature</a> is enabled)</i>.                                                                                                                |

### Get an Open Collection

Use this API to query your open collection record.

> Example request:

```shell
# Get an open collection
curl https://www.billplz.com/api/v4/open_collections/0pp87t_6 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "id": "0pp87t_6",
  "title": "MY FIRST API OPEN COLLECTION",
  "description": "Maecenas eu placerat ante. Fusce ut neque justo, et aliquet enim. In hac habitasse platea dictumst.",
  "reference_1_label": null,
  "reference_2_label": null,
  "email_link": null,
  "amount": 299,
  "fixed_amount": true,
  "tax": null,
  "fixed_quantity": true,
  "payment_button": "pay",
  "photo": {
    "retina_url": null,
    "avatar_url": null
  },
  "split_header": false,
  "split_payments": [],
  "url": "https://www.billplz.com/0pp87t_6",
  "status": "active",
  "redirect_uri": null
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/open_collections/{COLLECTION_ID}`

###### RESPONSE PARAMETER

| Parameter         | Description                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                | The collection ID.                                                                                                                                                                                                                                                                                                                                                                |
| title             | The collection's title.                                                                                                                                                                                                                                                                                                                                                           |
| description       | The collection description.                                                                                                                                                                                                                                                                                                                                                       |
| reference_1_label | Label #1 to reconcile payments.                                                                                                                                                                                                                                                                                                                                                   |
| reference_2_label | Label #2 to reconcile payments.                                                                                                                                                                                                                                                                                                                                                   |
| email_link        | A URL that email to customer after payment is successful.                                                                                                                                                                                                                                                                                                                         |
| amount            | The collection's fixed amount to create bill in the smallest currency unit.                                                                                                                                                                                                                                                                                                       |
| fixed_amount      | Boolean value. It returns to `false` if Open Amount.                                                                                                                                                                                                                                                                                                                              |
| tax               | Tax rate in positive integer format.                                                                                                                                                                                                                                                                                                                                              |
| fixed_quantity    | Boolean value. It returns `false` if Open Quantity.                                                                                                                                                                                                                                                                                                                               |
| payment_button    | Payment button's text.                                                                                                                                                                                                                                                                                                                                                            |
| photo[retina_url] | The retina dimension's (960x960) URL.                                                                                                                                                                                                                                                                                                                                             |
| photo[avatar_url] | The avatar dimension's (180x180) URL.                                                                                                                                                                                                                                                                                                                                             |
| split_header      | Boolean value. All bill and receipt templates will show split rule recipient's infographic if this was set to `true`.                                                                                                                                                                                                                                                             |
| split_payments    | Array that contains all split rule recipients in hash format. <br>`email` in hash represents the recipient's email. <br>`fixed_cut` in hash represents the recipient's fixed cut in smallest and positive currency unit. <br>`variable_cut` is the recipient's percentage cut in positive integer format. <br>`stack_order` is the order of the recipient defined in split rules. |
| url               | URL to the collection.                                                                                                                                                                                                                                                                                                                                                            |
| status            | Collection's status, it is either `active` and `inactive`.                                                                                                                                                                                                                                                                                                                        |
| redirect_uri      | URL to redirect the customer after payment <i>(completed or failed)</i>. Billplz will do a GET to redirect_uri, with bill's `ID` appended to the URL <i>(additional `paid`, `paid_at` and `x_signature` if <a href="#x-signature">x_signature</a> is enabled)</i>.                                                                                                                |

<aside class="notice">
  It will return 404 status code, and a message of RecordNotFound if the open collection you trying to query is not exists.
</aside>

### Get Open Collection Index

Use this API to retrieve your open collections list. To utilise paging, append a page parameter to the URL e.g. ?page=1. If there are 15 records in the response you will need to check if there is any more data by fetching the next page e.g. ?page=2 and continuing this process until no more results are returned.

> Example request:

```shell
# Get open collection index
curl https://www.billplz.com/api/v4/open_collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "open_collections": [
    {
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
      "photo": {
        "retina_url": "https://sample.net/assets/uploadPhoto.png",
        "avatar_url": "https://sample.net/assets/uploadPhoto.png"
      },
      "split_header": false,
      "split_payments": [
        {
          "email": "verified@account.com",
          "fixed_cut": null,
          "variable_cut": 20,
          "stack_order": 0
        }
      ],
      "url": "https://www.billplz.com/0pp87t_6",
      "status": "active",
      "redirect_uri": null
    }
  ],
  "page": 1
}
```

> Example request with optional arguments:

```shell
# Get collection index
curl https://www.billplz.com/api/v4/open_collections?page=2&status=active \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "open_collections": [
    {
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
      "photo": {
        "retina_url": "https://sample.net/assets/uploadPhoto.png",
        "avatar_url": "https://sample.net/assets/uploadPhoto.png"
      },
      "split_header": false,
      "split_payments": [
        {
          "email": "verified@account.com",
          "fixed_cut": null,
          "variable_cut": 20,
          "stack_order": 0
        }
      ],
      "url": "https://www.billplz.com/0pp87t_6",
      "status": "active",
      "redirect_uri": null
    }
  ],
  "page": 2
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/open_collections`

###### OPTIONAL ARGUMENTS

| Parameter | Description                                                                                                          |
| --------- | -------------------------------------------------------------------------------------------------------------------- |
| page      | Up to 15 open collections will be returned in a single API call per specified page. Default to **1** if not present. |
| status    | Parameter to filter open collection's status, valid value are `active` and `inactive`.                               |

### Customer Receipt Delivery

By default, all collections follow the Global Customer Receipt Notification configuration in your Account settings.

You can override the Global configuration on individual collections by calling [activate](#v4-collections-customer-receipt-delivery-activate) and [deactivate](#v4-collections-customer-receipt-delivery-deactivate).

#### Activate

If you wish to ignore the Global configuration and always send a receipt email to your customers, you may activate it per individual collection by calling to this API.

> Example request:

```shell
# Activate a collection's customer receipt delivery
curl -X POST \
https://www.billplz.com/api/v4/collections/qag4fe_o6/customer_receipt_delivery/activate \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/collections/{COLLECTION_ID}/customer_receipt_delivery/activate`

###### URL PARAMETER

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

#### Deactivate

If you wish to ignore the Global configuration and always not to send a receipt email to your customers, you may deactivate it per individual collection by calling to this API.

> Example request:

```shell
# Deactivate a collection's customer receipt delivery
curl -X POST \
https://www.billplz.com/api/v4/collections/qag4fe_o6/customer_receipt_delivery/deactivate \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/collections/{COLLECTION_ID}/customer_receipt_delivery/deactivate`

###### URL PARAMETER

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

#### Set Global

Use this API to set your individual collections to follow Global Customer Receipt Notification configuration if you want your collection to send the email base on Global configuration. By default, all collections are following Global configuration.

> Example request:

```shell
# Set a collection's customer receipt delivery to Global
curl -X POST \
https://www.billplz.com/api/v4/collections/qag4fe_o6/customer_receipt_delivery/global \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/collections/{COLLECTION_ID}/customer_receipt_delivery/global`

###### URL PARAMETER

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

#### Get Status

Use this API to get a Collection's Customer Receipt Notification status.

> Example request:

```shell
# Get a collection's customer receipt delivery
curl https://www.billplz.com/api/v4/collections/qag4fe_o6/customer_receipt_delivery \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "id": "qag4fe_o6",
  "customer_receipt_delivery": "global"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/collections/{COLLECTION_ID}/customer_receipt_delivery`

###### URL PARAMETER

| Parameter     | Description                                  |
| ------------- | -------------------------------------------- |
| COLLECTION_ID | Collection ID returned in Collection object. |

###### RESPONSE PARAMETER

| Parameter                 | Description                                                                                       |
| ------------------------- | ------------------------------------------------------------------------------------------------- |
| id                        | ID that represents a collection.                                                                  |
| customer_receipt_delivery | Collection's Customer Receipt Notification status, it is either `active`, `inactive` or `global`. |

## Webhook Rank

Webhook Rank has been introduced to ensure callback is running at it's best. The higher the ranking, the higher priority for the callback to be executed. Use this API to query your current Account Ranking. `0.0` indicate highest ranking (default) and `10.0` lowest ranking.

<aside class="notice">
  The ranking will be reset to default daily at 17:00.
</aside>

> Example request:

```shell
# Webhook Rank
curl https://www.billplz.com/api/v4/webhook_rank \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "rank": 0.0
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/webhook_rank`

###### RESPONSE PARAMETER

| Parameter | Description                 |
| --------- | --------------------------- |
| rank      | Ranking Number (0.0 - 10.0) |

## Get Payment Gateways

Use this API to get a complete list of supported payment gateways' bank code that need for setting `reference_1` in [API#bypass-billplz-bill-page](#direct-payment-gateway-bypass-billplz-bill-page).

This API returns not only online banking, but also all other payment gateways' bank code that are supported in [API#bypass-billplz-bill-page](#direct-payment-gateway-bypass-billplz-bill-page).

<aside class="success">
  Pull payment gateway list on <strong>hourly</strong> basis.
</aside>

> Example request:

```shell
curl https://www.billplz.com/api/v4/payment_gateways \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Response:

```json
{
  "payment_gateways": [
    {
      "code": "MBU0227",
      "active": true,
      "category": "fpx"
    },
    {
      "code": "OCBC0229",
      "active": false,
      "category": "fpx"
    },
    {
      "code": "BP-FKR01",
      "active": true,
      "category": "billplz"
    },
    {
      "code": "BP-PPL01",
      "active": true,
      "category": "paypal"
    },
    {
      "code": "BP-2C2P1",
      "active": false,
      "category": "2c2p"
    },
    {
      "code": "BP-OCBC1",
      "active": true,
      "category": "ocbc"
    }
  ]
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v4/payment_gateways`

###### RESPONSE PARAMETER

| Parameter | Description                                                                                                                                                                                                                                       |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| code      | This is the bank code that need to set to `reference_1`. Case sensitive.                                                                                                                                                                          |
| active    | `true` or `false` boolean that represents payment gateway's availability. If an inactive / invalid payment gateway was set to `reference_1`, the payment process will show Billplz page for payer to choose another payment option from the list. |
| category  | Category this payment gateway belongs to.                                                                                                                                                                                                         |

#### Payment Gateway Abbreviations

| Code            | Name                        |
| --------------- | --------------------------- |
| ABMB0212        | allianceonline              |
| ABB0233         | affinOnline                 |
| ABB0234\*       | Affin Bank                  |
| AMBB0209        | AmOnline                    |
| AGRO01          | AGRONet                     |
| BCBB0235        | CIMB Clicks                 |
| BIMB0340        | Bank Islam Internet Banking |
| BKRM0602        | i-Rakyat                    |
| BMMB0341        | i-Muamalat                  |
| BOCM01\*        | Bank of China               |
| BSN0601         | myBSN                       |
| CIT0219         | Citibank Online             |
| HLB0224         | HLB Connect                 |
| HSBC0223        | HSBC Online Banking         |
| KFH0346         | KFH Online                  |
| MB2U0227        | Maybank2u                   |
| MBB0227         | Maybank2E                   |
| MBB0228         | Maybank2E                   |
| OCBC0229        | OCBC Online Banking         |
| PBB0233         | PBe                         |
| RHB0218         | RHB Now                     |
| SCB0216         | SC Online Banking           |
| UOB0226         | UOB Internet Banking        |
| UOB0229\*       | UOB Bank                    |
| TEST0001\*      | Test 0001                   |
| TEST0002\*      | Test 0002                   |
| TEST0003\*      | Test 0003                   |
| TEST0004\*      | Test 0004                   |
| TEST0021\*      | Test 0021                   |
| TEST0022\*      | Test 0022                   |
| TEST0023\*      | Test 0023                   |
| BP-FKR01\*      | Billplz Simulator           |
| BP-PPL01        | PayPal                      |
| BP-OCBC1        | Visa / Mastercard           |
| BP-2C2P1        | e-pay                       |
| BP-2C2PC        | Visa / Mastercard           |
| BP-2C2PU        | UnionPay                    |
| BP-2C2PGRB      | Grab                        |
| BP-2C2PGRBPL    | GrabPayLater                |
| BP-2C2PATOME    | Atome                       |
| BP-2C2PBST      | Boost                       |
| BP-2C2PTNG      | TnG                         |
| BP-2C2PSHPE     | Shopee Pay                  |
| BP-2C2PSHPQR    | Shopee Pay QR               |
| BP-2C2PIPP      | IPP                         |
| BP-BST01        | Boost                       |
| BP-TNG01        | TouchNGo E-Wallet           |
| BP-SGP01        | Senangpay                   |
| BP-BILM1        | Visa / Mastercard           |
| BP-RZRGRB       | Grab                        |
| BP-RZRBST       | Boost                       |
| BP-RZRTNG       | TnG                         |
| BP-RZRPAY       | RazerPay                    |
| BP-RZRMB2QR     | Maybank QR                  |
| BP-RZRWCTP      | WeChat Pay                  |
| BP-RZRSHPE      | Shopee Pay                  |
| BP-MPGS1        | MPGS                        |
| BP-CYBS1        | Secure Acceptance           |
| BP-EBPG1        | Visa / Mastercard           |
| BP-EBPG2        | AMEX                        |
| BP-PAYDE        | Paydee                      |
| BP-MGATE1       | Visa / Mastercard / AMEX    |
| B2B1-ABB0235    | AFFINMAX                    |
| B2B1-ABMB0213   | Alliance BizSmart           |
| B2B1-AGRO02     | AGRONetBIZ                  |
| B2B1-AMBB0208   | AmAccess Biz                |
| B2B1-BCBB0235   | BizChannel@CIMB             |
| B2B1-BIMB0340   | Bank Islam eBanker          |
| B2B1-BKRM0602   | i-bizRAKYAT                 |
| B2B1-BMMB0342   | iBiz Muamalat               |
| B2B1-BNP003     | BNP Paribas                 |
| B2B1-CIT0218    | CitiDirect BE               |
| B2B1-DBB0199    | Deutsche Bank Autobahn      |
| B2B1-HLB0224    | HLB ConnectFirst            |
| B2B1-HSBC0223   | HSBCnet                     |
| B2B1-KFH0346    | KFH Online                  |
| B2B1-MBB0228    | Maybank2E                   |
| B2B1-OCBC0229   | Velocity@ocbc               |
| B2B1-PBB0233    | PBe                         |
| B2B1-PBB0234    | PB enterprise               |
| B2B1-RHB0218    | RHB Reflex                  |
| B2B1-SCB0215    | SC Straight2Bank            |
| B2B1-TEST0021\* | SBI Bank A                  |
| B2B1-TEST0022\* | SBI Bank B                  |
| B2B1-TEST0023\* | SBI Bank C                  |
| B2B1-UOB0228    | UOB BIBPlus                 |

\* Only applicable in staging environment.

## Tokenization

This feature allows you to exchange for a card's tokenization from our provider's PCI DSS certified vault, and use the token to charge your customer later.

###### Providers

| Provider  | Type | Eligibility              |
| --------- | ---- | ------------------------ |
| Senangpay | 3DS  | Any paid membership plan |

### Senangpay

This feature enables you to tokenize 3DS Visa / Mastercard cards to be charged later, which will be stored in Senangpay's PCI DSS certified servers.

###### Ways to use Senangpay's tokenization:

1. [Tokenize customer's card](#v4-tokenization-senangpay-create-card), so they only need to enter their card details once.
1. Using [token](#v4-tokenization-senangpay-create-card) obtained, pay a Billplz bill by [charging customers's card](#v4-tokenization-senangpay-charge-card), anytime, any amount.
1. Using [token](#v4-tokenization-senangpay-create-card) obtained, [pre authorize a bill payment](#v4-tokenization-senangpay-pre-auth), and [capture](#v4-tokenization-senangpay-pre-auth-capture) later in the future.

<aside class="notice">
  This feature won't be enabled by default, and only applicable to paid plan members. Email <a href="mailto:team@billplz.com?subject=Senangpay_Tokenization">team@billplz.com</a> for assistance.
</aside>

#### Create Card

Use this API to create a card token for 3DS Visa / Mastercard cards. Remember to store the response, as no card details nor tokens will be stored in Billplz's servers.

###### Flow

1. Create card & token using this [Create Card](#v4-tokenization-senangpay-create-card) API.
1. Merchant redirects card holder to Senangpay's 3DS authentication page.
1. Card holder inputs credit/debit card details and submits the form.
1. Upon 3DS verifcation success, Billplz will send a POST request to merchant's callback_url containing masked card details together with CARD_ID and TOKEN.
1. Merchant compares the checksum sent, and store card details if they match.

To charge a card with the token generated, refer to [Charge Card](#v4-tokenization-senangpay-charge-card) API.

> Example request:

```shell
# Creates a card token
curl https://www.billplz.com/api/v4/cards \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d name="Michael" \
  -d email="api@billplz.com" \
  -d phone="60122345678" \
  -d callback_url="https://example.com/callback_url"
```

> Response:

```json
{
  "id": "8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6",
  "card_number": null,
  "provider": null,
  "token": null,
  "status": "pending",
  "authentication_redirect_url": "https://senangpay.my/some_link",
  "callback_url": "https://example.com/callback_url"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/cards`

###### REQUIRED ARGUMENTS

| Parameter    | Description                                                                        |
| ------------ | ---------------------------------------------------------------------------------- |
| name         | Name on the card / name of the card owner.                                         |
| email        | Email of the card owner.                                                           |
| phone        | Contact number of card owner.                                                      |
| callback_url | Web hook URL to be called after card token is created. It will POST a Card object. |

<aside class="notice">
  This tokenization function can only tokenize Visa / Mastercard cards. You won't be able to tokenize Amex cards.
</aside>

#### Delete Card

Use this API to delete a card.

> Example request:

```shell
# Delete an active card token
curl -X DELETE https://www.billplz.com/api/v4/cards/8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d token="77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740"
```

> Response:

```json
{
  "id": "8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6",
  "card_number": "1118",
  "provider": "mastercard",
  "token": "77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740",
  "status": "deleted"
}
```

###### HTTP REQUEST

`DELETE https://www.billplz.com/api/v4/cards/{CARD_ID}`

###### REQUIRED ARGUMENTS

| Parameter | Description   |
| --------- | ------------- |
| token     | Card's token. |

#### 3D Secure Update

Billplz will send a POST request to `callback_url` provided within an hour, regardless the card holder has completed the 3DSecure verification or not. This callback_url will also serve as `redirect_url` on the client's side. You are required to redirect every time the 3D secure update is being given regardless of callback or redirect.

<aside class="notice">
  Acceptable HTTP response status code is within the range of <code>200</code> - <code>308</code>. Responding with other than acceptable range will be considered as failure.
</aside>

> Example request to callback_url:

```shell
# POST request sent by Billplz to your callback_url
curl https://www.example.com/callback \
  -d id="a35296ad-b50c-4179-8024-036da00c1aee" \
  -d card_number="1118" \
  -d provider="mastercard" \
  -d token="77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740" \
  -d status="active" \
  -d checksum="ef5e54a22af0925cba88fab467119742e90262e3646eea1dee3949938daf3a38"
```

###### HTTP REQUEST

`POST {CALLBACK_URL}`

###### POST PARAMETER

| Parameter   | Description                                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------------------------- |
| id          | ID that represents card.                                                                                    |
| card_number | Last 4 digits of card's number.                                                                             |
| token       | Card's token.                                                                                               |
| status      | Status that represents the card's status, possible values are `pending`, `active`, `failed`, and `deleted`. |
| checksum    | Digital signature computed with posted data and shared XSignature Key.                                      |

###### CHECKSUM

Checksum is a digital signature computed with posted data and shared XSignature Key, similar to the X Signature received on Payment Completion.

For security purposes, checksum is inserted into this POST request so that merchants can verify this request comes from Billplz. To learn more on how to calculate this checksum, [click here](#x-signature).

<aside class="success">
  Check the checksum value for better security.
</aside>

#### Charge Card

Use this API to make bill payment by charging a Visa / Mastercard card with [token generated](#v4-tokenization-senangpay-create-card).

###### REQUIREMENTS

1. Collection without split recipients (split payment).
1. Bill. Email and Mobile number are required during bill creation.
1. [Card Token & ID](#v4-tokenization-senangpay-create-card).

<aside class="notice">
  This function is only applicable for Visa / Mastercard cards. You won't be able to charge Amex cards.
</aside>

> Example request:

```shell
# Make bill payment with token
curl https://www.billplz.com/api/v4/bills/awyzmy0m/charge \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d token="77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740" \
  -d card_id="8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6"
```

> Response:

```json
{
  "amount": 10000,
  "status": "success",
  "reference_id": "15681981586116610",
  "hash_value": "1b66606732d846192b0b6aa4b754b3c8addd59072fce4bdd066b5d631c31d5e8",
  "message": "Payment was successful"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/bills/{BILL_ID}/charge`

###### REQUIRED ARGUMENTS

| Parameter | Description                |
| --------- | -------------------------- |
| card_id   | ID that represents a card. |
| token     | Card token.                |

#### Pre-Auth

Use this API to pre-authorize a transaction with [token generated](#v4-tokenization-senangpay-create-card), and capture later.
Amount will not be charged to the card, but the amount will be blocked until the transaction is captured or released.

###### REQUIREMENTS

1. Collection without split recipients (split payment).
1. [Card Token & ID](#v4-tokenization-senangpay-create-card).

<aside class="notice">
  This function is only applicable for Visa / Mastercard cards. You won't be able to charge Amex cards.
</aside>

> Example request:

```shell
# Pre authorize bill payment with token
curl https://www.billplz.com/api/v4/bills/awyzmy0m/preauth \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d token="77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740" \
  -d card_id="8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6"
```

> Response:

```json
{
  "amount": 10000,
  "preauth_status": "success",
  "preauth_id": "732d8sdk778912e81003946192b0b6aa4b754b3c8addd5",
  "hash_value": "1b66606732d846192b0b6aa4b754b3c8addd59072fce4bdd066b5d631c31d5e8",
  "message": "Preauth was successful"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth`

###### REQUIRED ARGUMENTS

| Parameter | Description                |
| --------- | -------------------------- |
| card_id   | ID that represents a card. |
| token     | Card token.                |

#### Pre-Auth Capture

Use this API to capture a pre-authorize transaction with [token generated](#v4-tokenization-senangpay-create-card). Amount will be charged to the credit card right away.

###### REQUIREMENTS

1. Collection without split recipients (split payment).
1. [Card Token & ID](#v4-tokenization-senangpay-create-card).
1. [Preauth ID](#v4-tokenization-senangpay-preauth).

<aside class="notice">
  This function is only applicable for Visa / Mastercard cards. You won't be able to charge Amex cards.
</aside>

> Example request:

```shell
# Capture a pre-authed transaction
curl https://www.billplz.com/api/v4/bills/awyzmy0m/preauth_capture \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d token="77d62ad5a3ae56aafc8e3529b89d0268afa205303f6017afbd9826afb8394740" \
  -d card_id="8727fc3a-c04c-4c2b-9b67-947b5cfc2fb6" \
  -d preauth_id="732d8sdk778912e81003946192b0b6aa4b754b3c8addd5"
```

> Response:

```json
{
  "amount": 10000,
  "status": "success",
  "reference_id": "15681981586116610",
  "preauth_id": "732d8sdk778912e81003946192b0b6aa4b754b3c8addd5",
  "hash_value": "1b66606732d846192b0b6aa4b754b3c8addd59072fce4bdd066b5d631c31d5e8",
  "message": "Payment was successful"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v4/bills/{BILL_ID}/preauth_capture`

###### REQUIRED ARGUMENTS

| Parameter  | Description                   |
| ---------- | ----------------------------- |
| card_id    | ID that represents a card.    |
| token      | Card token.                   |
| preauth_id | ID for pre-authed transaction |

# V5

_This version is in active development state. New Feature will be introduced in this version._

V5 API introduces new security measures. Every request made in V5 endpoints must include **_epoch_** and **_checksum_** values in addition to each endpoint's required arguments.

- **Epoch** param must be in UNIX epoch time format
- **Checksum** calculation is specific to each endpoint, please refer to the **CHECKSUM ARGUMENTS** of each endpoint for more information on this.
- Checksum signature must be calculated using **HMAC_SHA512** together with your account **XSignature key**
- Please refer to the [guide](#v5-checksum) on how to generate a v5 checksum signature

<aside class="warning">
  The formation of checksum signature in API V5 is different from the way Billplz's XSignatureVerification is formatted.
</aside>

<aside class="notice">
  Each endpoint will indicate the necessary values required to format the string for checksum generation
</aside>

<aside class="notice">
  Optional values required to format the string for checksum generation are denoted with an asterisk (*)
</aside>

## Payment Order Flow

Payment Order allows you to make payment to any account bank registered in Malaysia. Since Bank doesn't provide way to programatically make payment to bank account, you can achieve that by using our Payment Order.

Payment Order implements the same collection concept as a bill. You will have collection that consists of multiple payment orders.

Before proceeding further, you need to ensure that you have enough payment order limit to perform payment order. To increase payment order limit, navigate to payment order tab and you will notice the payment order limit at the top.

![Payment Order Limit Interface Screenshot.](payoutlimit.png)

To start using the API, you would have to create a Payment Order Collection. Then the payment order will kicks in as per below:

1. Get bank information from the recipient.
2. Execute [Create a Payment Order](#v5-payment-order-create-a-payment-order) API.
3. The payment will be settled to the receipient once the processing completed.

## Payment Order Collections

### Create a Payment Order Collection

Payment Order Collection used to group all your Payment Orders to make payment order transfers.

> Example request:

```shell
# Creates a Payment Order collection
curl https://www.billplz.com/api/v5/payment_order_collections \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d title="My First API Payment Order Collection"  \
  -d epoch=1668147670 \
  -d callback_url="https://example.com/payment_order_collection_callback" \
  -d checksum="7ea7b8aed3093b058bfee1d67387f0d6f3be1d776ae3ebd82462237ccf2ffc1ae8b9bf38ad43a81bb2b15f817970afdf656dc622258230b08aa26c7418274041"
```

> Response:

```json
{
  "id": "8f4e331f-ac71-435e-a870-72fe520b4563",
  "title": "My First API Payment Order Collection",
  "payment_orders_count": "0",
  "paid_amount": "0",
  "status": "active",
  "callback_url": "https://example.com/payment_order_collection_callback"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v5/payment_order_collections`

###### REQUIRED ARGUMENTS

| Parameter | Description                                                                                               |
| --------- | --------------------------------------------------------------------------------------------------------- |
| title     | The collection title. Will be displayed on bill template. String format.                                  |
| epoch     | The current time in UNIX epoch time format.                                                               |
| checksum  | Required values for [checksum signature](#v5-checksum) in this order: **[ title, callback_url*, epoch ]** |

###### OPTIONAL ARGUMENTS

| Parameter    | Description                                                                                                                                                             |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback_url | Web hook URL to be called after payment order's transaction completed. It will POST a Payment Order object. [Find out more](#payment-completion-payment-order-callback) |

###### RESPONSE PARAMETER

| Parameter            | Description                                                                                                                                                             |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                   | ID that represents a collection.                                                                                                                                        |
| title                | The collection's title in string format.                                                                                                                                |
| payment_orders_count | The number of payment order belongs to this collection.                                                                                                                 |
| paid_amount          | Total paid amount for payment orders in this collection. <br>A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00).                        |
| status               | Collection's status, it is either `active` and `inactive`.                                                                                                              |
| callback_url         | Web hook URL to be called after payment order's transaction completed. It will POST a Payment Order object. [Find out more](#payment-completion-payment-order-callback) |

### Get a Payment Order Collection

Use this API to query your Payment Order Collection record.

> Example request:

```shell
# Get a Payment Order collection
curl -G https://www.billplz.com/api/v5/payment_order_collections/8f4e331f-ac71-435e-a870-72fe520b4563 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d epoch=1668147747 \
  -d checksum="2bc6333ac9539e8fc5e520432373991403155f80315c7f3c601bd6a5535ee9ae70b0d8c29823ccf3be73a251caced9f5a165d6c0e9cadd451931ec3c1ede006d"
```

> Response:

```json
{
  "id": "8f4e331f-ac71-435e-a870-72fe520b4563",
  "title": "My First API Payment Order Collection",
  "payment_orders_count": "0",
  "paid_amount": "0",
  "status": "active",
  "callback_url": "https://example.com/payment_order_collection_callback"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v5/payment_order_collections/{payment_order_collection_id}`

###### REQUIRED ARGUMENTS

| Parameter                   | Description                                                                                                      |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| payment_order_collection_id | The Payment Order Collection ID. A string.                                                                       |
| epoch                       | The current time in UNIX epoch time format.                                                                      |
| checksum                    | Required values for [checksum signature](#v5-checksum) in this order: **[ payment_order_collection_id, epoch ]** |

## Payment Order

### Create a Payment Order

To make a payment transfer to another bank account, simply create a Payment Order.
To create a Payment Order, you would need the Payment Order collection's ID. Each Payment Order must be created within a Payment Order Collection.

<aside class=info>
 Depending on your plan, a fee might be charged from your credits when you successfully created a Payment Order request
</aside>

<aside class="warning">
  It returns status code of 422 with message 'You do not have enough payments' if you are trying to make a payment with total that are exceeding your <strong>Payment Order Limit</strong>.
</aside>

> Example request:

```shell
# Creates a Payment Order for RM 20.00
curl https://www.billplz.com/api/v5/payment_orders \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d payment_order_collection_id="8f4e331f-ac71-435e-a870-72fe520b4563" \
  -d bank_code="MBBEMYKL" \
  -d bank_account_number="543478924652" \
  -d name="Michael Yap" \
  -d description="Maecenas eu placerat ante." \
  -d total=2000
  -d epoch=1668148460
  -d checksum="1516c31cd226119732b66dd423a5ae8d9741594953dfb4bd510efc7349cd7579dc885014bbb67baf8398150cdba5ac42c35196ef1a1acea437386711e159ce62"
```

> Response:

```json
{
  "id": "ddfb5a5d-7b09-4f8c-9258-866f7ef4fa7c",
  "payment_order_collection_id": "8f4e331f-ac71-435e-a870-72fe520b4563",
  "bank_code": "MBBEMYKL",
  "bank_account_number": "543478924652",
  "name": "Michael Yap",
  "description": "Maecenas eu placerat ante.",
  "email": "hello@billplz.com",
  "status": "enquiring",
  "notification": false,
  "recipient_notification": true,
  "total": "2000",
  "reference_id": null,
  "display_name": "Michael Yap"
}
```

> Example request with optional arguments:

```shell
# Creates a Payment Order for RM 20.00
curl https://www.billplz.com/api/v5/payment_orders \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d payment_order_collection_id="8f4e331f-ac71-435e-a870-72fe520b4563" \
  -d bank_code="MBBEMYKL" \
  -d bank_account_number="543478924652" \
  -d name="Michael Yap" \
  -d description="Maecenas eu placerat ante." \
  -d total=2000 \
  -d email="recipient@email.com" \
  -d notification=true \
  -d recipient_notification=false \
  -d reference_id="paymentOrder1" \
  -d epoch=1668148796
  -d checksum="fce47038896eb5c70fd33a8faf91529023cf67fa48afcc31adb20263e45a37b5cd0211fd95377a1053b9bbf70de0245c9117576998264eebb932f8244c878eed"
```

> Response:

```json
{
  "id": "cc92738f-dfda-4969-91dc-22a44afc7e26",
  "payment_order_collection_id": "8f4e331f-ac71-435e-a870-72fe520b4563",
  "bank_code": "MBBEMYKL",
  "bank_account_number": "543478924652",
  "name": "Michael Yap",
  "description": "Maecenas eu placerat ante.",
  "email": "recipient@email.com",
  "status": "enquiring",
  "notification": true,
  "recipient_notification": false,
  "total": "2000",
  "reference_id": "paymentOrder1",
  "display_name": "Michael Yap"
}
```

###### HTTP REQUEST

`POST https://www.billplz.com/api/v5/payment_orders`

###### REQUIRED ARGUMENTS

| Parameter                   | Description                                                                                                                                                                                                                                                                                                                                    |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| payment_order_collection_id | The Payment Order Collection ID. A string.                                                                                                                                                                                                                                                                                                     |
| bank_code                   | [SWIFT Bank Code](#v5-payment-order-swift-bank-code-table) that represents bank, in string value. Case sensitive.                                                                                                                                                                                                                              |
| bank_account_number         | Bank account number, in string value.                                                                                                                                                                                                                                                                                                          |
| name                        | Payment Order API's recipient name. Useful for identification on recipient part.                                                                                                                                                                                                                                                               |
| description                 | The Payment Order API's description. Will be displayed on bill template. String format (Max of 200 characters).                                                                                                                                                                                                                                |
| total                       | Total amount you would like to transfer to the recipient. <br>A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00).                                                                                                                                                                                              |
| epoch                       | The current time in UNIX epoch time format.                                                                                                                                                                                                                                                                                                    |
| checksum                    | Required values for [checksum signature](#v5-checksum) in this order: **[ payment_order_collection_id, bank_account_number, total, epoch ]**                                                                                                                                                                                                   |

###### OPTIONAL ARGUMENTS

| Parameter              | Description                                                                                                                                                                                                                                                                                                                        |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email                  | The email address of recipient (it default to sender's email if not present). <br>A receipt will be sent to this email once the Payment Order has been processed.                                                                                                                                                                  |
| notification           | Boolean value. As a sender, you can opt-in for email notification by setting this to `true`. Sender will receive email once a Payment Order has been completed. Default value is `false`.                                                                                                                                          |
| recipient_notification | Boolean value. If this is set to `true`, recipient of the Payment Order will receive email notification once the Payment Order has been completed. Default value is `true`. Set to false if you do not like the recipient to receive any email notifications.                                                                      |
| reference_id           | Payment Order's unique reference ID scoped by [Payment Order Collection](#v5-payment-order-collections). This helps maintain data integrity by ensuring that no two rows of data in a payment order collection have identical reference_id value. Useful to prevent unintentional payment order creation. (Max of 255 characters). |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                 |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| id                          | ID that represents a Payment Order.                                                                                         |
| payment_order_collection_id | The Payment order collection's title in string format.                                                                      |
| bank_code                   | SWIFT Bank Code that represents bank, in string value. Case sensitive.                                                      |
| bank_account_number         | Bank account number, in string value.                                                                                       |
| name                        | Payment Order's recipient name.                                                                                             |
| description                 | The Payment Order's description.                                                                                            |
| email                       | The email address of recipient (it default to sender's email if not present).                                               |
| status                      | Payment Order status. It is either `processing` or `enquiring` or `executing` or `reviewing` or `completed` or `refunded`.  |
| notification                | Boolean value. Sender will receive email notification if this is `true`.                                                    |
| recipient_notification      | Boolean value. Recipient will receive email notification if this is `true`.                                                 |
| total                       | Total amount transfer to the recipient. A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). |
| reference_id                | Payment Order's reference ID. Useful for identification on recipient part.                                                  |

### Get a Payment Order

Use this API to query your Payment Order record.

> Example request:

```shell
# Get a Payment Order
curl -G https://www.billplz.com/api/v5/payment_orders/cc92738f-dfda-4969-91dc-22a44afc7e26 \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d epoch=1668149595 \
  -d checksum="92987e17459c6e488b83c02dea1693615011fee049d88a3eb9745538f191e323ac4f067571aa8abc335075470b06693994443b52b78be22fbd12b44cb699b265"
```

> Response:

```json
{
  "id": "cc92738f-dfda-4969-91dc-22a44afc7e26",
  "payment_order_collection_id": "8f4e331f-ac71-435e-a870-72fe520b4563",
  "bank_code": "MBBEMYKL",
  "bank_account_number": "543478924652",
  "name": "Michael Yap",
  "description": "Maecenas eu placerat ante.",
  "email": "recipient@email.com",
  "status": "enquiring",
  "notification": true,
  "recipient_notification": false,
  "total": "2000",
  "reference_id": "paymentOrder1",
  "display_name": "Michael Yap"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v5/payment_orders/{payment_order_id}`

###### REQUIRED ARGUMENTS

| Parameter        | Description                                                                                           |
| ---------------- | ----------------------------------------------------------------------------------------------------- |
| payment_order_id | The Payment Order ID. A string.                                                                       |
| epoch            | The current time in UNIX epoch time format.                                                           |
| checksum         | Required values for [checksum signature](#v5-checksum) in this order: **[ payment_order_id, epoch ]** |

###### RESPONSE PARAMETER

| Parameter                   | Description                                                                                                                 |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| id                          | ID that represents a Payment Order.                                                                                         |
| payment_order_collection_id | The Payment order collection's title in string format.                                                                      |
| bank_code                   | SWIFT Bank Code that represents bank, in string value. Case sensitive.                                                      |
| bank_account_number         | Bank account number, in string value.                                                                                       |
| name                        | Payment Order's recipient name.                                                                                             |
| description                 | The Payment Order's description.                                                                                            |
| email                       | The email address of recipient (it default to sender's email if not present).                                               |
| status                      | Payment Order status. It is either `processing` or `enquiring` or `executing` or `reviewing` or `completed` or `refunded`.  |
| notification                | Boolean value. Sender will receive email notification if this is `true`.                                                    |
| recipient_notification      | Boolean value. Recipient will receive email notification if this is `true`.                                                 |
| total                       | Total amount transfer to the recipient. A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). |
| reference_id                | Payment Order API's reference ID. Useful for identification on recipient part.                                              |

<aside class="warning">
  The usage of this API is not recommended as it is subject to our <a href="#rate-limit">Rate Limit</a> policy.
</aside>

<aside class="info">
  If your payment order collection was configured with a callback_url, billplz will POST a callback to the designated url with the result of your payment order. Find out more about <a href="#payment-completion-payment-order-callback">Payment Order Callbacks</a>
</aside>

### SWIFT Bank Code Table

| Bank                                                        | Code     |
| ----------------------------------------------------------- | -------- |
| Affin Bank Berhad                                           | PHBMMYKL |
| AGROBANK / BANK PERTANIAN MALAYSIA BERHAD                   | AGOBMYKL |
| Alliance Bank Malaysia Berhad                               | MFBBMYKL |
| AL RAJHI BANKING & INVESTMENT CORPORATION (MALAYSIA) BERHAD | RJHIMYKL |
| AmBank (M) Berhad                                           | ARBKMYKL |
| Bank Islam Malaysia Berhad                                  | BIMBMYKL |
| Bank Kerjasama Rakyat Malaysia Berhad                       | BKRMMYKL |
| Bank Muamalat (Malaysia) Berhad                             | BMMBMYKL |
| Bank Simpanan Nasional Berhad                               | BSNAMYK1 |
| CIMB Bank Berhad                                            | CIBBMYKL |
| Citibank Berhad                                             | CITIMYKL |
| Hong Leong Bank Berhad                                      | HLBBMYKL |
| HSBC Bank Malaysia Berhad                                   | HBMBMYKL |
| Kuwait Finance House                                        | KFHOMYKL |
| Maybank / Malayan Banking Berhad                            | MBBEMYKL |
| OCBC Bank (Malaysia) Berhad                                 | OCBCMYKL |
| Public Bank Berhad                                          | PBBEMYKL |
| RHB Bank Berhad                                             | RHBBMYKL |
| Standard Chartered Bank (Malaysia) Berhad                   | SCBLMYKX |
| United Overseas Bank (Malaysia) Berhad                      | UOVBMYKL |

## Payment Order Limit

### Get a Payment Order Limit

Use this API to query your Payment Order Limit record.

> Example request:

```shell
# Get a Payment Order Limit
curl -G https://www.billplz.com/api/v5/payment_order_limit \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81: \
  -d epoch=1685591208 \
  -d checksum="e18c50ca130db623d350123ed9cc0c83120361d1045737eb172396b3b41b0141c24c26de6ca41b66dfa476c2c5299a31df21c1fdbf6e0b585ea6e7a975fbd555"
```

> Response:

```json
{
  "total": "9600"
}
```

###### HTTP REQUEST

`GET https://www.billplz.com/api/v5/payment_order_limit`

###### REQUIRED ARGUMENTS

| Parameter | Description                                                                         |
| --------- | ----------------------------------------------------------------------------------- |
| epoch     | The current time in UNIX epoch time format.                                         |
| checksum  | Required values for [checksum signature](#v5-checksum) in this order: **[ epoch ]** |

###### RESPONSE PARAMETER

| Parameter | Description                                                                                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| total     | Total amount available in your Payment Order Limit that you can use to perform payment order. A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). |

<aside class="notice">
  In sandbox, you are limited to 1 request / 10 seconds. In production you are limited to 3 requests / 10 minutes.
</aside>

# X Signature

<aside class="notice">
  This section is not to be confused with V5 Checksum, please refer <a href="#v5-checksum">here</a> to find out how to generate a V5 Checksum.
</aside>

###### X SIGNATURE CALCULATION

The message level security is implemented by using signing and verification of the message.

###### STEP 1 - CONSTRUCT SOURCE STRING

- The source string should be formed with elements, construct with key and value pair data.
- The elements should then be sorted in ascending order, case-insensitive.
- Each element should be separated by a "|" (pipe) character in between them.
- Below is a sample of key value pairs:

  `collection_id: 'inbmmepb', description: 'testing', email: 'api@billplz.com', name: 'Michael', amount: '200', callback_url: 'https://example.com/webhook'`

- Below is sample of constructed source string by using key value pairs above:

  `amount200|callback_urlhttps://example.com/webhook|collection_idinbmmepb|descriptiontesting|emailapi@billplz.com|nameMichael`

- In the case of nested data, the parent key should be prepanded to the nested elements.

- Example of nested key value pairs:

  `collections: [{id: 'inbmmepb', title: 'testing'}, {id: 'xyzabc', title: 'testing x 2'}], page: 1`

- Will construct source string as below:

  `collectionsidinbmmepb|collectionsidxyzabc|collectionstitletesting|collectionstitletesting x 2|page1`

###### STEP 2 - SIGN THE SOURCE STRING

- Get your XSignature Key from setting page.
- Sign the constructed source string in step #1 using **HMAC_SHA256** and the shared XSignature Key.

Below is a sample of constructed source string:

`amount200|callback_urlhttps://example.com/webhook|collection_idinbmmepb|descriptiontesting|emailapi@billplz.com|nameMichael`

Below is the final x_signature value using above source string with 'abc123cde456' as XSignature Key:

`ef5e54a22af0925cba88fab467119742e90262e3646eea1dee3949938daf3a38`

<aside class="notice">
  Elements in the source string should not contain <code>x_signature</code>.
</aside>

### Extra Payment Completion Information

To get `transaction_id` and `transaction_status`, enable **Extra Payment Completion Information**.

![Activate the extra payment completion information.](x_spc_transaction_setting.png)

<aside class="notice">
  You are advised to activate this option when integrating with FPX B2B banks for better user experience.
</aside>

# V5 Checksum

V5 API introduces new security measures. Every request made in V5 endpoints must include **_epoch_** and **_checksum_** values in addition to each endpoint's required arguments.

- **Epoch** param must be in UNIX epoch time format
- **Checksum** calculation is specific to each endpoint, please refer to the **CHECKSUM ARGUMENTS** of each endpoint for more information on this.
- Checksum signature must be calculated using **HMAC_SHA512** together with your account **XSignature key**

###### \#1. Format the raw_string with only the values of the following keys in strict order

In the example of creating a payment order collection, the required values for checksum signature is **[ title, epoch ]** and assuming your parameters are `{title: "My payment order title", epoch: 1681724303}`, you are only required to join the values of the required arguments to form the raw string for checksum signature generation.

Your raw string would look like this `"My payment order title1681724303"`

###### \#2.Digest the raw_string together with your account XSignature key using **HMAC_SHA512**

Subsequently to get the valid checksum signature, you must calculate the raw string using **HMAC_SHA512** together with your account XSignature key

For example with the string above `"My payment order title1681724303"` and using a sample XSignature key: `S-R5t3Uw6SrwXNWyZV-naVHg`

The expected generated checksum signature would equal: `575c35c13ba37ccc2a434529e5082a71a574d304ba007592af44339d4436467d6a49107c95e51905cd80dce0f745760bd42fe73e2bc3bcd7ab79d07cc7fb4fa4`

###### \#( Extra ) Formating checksum with optional arguments

In some endpoints you may include certain optional arguments as well, these optional arguments may need to be included in the checksum signature formation following the strict order of the values indicated in each endpoints.

If you are not including these optional arguments in your parameters, you may omit the values in the checksum formation. The optional arguments for checksum formation are denoted with an asterisk (**\***) in each endpoint's checksum value guide.

Here's an example:

Assuming your parameters are `{title: "My payment order title", callback_url: 'https://myawesomeweb.site', epoch: 1681724303}`. Subsequently your raw string would look like this `"My payment order titlehttps://myawesomeweb.site1681724303"`

<aside class="notice">
  Each endpoint will indicate the necessary values required to format the string for checksum generation
</aside>

<aside class="notice">
  Optional values required to format the string for checksum generation are denoted with an asterisk (*)
</aside>

# Payment Completion

## Basic Callback Url

`callback_url` feature does not give your users the same instant bill update experience as of redirect_url feature, however **it does guarantee you 100% execution backend to backend update**.

`callback_url` feature integration is **compulsory** but `redirect_url` feature is **optional**.

\*\***_you are strongly advised to integrate both redirect_url feature (for better user experience) and callback_url feature (for 100% transaction update)_**

> Example Server Side Request from Billplz:

```
POST /webhook/ HTTP/1.1
Connection: close
Host: 127.0.0.1
Content-Length: 346
Content-Type: application/x-www-form-urlencoded

  id="W_79pJDk"
  &collection_id="599"
  &paid="true"
  &state="paid"
  &amount="200"
  &paid_amount="0"
  &due_at="2020-12-31"
  &email="api@billplz.com"
  &mobile="+60112223333"
  &name="MICHAEL API"
  &metadata[id]="9999"
  &metadata[description]="This is to test bill creation"
  &url="http://billplz.dev/bills/W_79pJDk"
  &paid_at="2015-03-09 16:23:59 +0800"
```

> Body formatted for readability.

###### SERVER SIDE REQUEST

Billplz will make a POST request to `callback_url` with Bill object upon payment completion (`failure` or `success`).

A bill that catched payment failure will be in `due` state.

By default, `Basic Callback URL` feature is disabled due to security reason.

<aside class="notice">
  Billplz account registered before September 2018 will have Basic Callback URL feature enabled by default.
</aside>

###### HTTP REQUEST

`POST {CALLBACK_URL}`

###### POST PARAMETER

| Parameter     | Description                                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------- |
| id            | ID that represents bill.                                                                                               |
| collection_id | ID that represents the collection where the bill belongs to.                                                           |
| paid          | Boolean value to tell if a bill has paid. It will return `false` for `due` and `deleted` bills; `true` for paid bills. |
| state         | State that representing the bill's status, possible states are `due`, `deleted`, and `paid`.                           |
| amount        | Bill's amount in positive integer, smallest currency unit (e.g 100 cents to charge RM 1.00).                           |
| paid_amount   | Bill's paid amount in positive integer, smallest currency unit (e.g 100 cents to charge RM 1.00).                      |
| due_at        | Due date for the bill, in format `YYYY-MM-DD`. Example, `2020-12-31`.                                                  |
| email         | The email address of the bill's recipient.                                                                             |
| mobile        | Recipient's mobile number, in format `+601XXXXXXXX`                                                                    |
| name          | Recipient's name.                                                                                                      |
| metadata      | Deprecated hash value data.                                                                                            |
| URL           | URL to the bill page.                                                                                                  |
| paid_at       | Date time when the bill was paid, in format `YYYY-MM-DD HH:MM:SS TimeZone`. Example, `2017-02-13 05:43:43 +0800`       |

Callback request will be timeout at 20 seconds.

Billplz server also expecting the end point server responds with status code of 200.

In a case of either the end point server does not able to respond within the limit seconds (20 secs) or does not respond with 200 status code, the callback will consider as failure.

On failure, the job is rescheduled for 2nd attempt in `15 seconds + (random 0-300 seconds)`, while 3rd and 4th attempts will rescheduled at `15 minutes + (random 0-300 seconds)`. The 5th (last) attempt will be `24 hours + random (0-300 seconds)` after 4th attempt.

Assuming the 1st callback is initiated at 13:00:00. The 2nd attempt will be at 13:00:15 + N seconds. The 3rd attempt will be at 13:15:15 + N seconds. The 4th attempt will be at 13:30:15 + N seconds. The fifth attempt will be at 13:30:15+ N seconds next day.

Billplz will attempt for maximum of 5 times and the callback will be removed from the system queue permanently after that.

<aside class="warning">
  YOUR <a href="#webhook-rank">ACCOUNT RANK</a> WILL BE DEGRADED BY 1 FOR EVERY UNSUCCESSFUL ATTEMPT OF CALLBACK AND WILL RESULT TO LOWER SCHEDULING PRIORITY FOR THE CALLBACK TO BE EXECUTED FOR YOUR ACCOUNT.
</aside>

<aside class="warning">
  YOU ARE STRONGLY ADVISED TO USE <a href="#payment-completion-x-signature-callback-url">X SIGNATURE CALLBACK URL</a>. USING X SIGNATURE ALLOWS YOU TO VALIDATE THE DATA INTEGRITY BY USING CHECKSUM.
</aside>

## Basic Redirect Url

`redirect_url` feature is just a front-end redirection to `redirect_url` specified, **it does not guaranteed the execution**, due to many factors (user disconnected, browser closed, app hang, etc).

But it is good to have, to provide better user experience (front-end instant bill status update)

`callback_url` feature integration is **compulsory** but `redirect_url` feature is **optional**.

\*\***_you are strongly advised to integrate both redirect_url feature (for better user experience) and callback_url feature (for 100% transaction update)_**

###### CLIENT SIDE REQUEST

If `redirect_url` exists, Billplz will redirect the browser to `redirect_url`

By default, `Basic Redirect URL` feature is disabled due to security reason.

<aside class="notice">
  Billplz account registered before September 2018 will have Basic Redirect URL feature enabled by default.
</aside>

###### HTTP REQUEST

`GET {REDIRECT_URL}?billplz[id]=W_79pJDk`

###### URL PARAMETER

| Parameter   | Description              |
| ----------- | ------------------------ |
| billplz[id] | ID that represents bill. |

<aside class="warning">
  YOU ARE STRONGLY ADVISED TO USE <a href="#payment-completion-x-signature-redirect-url">X SIGNATURE REDIRECT URL</a>. USING X SIGNATURE ALLOWS YOU TO VALIDATE THE DATA INTEGRITY BY USING CHECKSUM.
</aside>

## X Signature Callback Url

`callback_url` feature does not give you the same instant bill update experience as of redirect_url feature, however **it does guarantee you 100% execution backend to backend update**.

`callback_url` feature integration is **compulsory** but `redirect_url` feature is **optional**.

\*\***_you are strongly advised to integrate both redirect_url feature (for better user experience) and callback_url feature (for 100% transaction update)_**

###### SERVER SIDE REQUEST

Billplz will make a POST request to `callback_url` with Bill object upon payment completion (`failure` or `success`).

X Signature Callback URL request created through the API by Billplz can be verified by calculating a digital signature - `x_signature`.

Each callback request includes a `x_signature` which is generated using HMAC-SHA256 algorithm and shared XSignature Key, along with the data sent in the request.

To verify that the request came from Billplz, compute the HMAC-SHA256 digest according to the <a href="#x-signature">X Signature section</a> and compare it to the value in the `x_signature` parameter. If they match, you can be sure that the callback was sent from Billplz and the data has not been compromised.

A bill that catched payment failure will be in `due` state.

<aside class="success">
  By default, X Signature Callback URL feature is activated for account registered starting September 2018.
</aside>

> Example Server Side Request from Billplz:

```
POST /webhook/ HTTP/1.1
Connection: close
Host: 127.0.0.1
Content-Length: 346
Content-Type: application/x-www-form-urlencoded

  id="W_79pJDk"
  &collection_id="599"
  &paid="true"
  &state="paid"
  &amount="200"
  &paid_amount="0"
  &due_at="2020-12-31"
  &email="api@billplz.com"
  &mobile="+60112223333"
  &name="MICHAEL API"
  &url="http://billplz.dev/bills/W_79pJDk"
  &paid_at="2015-03-09 16:23:59 +0800"
  &x_signature="f0ff6c564f98d5403e2b26fbd3d45309c76eb68d8c5bcda0d48b541c3502a396"
```

> Body formatted for readability.

> Example Server Side Request with transaction from Billplz:

```
POST /webhook/ HTTP/1.1
Connection: close
Host: 127.0.0.1
Content-Length: 376
Content-Type: application/x-www-form-urlencoded

  id="pjpetpdy"
  &collection_id="bvgo7ueb"
  &paid="true"
  &state="paid"
  &amount="100"
  &paid_amount="100"
  &due_at="2020-8-7"
  &email="api@billplz.com"
  &mobile=""
  &name="WILL"
  &url="http://www.billplz.test:3000/bills/pjpetpdy"
  &paid_at="2020-08-07 15:08:19 +0800"
  &transaction_id="711044E05418"
  &transaction_status="completed"
  &x_signature="b5ca81d0f1b87ab3b17580236735c68af92936de9aaa588751a4371ac4944a56"
```

> Body formatted for readability.

###### HTTP REQUEST

`POST {CALLBACK_URL}`

###### POST PARAMETER

| Parameter          | Description                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id                 | ID that represents bill.                                                                                                                                                                       |
| collection_id      | ID that represents the collection where the bill belongs to.                                                                                                                                   |
| paid               | Boolean value to tell if a bill has paid. It will return `false` for `due` and `deleted` bills; `true` for `paid` bills.                                                                       |
| state              | State that representing the bill's status, possible states are `due`, `deleted` and `paid`.                                                                                                    |
| amount             | Bill's amount in positive integer, smallest currency unit (e.g 100 cents to charge RM 1.00).                                                                                                   |
| paid_amount        | Bill's paid amount in positive integer, smallest currency unit (e.g 100 cents to charge RM 1.00).                                                                                              |
| due_at             | Due date for the bill, in format `YYYY-MM-DD`. Example, `2020-12-31`.                                                                                                                          |
| email              | The email address of the bill's recipient.                                                                                                                                                     |
| mobile             | Recipient's mobile number.                                                                                                                                                                     |
| name               | Recipient's name.                                                                                                                                                                              |
| URL                | URL to the bill page.                                                                                                                                                                          |
| paid_at            | Date time when the bill was paid, in format `YYYY-MM-DD HH:MM:SS TimeZone`. Example, `2015-03-09 16:23:59 +0800`.                                                                              |
| transaction_id     | ID that represent the transaction. (Enable `Extra Payment Completion Information` option to receive this parameter)                                                                            |
| transaction_status | Status that representing the transaction's status, possible statuses are `pending`, `completed` and `failed`. (Enable `Extra Payment Completion Information` option to receive this parameter) |
| x_signature        | Digital signature computed with posted data and shared XSignature Key.                                                                                                                         |

Callback request will be timeout in 20 seconds.

Billplz server also expecting the end point server responds with status code of 200.

In a case of either the end point server does not able to respond within the limit seconds (20 secs) or does not respond with 200 status code, the callback will consider as failure.

On failure, the job is rescheduled for 2nd attempt in `15 seconds + (random 0-300 seconds)`, while 3rd and 4th attempts will rescheduled at `15 minutes + (random 0-300 seconds)`. The 5th (last) attempt will be `24 hours + random (0-300 seconds)` after 4th attempt.

Assuming the 1st callback is initiated at 13:00:00. The 2nd attempt will be at 13:00:15 + N seconds. The 3rd attempt will be at 13:15:15 + N seconds. The 4th attempt will be at 13:30:15 + N seconds. The fifth attempt will be at 13:30:15+ N seconds next day.

Billplz will attempt for maximum of 5 times and the callback will be removed from the system queue permanently after that.

###### SAMPLE HTTP REQUEST FROM X SIGNATURE CALLBACK URL FEATURE

`POST {CALLBACK_URL}`

with POST body,
`{:id=>"zq0tm2wc", :collection_id=>"yhx5t1pp", :paid=>true, :state=>"paid", :amount=>100, :paid_amount=>100, :due_at=>"2018-9-27", :email=>"tester@test.com", :mobile=>nil, :name=>"TESTER", :url=>"http://www.billplz-sandbox.com/bills/zq0tm2wc", :paid_at=>"2018-09-27 15:15:09 +0800", :x_signature=>"0fe0a20b8d557eeae570377783d062a3816a9ea80f368860bacfa7ec3ca4d00e"}`

######\#1 Extract all key-value pair parameters except x_signature.

`id="zq0tm2wc"`

`collection_id="yhx5t1pp"`

`paid="true"`

`state="paid"`

`amount="100"`

`paid_amount="100"`

`due_at="2018-9-27"`

`email="tester@test.com"`

`mobile=""`

`name="TESTER"`

`url="http://www.billplz-sandbox.com/bills/zq0tm2wc"`

`paid_at="2018-09-27 15:15:09 +0800"`

######\#2 Construct source string from each key-value pair parameters above.

`idzq0tm2wc`

`collection_idyhx5t1pp`

`paidtrue`

`statepaid`

`amount100`

`paid_amount100`

`due_at2018-9-27`

`emailtester@test.com`

`mobile`

`nameTESTER`

`urlhttp://www.billplz-sandbox.com/bills/zq0tm2wc`

`paid_at2018-09-27 15:15:09 +0800`

######\#3 Sort in ascending order, case-insensitive.

`amount100`

`collection_idyhx5t1pp`

`due_at2018-9-27`

`emailtester@test.com`

`idzq0tm2wc`

`mobile`

`nameTESTER`

`paid_amount100`

`paid_at2018-09-27 15:15:09 +0800`

`paidtrue`

`statepaid`

`urlhttp://www.billplz-sandbox.com/bills/zq0tm2wc`

######\#4 Combine sorted source strings with "|" (pipe) character in between.

`amount100|collection_idyhx5t1pp|due_at2018-9-27|emailtester@test.com|idzq0tm2wc|mobile|nameTESTER|paid_amount100|paid_at2018-09-27 15:15:09 +0800|paidtrue|statepaid|urlhttp://www.billplz-sandbox.com/bills/zq0tm2wc`

######\#5 Compute x_signature by signing the final source in #4 using HMAC_SHA256 and the sample XSignature Key.

`S-s7b4yWpp9h7rrkNM1i3Z_g`

`0fe0a20b8d557eeae570377783d062a3816a9ea80f368860bacfa7ec3ca4d00e`

######\#6 Compare the computed x_signature with the x_signature passed in the request.

If they match, you can be sure that the callback was sent from Billplz and the data has not been compromised, and you **do not need to trigger additional Get a Bill API for primary status validation**.

<aside class="warning">
  YOUR ACCOUNT RANK WILL BE DEGRADED BY 1 FOR EVERY UNSUCCESSFUL ATTEMPT OF CALLBACK AND WILL RESULT TO LOWER SCHEDULING PRIORITY FOR THE CALLBACK TO BE EXECUTED FOR YOUR ACCOUNT.
</aside>

## X Signature Redirect Url

`redirect_url` feature is just a front-end redirection to the `redirect_url` specified, **it does not guaranteed the execution**, due to many factors (user disconnected, browser closed, app hang, etc).

But it is good to have, to provide better user experience (front-end instant bill status update)

`callback_url` feature integration is **compulsory** but `redirect_url` feature is **optional**.

\*\***_you are strongly advised to integrate both redirect_url feature (for better user experience) and callback_url feature (for 100% transaction update)_**

###### CLIENT SIDE REQUEST

If `redirect_url` exists, Billplz will redirect the browser to `redirect_url`

X Signature Redirect URL request created by Billplz can be verified by calculating a digital signature - `x_signature`.

Each redirect request includes a x_signature which is generated using HMAC-SHA256 algorithm and shared XSignature Key, along with the parameters passed in the request.

To verify that the request came from Billplz, compute the HMAC-SHA256 digest according to the <a href="#x-signature">X Signature section</a> and compare it to the value in the `x_signature` parameter. If they match, you can be sure that the redirection was sent from Billplz and the data has not been compromised.

<aside class="success">
  By default, X Signature Redirect URL feature is activated for account registered starting September 2018.
</aside>

###### HTTP REQUEST

`GET {REDIRECT_URL}?billplz[id]=ifpgaa&billplz[paid]=true&billplz[paid_at]=2017-01-04%2013%3A10%3A45%20%2B0800&billplz[x_signature]=76de0ad8cc23706d694fd81c9027b8e3ae169e0880d75e1e123c5832c82bf707`

###### URL PARAMETER

| Parameter                   | Description                                                                                                                                                                                    |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| billplz[id]                 | ID that represents bill.                                                                                                                                                                       |
| billplz[paid]               | Boolean value to tell if a bill has paid. It will return `false` for `due` and `deleted` bills; `true` for paid bills.                                                                         |
| billplz[paid_at]            | Date time when the bill was paid, in format `YYYY-MM-DD HH:MM:SS TimeZone`. Example, `2017-01-04 13:10:45 +0800`.                                                                              |
| billplz[transaction_id]     | ID that represent the transaction. (Enable `Extra Payment Completion Information` option to receive this parameter)                                                                            |
| billplz[transaction_status] | Status that representing the transaction's status, possible statuses are `pending`, `completed` and `failed`. (Enable `Extra Payment Completion Information` option to receive this parameter) |
| billplz[x_signature]        | Digital signature computed with passing parameters and shared XSignature Key.                                                                                                                  |

###### SAMPLE HTTP REQUEST FROM X SIGNATURE REDIRECT URL FEATURE

`GET {REDIRECT_URL}?billplz[id]=zq0tm2wc&billplz[paid]=true&billplz[paid_at]=2018-09-27%2015%3A15%3A09%20%2B0800&billplz[x_signature]=4aab095fe5a39b1d534500988f9a0cb085cd1b6d5bbb55dd4e02ea6fa102b47b`

######\#1 Extract all key-value pair parameters except x_signature.

`billplz[id]="zq0tm2wc"`

`billplz[paid]="true"`

`billplz[paid_at]="2018-09-27 15:15:09 +0800"`

######\#2 Construct source string from each key-value pair parameters above.

`billplzidzq0tm2wc`

`billplzpaidtrue`

`billplzpaid_at2018-09-27 15:15:09 +0800`

######\#3 Sort in ascending order, case-insensitive.

`billplzidzq0tm2wc`

`billplzpaid_at2018-09-27 15:15:09 +0800`

`billplzpaidtrue`

######\#4 Combine all source strings with "|" (pipe) character in between.

`billplzidzq0tm2wc|billplzpaid_at2018-09-27 15:15:09 +0800|billplzpaidtrue`

######\#5 Compute x_signature by signing the final source in #4 using HMAC_SHA256 and the sample XSignature Key.

`S-s7b4yWpp9h7rrkNM1i3Z_g`

`4aab095fe5a39b1d534500988f9a0cb085cd1b6d5bbb55dd4e02ea6fa102b47b`

######\#6 Compare parameter x_signature passed in the request with the computed x_signature in #5.

If they match, you can be sure that the redirection was sent from Billplz and the data has not been compromised, and you **do not need to trigger additional Get a Bill API for primary status validation**.

## Payment Order Callback

Payment order callbacks allows your system to be notified in the event of the payment order status change. This will ensure that your system can be hooked to perform an action in the event of payment order status. The callback notification will be send when the status changed to `completed` or `refunded`.

###### SERVER SIDE REQUEST

If `callback_url` exists in your payment order collection, Billplz will make a POST request with Payment Order object upon status change (`completed` or `refunded`).

Each callback request includes a checksum which is generated using HMAC-SHA512 algorithm and your XSignature Key, along with certain key data sent in the request. The data integrity of the data from Billplz can be verified by calculating a checksum signature and comparing with the checksum value included within the posted callback object.

If they match, you can be sure that the callback was sent from Billplz and the data has not been compromised.

> Example Server Side Request from Billplz:

```
POST /webhook/ HTTP/1.1
Connection: close
Host: 127.0.0.1
Content-Length: 346
Content-Type: application/x-www-form-urlencoded

  id="0b924e37-7418-4d17-b234-8de424dc48e5"
  &payment_order_collection_id="0c78f8c6-5bd3-4663-8c08-9e2c805418a2"
  &bank_code="PHBMMYKL"
  &bank_account_number="1234567890"
  &name="Michae Yap"
  &description="Maecenas eu placerat ante."
  &email="api@billplz.com"
  &status="refunded"
  &notification="false"
  &recipient_notification="true"
  &reference_id="My first payment order"
  &display_name=""
  &total=50000
  &epoch=1681895891
  &checksum="2720f5ef16c7d04677829789fb74bccb08b90041e4e27916d85cb6fbbece58a7ab48538e8b62bcedab3b236bd38e6517860b593b8fe9bfa77bed979994f2ca1a"
```

> Body formatted for readability.

###### HTTP REQUEST

`POST {CALLBACK_URL}`

###### POST PARAMETER

| Parameter                   | Description                                                                                                                 |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| id                          | ID that represents payment order.                                                                                           |
| payment_order_collection_id | The payment order collection's id in string format.                                                                         |
| bank_code                   | Bank Code that represents bank, in string value.                                                                            |
| bank_account_number         | Bank account number, in string value.                                                                                       |
| name                        | Payment Order's recipient name.                                                                                             |
| description                 | The Payment Order's description.                                                                                            |
| email                       | The email address of recipient.                                                                                             |
| status                      | Payment Order status. It is either `processing` or `completed` or `refunded` or `cancelled`.                                |
| notification                | Boolean value.                                                                                                              |
| recipient_notification      | Boolean value.                                                                                                              |
| reference_id                | Payment Order's reference ID. Useful for identification on recipient part.                                                  |
| display_name                | Bank Account Owner's name                                                                                                   |
| total                       | Total amount transfer to the recipient. A positive integer in the smallest currency unit (e.g 100 cents to charge RM 1.00). |
| epoch                       | Current time in UNIX epoch time format                                                                                      |
| checksum                    | Computed signature of strict ordered values of key data present in object                                                   |

Callback request will be timeout in 20 seconds.

Billplz server also expecting the end point server responds with status code of 200.

In a case of either the end point server does not able to respond within the limit seconds (20 secs) or does not respond with 200 status code, the callback will consider as failure.

On failure, the job is rescheduled for a 2nd attempt in one hour

Billplz will attempt for maximum of 2 times and the callback will be removed from the system queue permanently after that.

> Sample of callback POST request Payment Order Object from Billplz:

```json
{
  "id": "0b924e37-7418-4d17-b234-8de424dc48e5",
  "payment_order_collection_id": "0c78f8c6-5bd3-4663-8c08-9e2c805418a2",
  "bank_code": "PHBMMYKL",
  "bank_account_number": "1234567890",
  "name": "Michae Yap",
  "description": "Maecenas eu placerat ante.",
  "email": "api@billplz.com",
  "status": "refunded",
  "notification": "false",
  "recipient_notification": "true",
  "reference_id": "My first payment order",
  "display_name": "",
  "total": 50000,
  "epoch": 1681895891,
  "checksum": "2720f5ef16c7d04677829789fb74bccb08b90041e4e27916d85cb6fbbece58a7ab48538e8b62bcedab3b236bd38e6517860b593b8fe9bfa77bed979994f2ca1a"
}
```

###### CHECKSUM FORMAT GUIDE

In order to compare with the checksum sent together with the payment order object, you must first generate the checksum signature on your end first. _Required values for [checksum signature](#v5-checksum) in this order:_ **[id, bank_account_number, status, total, reference_id, epoch]**

Here's an example of a HTTP request from a payment order callback

`POST {CALLBACK_URL}` with POST body

`{:id=>"0b924e37-7418-4d17-b234-8de424dc48e5",:payment_order_collection_id=>"0c78f8c6-5bd3-4663-8c08-9e2c805418a2",:bank_code=>"PHBMMYKL",:bank_account_number=>"1234567890",:name=>"Michae Yap",:description=>"Maecenas eu placerat ante.",:email=>"api@billplz.com",:status=>"refunded",:notification=>true,:recipient_notification=>true,:reference_id=>"My first payment order",:display_name=>nil,:total=>500000,:epoch=>1681895891, :checksum=>"2720f5ef16c7d04677829789fb74bccb08b90041e4e27916d85cb6fbbece58a7ab48538e8b62bcedab3b236bd38e6517860b593b8fe9bfa77bed979994f2ca1a"}`

###### \#1. Format the raw_string with only the values of the following keys in strict order

object_keys: `[:id, :bank_account_number, :status, :total, :reference_id, :epoch]`

raw_string: `"0b924e37-7418-4d17-b234-8de424dc48e51234567890refunded500000My first payment order1681895891"`

###### \#2.Digest the raw_string together with your account XSignature key using **HMAC_SHA512**

example_XSignature: `S-R5t3Uw6SrwXNWyZV-naVHg`

generated_checksum_value: `2720f5ef16c7d04677829789fb74bccb08b90041e4e27916d85cb6fbbece58a7ab48538e8b62bcedab3b236bd38e6517860b593b8fe9bfa77bed979994f2ca1a`

###### \#3.Compare your generated checksum signature with the one included in the callback

if the value of your checksum is the same as the one included within the callback object, you can be sure that the request is valid from Billplz and the data is not altered.

<aside class="notice">
  The order of the values must be in the specific order as noted above
</aside>

<aside class="warning">
  YOUR ACCOUNT RANK WILL BE DEGRADED BY 1 FOR EVERY UNSUCCESSFUL ATTEMPT OF CALLBACK AND WILL RESULT TO LOWER SCHEDULING PRIORITY FOR THE CALLBACK TO BE EXECUTED FOR YOUR ACCOUNT.
</aside>

# Rate Limit

> Example request:

```shell
curl https://www.billplz.com/api/v3/bills/8X0Iyzaw \
  -u 73eb57f0-7d4e-42b9-a544-aeac6e4b0f81:
```

> Example response header without rate limit:

```
RateLimit-Limit: unlimited
RateLimit-Remaining: unlimited
RateLimit-Reset: unlimited
Content-Type: application/json;
```

> Example response header with rate limit:

```
RateLimit-Limit: 100
RateLimit-Remaining: 99
RateLimit-Reset: 299
Content-Type: application/json;
```

> Example response after exceeding limit:

```
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 299
Content-Type: application/json;
```

```json
{
  "error": {
    "type": "RateLimit",
    "message": "Too many requests"
  }
}
```

All **API GET method** Endpoints are subject to rate limit.

The limit can be either **100 requests per request window (5 minutes period)** or **10 requests per request window (5 minutes period)**, depending on the level of abusion. The limit is cumulative either per IP address or account, and is not counted per API Endpoint Basis. Requesting for specific API Endpoint will reduce the limit for another API Endpoint within a request window.

<aside class="notice">
  We will set a rate limit for specific IP Address or per account basis if the usage appears to be abusive. We reserve the rights, at our sole discretion to set this rate-limit without any further notices.
</aside>

###### RESPONSE HEADER

| Parameter           | Description                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| RateLimit-Limit     | Total requests allowed per request window. It's either `unlimited`, or `100` or `10`.                                               |
| RateLimit-Remaining | Total available requests allowed per request window. It's either `unlimited`, or less than or equal to `RateLimit-Limit` parameter. |
| RateLimit-Reset     | Time remaining (in seconds) for next request window restart. It's either `unlimited`, or less than or equal to `300`.               |

<aside class="notice">
  Exceeding the rate limit will result to <code>429</code> Too Many Requests.
</aside>

<aside class="success">
  Normal HTTP status code will be <code>200</code>.
</aside>

###### RESPONSE PARAMETER

Response parameter will be as usual if didn't exceed the rate limit. Otherwise, the response parameters will be as follows:

| Parameter      | Value               |
| -------------- | ------------------- |
| error[type]    | `RateLimit`         |
| error[message] | `Too many requests` |
