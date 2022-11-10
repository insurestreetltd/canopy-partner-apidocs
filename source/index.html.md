---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript
  - php
  # - ruby
  # - python

# toc_footers:
# - <a href='#'>Sign Up for a Developer Key</a>
# - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the Canopy API for Partners! We follow standard behaviour in terms of URL's, JSON request/response bodies where applicable and standard HTTP error codes.

# Requesting Your Credentials

Credentials are provided on request by Canopy to yourselves. Please speak to your account manager here to obtain the details for the environments.

# Using API key

In the credentials you were sent you should have API key. Canopy expects for the API key to be included in all requests to the API in a header that looks like the following:

`x-api-key: ePYgsiGbWF5aAHBj0xT9Pa1k5li0NPMD25PQXbAC`

# Requesting an API Token

> To get API token, use this code:

```javascript
const jwt = require("jsonwebtoken");
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const partnerId = "your_partner_id";
const partnerSecretKey = "your_secret_key";
const canopyApiKey = "your_api_key";

// Function that generates jwt token payload using partnerId
const generatePayload = () => {
  const now = Math.floor(Date.now() / 1000); // sec
  const expires = now + 60 * 60;
  const payload = {
    iss: "canopy.rent",
    scope: "request.write_only document.read_only",
    aud: `referencing-requests/partner/${partnerId}/token`,
    exp: expires,
    iat: now,
  };

  return payload;
};

const authenticate = async () => {
  // Generate jwt key
  const jwtKey = await jwt.sign(generatePayload(), partnerSecretKey);

  // Send POST request
  return axios({
    url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/token`,
    method: "POST",
    headers: {
      // Add API key from the credentials into headers
      "x-api-key": canopyApiKey,
    },
    data: {
      jwtKey: jwtKey,
    },
  });
};

authenticate();
```

```php
<?php
require_once 'vendor/autoload.php';

// For JWT generation we used Firebase PHP-JWT library: https://github.com/firebase/php-jwt
use \Firebase\JWT\JWT;

$partnerId = "your_partner_id";
$secretKey = "your_secret_key";

$now = time();

// Generate JWT payload
$payload = array(
    "iss" => "canopy.rent",
    "scope" => "request.write_only document.read_only",
    "aud" => "referencing-requests/partner/$partnerId/token",
    "exp" => $now + 60 * 60,
    "iat" => $now
);

// Generate JWT key using payload and secret key
$jwt = JWT::encode($payload, $secretKey);

echo $jwt, "\n";

$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";

// We used Guzzle as HTTP client: https://docs.guzzlephp.org/en/stable/#
$client = new \GuzzleHttp\Client();

