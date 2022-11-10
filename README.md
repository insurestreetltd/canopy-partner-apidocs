![Logo](/canopylogo.png "Logo")

# API Documentation

## Introduction

The Canopy API is REST based.

https://en.wikipedia.org/wiki/Representational_state_transfer

We follow standard behaviour in terms of URL's, JSON request/response bodies where applicable and standard HTTP error codes.

## Environment Details

Canopy has three environments. Each environment has its own URL, which will be used by external partners in order to test functionality and then use
it on production:

- Development
- Staging
- Production

## Requesting Your Credentials

Credentials are provided on request by Canopy to yourselves. Please speak to your account manager here to obtain the details for the environments.

## Requesting an API Token

You need to request an API token to make calls to the Canopy API. In order to do this you need to:

1. Generate a payload using the partnerId from the credentials you have been sent. The example here uses Javascript:

   ```
   function generatePayload(partnerId) {
      const now = Math.floor(Date.now() / 1000); // sec
        const expires = now + 60 * 60;
        var payload = {
            iss: 'canopy.rent',
            scope: 'request.write_only document.read_only',
            aud: `referencing-requests/partner/${partnerId}/token`,
            exp: expires,
            iat: now
        };
      return payload;
    }
   ```

2. You need to sign this with JWT (e.g. jsonwebtoken in Javascript) using the secretKey in the credentials you were sent

   ```
   // In this example we use jsonwebtoken library for node.js: https://www.npmjs.com/package/jsonwebtoken
   const jwtKey = jwt.sign(generatePayload(config), secretKey);
   ```

   Full example of generating JWT key using PHP:

   ```
    <?php
    // In this example we use Firebase PHP-JWT library for PHP: https://github.com/firebase/php-jwt
    use \Firebase\JWT\JWT;


    $partnerId = "partner_id";
    $secretKey = "secret_key";

    $now = time();

    $payload = array(
        "iss" => "canopy.rent",
        "scope" => "request.write_only document.read_only",
        "aud" => "referencing-requests/partner/$partnerId/token",
        "exp" => $now + 60 * 60,
        "iat" => $now
    );

    $jwt = JWT::encode($payload, $secretKey);

    print_r($jwt);
    ?>
   ```

3. Finally make a POST request to the following endpoint:

   ```
   POST /referencing-requests/partner/:partnerId/token
   ```

   Include x-api-key header with the apiKey from the credentials you were sent:

   ```
   x-api-key: apiKey
   ```

   Add generated JWT key in the request body:

   ```
    {
      "jwtKey": "generated_jwt_key"
    }
   ```

   If the request is successful, the response body will contain the token for future API requests with an expires timestamp:

   ```
   ...
   Response: {
     "success": true,
     "access_token": "Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ",
     "expires": 1594907203
   }
   ...
   ```

## Using the Authorization Token

You need to include the x-api-key and authorization token in the headers with all Canopy API endpoint requests.

```
x-api-key: key
Authorization: token
```

This token has an expiryTime of 20 minutes, after which you will receive an authorization error, and you will then need to retrieve a new token.

## Refresh Secret Key

Requires authorization. This endpoint may be called to change current `secretKey` and get back freshly generated one. Accepts currently active `secretKey` in request body. After receiving successful response with new `secretKey`, previous `secretKey` becomes stale and should not be used for generation `jwtKey`.

```
POST /referencing-requests/partner/refresh
```

Request body:

```
secretKey: string;
```

Response body:

```
secretKey: string;
```

## Webhooks Endpoints

### Register Webhook

The endpoint below registers the webhook with the appropriate type in Canopy system. After that, Rent Passport updates will be sent to the specified callback url.

```
POST /referencing-requests/partner/:partnerId/webhook/register
```

Parameters:

```
partnerId: your partner reference
```

Request body:

```
type: string (required) - scpecifies which type of updates will be sent from Canopy, one of ["PASSPORT_STATUS_UPDATES", "REQUEST_STATUS_UPDATES", "PASSPORT_ALL_NOTES", "PASSPORT_AGENT_NOTES", "PASSPORT_REPORT_NOTES"];
callbackUrl: string (required) - the URL of the webhook endpoint;
additionalSettings: string[] (optional) - list of Rent Passport sections for which updates will be sent;
  - This field should only be added in case "PASSPORT_STATUS_UPDATES" webhook type is specified.
  - The following sections can be specified inside the array: ["INCOME", "RENT", "CREDIT_CHECK", "SAVINGS", "RENTAL_PREFERENCES", "EMPLOYEE_REFERENCE", "LANDLORD_REFERENCE"]
  - There is no need to explicitly specify the names of all sections if you want to receive all updates. Just send an empty array - [].
```

