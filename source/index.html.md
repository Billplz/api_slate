---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - php

toc_footers:
  - <a href='https://www.billplz.com'>Sign Up to Start</a>
  - <a href='https://www.billplz-sandbox.com'>Sign Up to Sandbox</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Billplz API! You can use our API to access Billplz API endpoints to start sending bills and collection payment.

Our API is organized around REST. JSON will be returned in all responses from the API, including errors. The API will accept both application/x-www-form-urlencoded and application/json.

Premium Plan has been discontinued on 24 September 2018. All API features are available to all eligible* accounts.

# Authentication

> To authorize, use this code:

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
> If authenticated, you should not see Access denied response.

> curl uses the -u flag to pass basic auth credentials (adding a colon after your API key will prevent it from asking you for a password).

You authenticate to the Billplz API by providing your API Secret Keys in the request. You can get your API keys from your account’s settings page.

Authentication to the API occurs via HTTP Basic Auth. Provide your API key as the basic auth username. You do not need to provide a password.

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. You must authenticate for all requests.

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

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

##### HTTP Request

`GET http://example.com/api/kittens`

##### Query Parameters

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

##### HTTP Request

`GET http://example.com/kittens/<ID>`

##### URL Parameters

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

##### HTTP Request

`DELETE http://example.com/kittens/<ID>`

##### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