// Send POST request to get access token with your api key and generated JWT key
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/token",
  [
    "headers" => [
      "Content-Type" => "application/json",
      "x-api-key" => $apiKey
    ],
    "json" => ["jwtKey" => $jwt]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "access_token": "Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ",
  "expires": 1594907203
}
```

### HTTP Request

`POST /referencing-requests/partner/:partnerId/token`

### Body Parameters

| Parameter | Type   | Required | Description       |
| --------- | ------ | -------- | ----------------- |
| jwtKey    | string | true     | Generated jwt key |

### Using API token

You need to request an API token to make calls to the Canopy API. Canopy expects for the API token to be included in all requests to the API in a header that looks like the following:

`Authorization: Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ`

# Refresh Secret Key

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/refresh`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    secretKey: "new_secret_key",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/partner/refresh",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => ["secretKey" => "new_secret_key"]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "secretKey": "new_secret_key"
  }
}
```

This endpoint may be called to change current `secretKey` (which is required for `jwtKey` generation) and get back freshly generated one. Accepts currently active `secretKey` in request body. After receiving successful response with new `secretKey`, previous `secretKey` becomes stale and should not be used for `jwtKey` generation.

### HTTP Request

`POST /referencing-requests/partner/refresh`

### Body Parameters

| Parameter | Type   | Required | Description                 |
| --------- | ------ | -------- | --------------------------- |
| secretKey | string | true     | Currently active secret key |

# Check permission

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const apiKey = "your_api_key";
const authorizationToken = "authorization_token";
const partnerId = "partner_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/permission?email=example@gmail.com`,
  method: "GET",
  headers: {
    "Authorization": authorizationToken,
    "x-api-key": apiKey,
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$partnerId = "your_partner_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "GET",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/permission?email=example@gmail.com",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "renterId": "UUID",
  }
}
```

This endpoint may be called to check permission by email

### HTTP Request

`GET /referencing-requests/partner/:partnerId/permission?email=example@gmail.com`

### URL Parameters

| Parameter  | Description            |
| ---------- | ---------------------- |
| partnerId  | Your partner reference |

### Query Parameters

| Parameter  | Required | Description            |
| ---------- | -------- | ---------------------- |
| email      | true     | Renter email           |


# Request permission

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const apiKey = "your_api_key";
const authorizationToken = "authorization_token";
const partnerId = "partner_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/permission`,
  method: "POST",
  headers: {
    "Authorization": authorizationToken,
    "x-api-key": apiKey,
  }
  data: {
    email: "example@gmail.com"
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$partnerId = "your_partner_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/permission",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "example@gmail.com",
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "renterId": "UUID",
    "permissionRequestId": "UUID",
  }
}
```

This endpoint creates new permission request.


### HTTP Request

`POST /referencing-requests/partner/:partnerId/permission`

### URL Parameters

| Parameter  | Description            |
| ---------- | ---------------------- |
| partnerId  | Your partner reference |


# Create connection

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const apiKey = "your_api_key";
const authorizationToken = "authorization_token";
const partnerId = "partner_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/branch-connection`,
  method: "POST",
  headers: {
    "Authorization": authorizationToken,
    "x-api-key": apiKey,
  }
  data: {
    branchId: "UUID",
    renterId: "UUID",
    listingName: "String",
    listingUrl: "URL String",
    propertyAddress: "String",
    inquiryType: "String",
    inquiryTimestamp: "timestamp",
    renterMessage: "String",
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$partnerId = "your_partner_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/branch-connection",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "branchId" => "UUID",
      "renterId" => "UUID",
      "listingName" => "String",
      "listingUrl" => "String",
      "propertyAddress" => "String",
      "renterMessage" => "String",
      "inquiryType" => "String",
      "inquiryTimestamp" => "timestamp",
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "HQlint": "URL",
  }
}
```

This endpoint creates new connection between renter and agent by branchId.


### HTTP Request

`POST /referencing-requests/partner/:partnerId/branch-connection`

### URL Parameters

| Parameter  | Description            |
| ---------- | ---------------------- |
| partnerId  | Your partner reference |

### Body Parameters

| Parameter          | Type     | Required | Description                                                                    |
| ------------------ | -------- | -------- | ------------------------------------------------------------------------------ |
| branchId           | string   | true     |  |                                                                                      
| renterId           | string   | true     | canopy renter id  |                                                                     
| listingName        | string   | false    |  |                                                                    
| listingUrl         | string   | false    |  |                                                                    
| propertyAddress    | string   | false    |  |                                                                    
| inquiryType        | string   | false    |  |                                                                  
| inquiryTimestamp   | string   | false    |  |                                                                  
| renterMessage      | string   | false    |  |


# Webhooks Endpoints

## Register Webhook

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const partnerId = "partner_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/webhook/register`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    type: "OVERALL_STATUS_UPDATES",
    callbackUrl: "https://callbackurl.com",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$partnerId = "your_partner_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/webhook/register",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "type" => "OVERALL_STATUS_UPDATES",
      "callbackUrl" => "https://callbackurl.com",
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "webhookType": "OVERALL_STATUS_UPDATES",
  "callbackUrl": "https://callbackurl.com"
}
```

This endpoint registers the webhook with the appropriate type in Canopy system. After that, Rent Passport updates will be sent to the specified callback url.

### HTTP Request

`POST /referencing-requests/partner/:partnerId/webhook/register`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| partnerId  | Your partner reference |

### Body Parameters

| Parameter          | Type     | Required | Description                                                                    |
| ------------------ | -------- | -------- | ------------------------------------------------------------------------------ |
| type               | string   | true     | Specifies which type of updates will be sent from Canopy. Could be one of ["OVERALL_STATUS_UPDATES", "REQUEST_STATUS_UPDATES"]                                                                                      
| callbackUrl        | string   | true     | The URL of the webhook endpoint  |


### Event types

If you subscribed to the `REQUEST_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` each time one of the following events trigger:

| Event                                      | Description                                         |
| ------------------------------------------ | --------------------------------------------------- |
| `INVITED`                                  | The user was invited to permission                     |
| `INVITE_RESENT`                            | Resent invitation email for the user                |
| `PERMISSIONED`                             | User accepts the permission                         |
| `PERMISSION_REJECTED`                      | User rejects the permission                         |
| `PERMISSION_STOPPED`                       | User stops the permission                           |


Once `REQUEST_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

| Parameter           | Type   |
| ------------------- | ------ |
| canopyRenterId      | UUID   |
| permissionRequestId | UUID   |
| partnerId           | UUID   |
| notes               | string |

If you are subscribed to the `OVERALL_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` when one of the following Rent Passport will be complete

### Notification format

Once `OVERALL_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

| Parameter          | Type   | Description                                                                              |
| ------------------ | ------ | ---------------------------------------------------------------------------------------- |
| canopyRenterId     | string |                                                                                          |
| partnerReferenceId | string |                                                                                          |
| overallStatus      | string | One of ["NOT_STARTED", "IN_PROGRESS", "ACCEPT", "CONSIDER", "HIGH_RISK"]                 |

## Unregister Webhook

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const partnerId = "partner_id";
const webhookType = "OVERALL_STATUS_UPDATES";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/partner/${partnerId}/webhook/${webhookType}`,
  method: "DELETE",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "DELETE",
  "$canopyEndpointBaseUri/referencing-requests/partner/$partnerId/webhook/$webhookType",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "webhookType": "OVERALL_STATUS_UPDATES"
}
```

This endpoint unregisters webhook and stops sending Rent Passport updates to the specified callback URL.

### HTTP Request

`DELETE /referencing-requests/partner/:partnerId/webhook/:webhookType`

### URL Parameters

| Parameter   | Description                                    |
| ----------- | ---------------------------------------------- |
| partnerId   | Your partner reference                         |
| webhookType | The type of the webhook you want to unregister |