Successful response body:

```
"success": true,
"webhookType": string,
"callbackUrl": string
```

If you subscribed to the `REQUEST_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` each time one of the following events trigger:

- `INVITED` - the user was invited to connect;

- `INVITE_RESENT` - resent invitation email for the user;

- `CONNECTED` - user accepts the connection;

- `CONNECTION_REJECTED` - user rejects the connection;

- `CONNECTION_STOPPED` - user stops the connection;

- `SENDING_COMPLETED_PASSPORT_FAILED` - sending the completed passport to partner failed;

- `PASSPORT_COMPLETED` - user complete his passport and the document was sent. If a guarantor was requested for this user, the status means that user and guarantor complete their passports and the document with both passports was sent;

- `INVALID_APPLICATION_DETAILS` - partner's request body with application details was invalid;

- `PASSPORT_COMPLETED_WAITING_FOR_GUARANTOR` - user complete his passport and the document was sent, but the guarantor requested for this user did not complete his passport;

- `GUARANTOR_REQUEST_CANCELLED` - the guarantor request for the specified user was cancelled;

Once `REQUEST_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

```
canopyReferenceId: uuid,
partnerReferenceId: string,
notes: string,
```

If you are subscribed to the `PASSPORT_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` when one of the following Rent Passport sections is updated (if updates for this section requested):

- `CREDIT CHECK`

- `INCOME`

- `RENT`

- `SAVINGS`

- `LANDLORD REFERENCE` - optional, only if FULL SCREENING requested;

- `EMPLOYER REFERENCE` - optional, only if FULL SCREENING requested;

Once `PASSPORT_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

```
canopyReferenceId: uuid,
partnerReferenceId: uuid,
updatedSection: {
  type: enum - one of [INCOME, RENT, CREDIT_CHECK, SAVINGS, EMPLOYEE_REFERENCE, LANDLORD_REFERENCE],
  newStatus: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  updatedAt: ISODateTime,
},
instant: {
  status: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
  sections: {
    income: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    rent: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    creditCheck: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    savings: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  };
};
full: { /* optional field, sent only if FULL SCREENING requested  */
  status: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
  sections: {
    employerReference: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    landlordReference: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  };
};
globalStatus: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
```

If you subscribed to the `PASSPORT_ALL_NOTES` | `PASSPORT_AGENT_NOTES` | `PASSPORT_REPORT_NOTES` type, the updates will be sent to the `callbackUrl` when one of the folowing notes updated(if webhook type require this note type and partner has acces to note section):

- `AGENT_NOTE`

- `PASSPORT_REPORT_NOTE`

Once `ADMIN_NOTE_CHANGED` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

```
canopyReferenceId: uuid,
partnerReferenceId: string,
noteType: enum - one of [AGENT, REPORT],
section: enum - one of [SUMMARY, INCOME, RENT, CREDIT_CHECK, RENTAL_PREFERENCES, EMPLOYEE_REFERENCE, LANDLORD_REFERENCE, SAVINGS,   RIGHT_TO_RENT],
action: enum - one of [CREATED, EDITED, DELETED]
note: string,
```

### Unregister Webhook

The endpoint below unregisters webhook and stops sending Rent Passport updates to the specified callback URL.

```
DELETE /referencing-requests/partner/:partnerId/webhook/:webhookType
```

Parameters:

```
partnerId: your partner reference,
webhookType: the type of the webhook you want to unregister
```

Successful response body:

```
"success": true,
"webhookType": string
```

Unsuccessful response body:

```
/* If you don't have webhook subscriptions of the specified type */

"success": false,
"requestId" — the identifier of the request,
"error": {
  "status": 404,
  "type" — error type,
  "message" — “No webhooks of the specified type found”,
}
```

## Errors

We use conventional HTTP response codes to indicate the success or failure of an API request, typically one of:

- 2xx - indicate success
- 4xx - indicate a partner error with information provided (missign required parameters etc..)
- 5xx - indicate an error with the Canopy platform

We suggest you wrap 4xx errors in your partner and provide a user friendly message to your end users as part of the integration.
