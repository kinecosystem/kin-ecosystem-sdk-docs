---
id: jwt
title: Authentication and JWT
---

## Whitelist vs JWT <a name="WhitelistNote"></a>

### Whitelist authentication

To be used for a quick first-time integration or sanity test. The authentication credentials are provided as simple appID and apiKey values. (For development and testing, you can use the default values provided in the Sample App.)

> **NOTE:**
> * You can only use whitelist authentication for the Playground environment. The Production environment requires that you use JWT authentication.
> * Whitelist authentication has a limitation on users number.
> * Creating your app custom offers requires JWT authentication.

### JWT authentication

A secure authentication method to be used in production.  
JWT is used for authorization with Kin servers and for transferring offers information between application and Kin servers in a trusted way.  
By digitally signing a request to and a response from the Kin server– each party may verify the other parties’ identity and request content authenticity without having to trust entirely the client.

The integrating application should provide the Kin team with one or more public signature keys and its corresponding keyID. The application will receive a JWT issuer identifier (ISS key) also called an app-id which uniquely identifies your app.

Go to [jwt.io](https://jwt.io) to learn more about the JWT standard.

> **NOTE:** It's important to generate the JWT Tokens and store the JWT private keys at your server side only.  
> The purpose of JWT is to establish a trusted communication between Kin server and your application server without relaying solely on the client.
> The client should not be allowed to request earn/spend without your server authorization, thus, the client should not keep the private keys and signs JWT tokens. The sample app does that only for the sake of simplification and serves as an example of client side APIs.

## Building the JWT Token <a name="BuildJWT"></a>

A JWT token is a string that is composed of 3 parts:

* **Header** – a JSON structure encoded in Base64Url
* **Payload** – a JSON structure encoded in Base64Url
* **Signature** – constructed with this formula: 

    ```ES256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)```  
    -- where the secret value is the private key of your agreed-on public/private key pair.

The 3 parts are then concatenated with the character “.” between each of the consecutive parts as pictured below:


```<header> + “.” + <payload> + “.” + <signature>```

See [jwt.io](https://jwt.io) to learn more about how to build a JWT token and to find libraries that you can use to do this.

This is the header structure:

```js
{
    "alg": "ES256",
    "typ": "JWT",
    "kid": "string" // ID of the keypair that was used to sign the JWT. 
    // IDs and public keys will be provided by the signing authority. 
    // This enables using multiple private/public key pairs. 
    // (The signing authority must provide the verifier with a list of public 
    // keys and their IDs in advance.)
}
```

This is the payload structure:

```js
{
    // standard fields
    iat: number;  // the time this token was issued, in seconds from Epoch
    iss: string;  // issuer (Kin will provide this value)
    exp: number;  // the time until this token expires, in seconds from Epoch 
    sub: "register"

    // application fields
    user_id: string; // A unique ID of the end user (must only be unique among your app’s users; not globally unique)
}
```

## JWT Service - A Helper Tool For Handling JWT Tokens <a name="JWTService"></a>

JWT Service is a helper tool for generating signed Kin SDK JWT tokens. It can be deployed and used as an application server-side. 

Using JWT service a developer can:

* Generate a JWT key-pair.
* Generate a signed JWT token for SDK [requests](#JWTRequests).  
* Validate Kin server response JWT - [PaymentConfirmation](#PaymentConfirmation).

See [JWT Service](jwt-service) for more details.


## JWT Requests <a name="JWTRequests"/>

### Register payload <a name="RegisterPayload"/>

Registration request, authorize a user with Kin SDK, required for launching the SDK.

```js
{
    // common fields
    iat: number; // issued at - seconds from epoch
    iss: string; // issuer - request origin 'app-id' provided by KIn
    exp: number; // expiration
    sub: string; // subject - "register"

    // application fields
    user_id: string; // id of the user - or a deterministic unique id for the user
}
```

##### [Example (viewable on jwt.io)](https://jwt.io/?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InJlZ2lzdGVyIiwidXNlcl9pZCI6ImFfaGFzaF9vbl9raWtfdXNlcl9pZCJ9.6DwojspQ46inlwYwn3NNH0bmQIzKd7mL0VX8V2ZmlH8aZqWF2UbK_Md5kcxgnXgq0P6tExVTr1vxpwhfj7_3dg)
```
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InJlZ2lzdGVyIiwidXNlcl9pZCI6ImFfaGFzaF9vbl9raWtfdXNlcl9pZCJ9.6DwojspQ46inlwYwn3NNH0bmQIzKd7mL0VX8V2ZmlH8aZqWF2UbK_Md5kcxgnXgq0P6tExVTr1vxpwhfj7_3dg
```

### Spend payload <a name="SpendPayload"/>

Custom spend request.

```js
{
    // common fields
    iat: number; // issued at - seconds from epoch
    iss: string; // issuer - request origin 'app-id' provided by Kin
    exp: number; // expiration
    sub: string; // subject - "spend"

    offer: {
    id: string; // offer unique id - generated by application server
        amount: number; // amount of kin for this offer
    },
    sender: {
        user_id: string; // optional: the user_id who will perform the spend
        title: string; // order title - appears in order history
        description: string; // order description - appears in order history
    }
}
```

##### [Example (viewable on jwt.io)](https://jwt.io/?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InNwZW5kIiwib2ZmZXIiOnsiaWQiOiJPMTIzMTIzMTIzIiwiYW1vdW50Ijo1MDAwfSwic2VuZGVyIjp7InRpdGxlIjoiQmx1ZSBUaGVtZSIsImRlc2NyaXB0aW9uIjoiT2NlYW4gQmx1ZSIsInVzZXJfaWQiOiJzb21lX3VzZXIifX0.BRacJ37zbbaKivWD_uhC2JXozzrHDN5B9VeG7d1BXjkpPYEfpaEy_kKF6xqp7oUMjEG2ltrOQPJ-UkFcFl6H-g)
```
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InNwZW5kIiwib2ZmZXIiOnsiaWQiOiJPMTIzMTIzMTIzIiwiYW1vdW50Ijo1MDAwfSwic2VuZGVyIjp7InRpdGxlIjoiQmx1ZSBUaGVtZSIsImRlc2NyaXB0aW9uIjoiT2NlYW4gQmx1ZSIsInVzZXJfaWQiOiJzb21lX3VzZXIifX0.BRacJ37zbbaKivWD_uhC2JXozzrHDN5B9VeG7d1BXjkpPYEfpaEy_kKF6xqp7oUMjEG2ltrOQPJ-UkFcFl6H-g
```

### Earn payload <a name="EarnPayload"/>

Custom earn request. 
```js
{
    // common fields
    iat: number; // issued at - seconds from epoch
    iss: string; // issuer - request origin 'app-id' provided by Kin
    exp: number; // expiration
    sub: string; // subject - "earn"

    offer: {
        id: string; // offer unique id - generated by application server
        amount: number; // amount of kin for this offer
    },
    recipient: {
        user_id: string; // the user_id who will perform the order
        title: string; // order title - appears in order history
        description: string; // order description - appears in order history
    }
}
```

##### [Example (viewable on jwt.io)](https://jwt.io/?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6ImVhcm4iLCJvZmZlciI6eyJpZCI6Ik8xMjMxMjMxMjMiLCJhbW91bnQiOjUwMDB9LCJyZWNpcGllbnQiOnsidGl0bGUiOiJCbHVlIFRoZW1lIiwiZGVzY3JpcHRpb24iOiJPY2VhbiBCbHVlIiwidXNlcl9pZCI6InNvbWVfdXNlciJ9fQ.R7OpvaZQIAzjQ0MSi5nC1c39oC9oN08NVKwricMyWnuMbK5FD9Qn6ecmol4JnMGE5IZA7j_LR-EEbVhhEYi57g)
```

eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6ImVhcm4iLCJvZmZlciI6eyJpZCI6Ik8xMjMxMjMxMjMiLCJhbW91bnQiOjUwMDB9LCJyZWNpcGllbnQiOnsidGl0bGUiOiJCbHVlIFRoZW1lIiwiZGVzY3JpcHRpb24iOiJPY2VhbiBCbHVlIiwidXNlcl9pZCI6InNvbWVfdXNlciJ9fQ.R7OpvaZQIAzjQ0MSi5nC1c39oC9oN08NVKwricMyWnuMbK5FD9Qn6ecmol4JnMGE5IZA7j_LR-EEbVhhEYi57g
```

#### PayToUser payload <a name="PayToUserPayload"/>
```js
{
    // common fields
    iat: number; // issued at - seconds from epoch
    iss: string; // issuer - request origin 'app-id' provided by Kin
    exp: number; // expiration
    sub: string; // subject - "pay_to_user"

    offer: {
        id: string; // offer unique id - generated by application server
        amount: number; // amount of kin for this offer - price
    },
    sender: {
        user_id: string; // optional: user_id who will perform the order
        title: string; // offer title - appears in order history
        description: string; // offer description - appears in order history
    },
    recipient: {
        user_id: string; // user_id who will receive the order
        title: string; // offer title - appears in order history
        description: string; // offer description - appears in order history
    }
}
```

##### [Example (viewable on jwt.io)](https://jwt.io/?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InBheV90b191c2VyIiwib2ZmZXIiOnsiaWQiOiJPMTIzMTIzMTIzIiwiYW1vdW50Ijo1MDAwfSwic2VuZGVyIjp7InRpdGxlIjoiVGlwIEZyb20gRG9vZHkiLCJkZXNjcmlwdGlvbiI6IkRvb2R5IHRpcHBlZCB5b3UiLCJ1c2VyX2lkIjoidXNlcjpuaXR6YW4ifSwicmVjaXBpZW50Ijp7InRpdGxlIjoiVGlwIFRvIE5pdHphbiIsImRlc2NyaXB0aW9uIjoiWW91IHRpcHBlZCBOaXR6YW4iLCJ1c2VyX2lkIjoidXNlcjpkb29keSJ9fQ.GVufyAI2UkpzzLlSL7a_kb5JcdSkgBb1PnzdL7wUkIGx-IUUu_pgTecElwTbZvWrCSr_GkJ4leD1MnXHSUb_QQ)

```
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayBzZXJ2ZXIiLCJleHAiOjE1MjYyMzkwMjIsInN1YiI6InBheV90b191c2VyIiwib2ZmZXIiOnsiaWQiOiJPMTIzMTIzMTIzIiwiYW1vdW50Ijo1MDAwfSwic2VuZGVyIjp7InRpdGxlIjoiVGlwIEZyb20gRG9vZHkiLCJkZXNjcmlwdGlvbiI6IkRvb2R5IHRpcHBlZCB5b3UiLCJ1c2VyX2lkIjoidXNlcjpuaXR6YW4ifSwicmVjaXBpZW50Ijp7InRpdGxlIjoiVGlwIFRvIE5pdHphbiIsImRlc2NyaXB0aW9uIjoiWW91IHRpcHBlZCBOaXR6YW4iLCJ1c2VyX2lkIjoidXNlcjpkb29keSJ9fQ.GVufyAI2UkpzzLlSL7a_kb5JcdSkgBb1PnzdL7wUkIGx-IUUu_pgTecElwTbZvWrCSr_GkJ4leD1MnXHSUb_QQ
```

### PaymentConfirmation payload <a name="PaymentConfirmation"></a>

A confirmation payload received by Kin server in case of completing successfully a custom offer flow.

```js
{
    // common fields
    iat: number; // issued at - seconds from epoch
    iss: string; // issuer - request origin 'app-id' provided by Kin
    exp: number; // expiration
    sub: string; // subject - "payment_confirmation"

    sender_user_id: string; // sender user identifier - same value as given by register
    recipient_user_id: string; // recipient user identifier - same value as given by register
    offer_id: string; // offer unique id - generated by application server
    payment: {
        blockchain: string; // identifier of the blockchain network the transaction was made on
        transaction_id: string; // stellar identifier of the blockchain transaction
    }
}
```

##### [Example (viewable on jwt.io)](https://jwt.io/?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayIsImV4cCI6MTUyNjIzOTAyMiwic3ViIjoicGF5bWVudF9jb25maXJtYXRpb24iLCJvZmZlcl9pZCI6Ik8xMjMxMjMxMjMiLCJzZW5kZXJfdXNlcl9pZCI6InVzZXI6ZG9vZHkiLCJyZWNpcGllbnRfdXNlcl9pZCI6InVzZXI6bml0emFuIiwicGF5bWVudCI6eyJibG9ja2NoYWluIjoic3RlbGxhci1tYWlubmV0IiwidHJhbnNhY3Rpb25faWQiOiJ0cmFuc2FjdGlvbjoxMjM0NSJ9fQ.-AbZOfC69eY1It43RccOXluY-sjWSi4JFvQkVKO9D2UgYU3jNPbEcBERLrqBHPSpS6f26LVpIsg5A81UQNoukw)
```
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InNvbWVfaWQifQ.eyJpYXQiOjE1MTYyMzkwMjIsImlzcyI6ImtpayIsImV4cCI6MTUyNjIzOTAyMiwic3ViIjoicGF5bWVudF9jb25maXJtYXRpb24iLCJvZmZlcl9pZCI6Ik8xMjMxMjMxMjMiLCJzZW5kZXJfdXNlcl9pZCI6InVzZXI6ZG9vZHkiLCJyZWNpcGllbnRfdXNlcl9pZCI6InVzZXI6bml0emFuIiwicGF5bWVudCI6eyJibG9ja2NoYWluIjoic3RlbGxhci1tYWlubmV0IiwidHJhbnNhY3Rpb25faWQiOiJ0cmFuc2FjdGlvbjoxMjM0NSJ9fQ.-AbZOfC69eY1It43RccOXluY-sjWSi4JFvQkVKO9D2UgYU3jNPbEcBERLrqBHPSpS6f26LVpIsg5A81UQNoukw
```

## Kin JWT Public keys <a name="PublicKeys"></a>

Public keys should be used by the Application server to verify Kin server responses. Kin server provides a list of a possible public keys and their corresponding key id. The key id, will appear as `kid` field in JWT header (see [Building the JWT Token](#BuildJWT)) and it will be used by the Application server to match the public key to the JWT response.  
The public keys can be accessed on Kin server under `config` request:  

* [Playground - https://api.developers.kinecosystem.com/v1/config](https://api.developers.kinecosystem.com/v1/config)

Config request will return a json that contain the jwt keys inside `jwt_keys` object:

##### Example
```js
{  
   "jwt_keys":{  
      "es256_25736782-e7a3-4964-bb6c-dd5b2be9774f":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE5qZcnsSc8mzt3okkDcaYwrH5WrqGa4nm\nsL2HvggEajeLKMHQsnC1Ffh/aTc0BAJUdNjuDiDwhWEPLAlDWT5QQw==\n-----END PUBLIC KEY-----\n"
      },
      "es256_4b2cc4be-4227-4cad-a543-2b2cb1154a85":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE5VrjZ3t1LuZpbb+eP18yYc0DliSXMlLJ\nUM19r3r6J8cXS4qyy//qZEXZhMXT2+l0OK5eSH0iJb7jaSLDKikDcg==\n-----END PUBLIC KEY-----\n"
      },
      "es256_58d0a66d-3295-4eb0-a941-461e04ffd03c":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE1QDcI/dMeVvJMbjqOSL+Hob2y7+5feoH\n0R85S37hmm6nfJYA/BsxFOAzcYy81soIrEMqhJ0aIFm4rkT1ZyshvQ==\n-----END PUBLIC KEY-----\n"
      },
      "es256_607aeb50-5dfa-465a-a329-779f0576c59b":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAErQLvT6MybuJsqRP1Q6UT5d0uTkYrGE4a\n1LsVl6PqtUgfIHe/UN9ZiHsyJwFpiqSeczKxFmYJ7q4g7xAK+lGEnQ==\n-----END PUBLIC KEY-----\n"
      },
      "es256_86aad391-42f1-44b5-808e-12b7bdd37458":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEfNRr7lPPbah4rhr1/44cHUpYe9UqNGbR\nvvSK7HTJmtDHjXa2rnPpor5jogslos33yPHr0ObeZQGF+wPdSMbb2w==\n-----END PUBLIC KEY-----\n"
      },
      "es256_a70882db-99cf-4f82-b1b1-1197d993d0d3":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEdA9wK35fBym0dPVicj7WJW2m3HrUu9Ua\neWa0+RyYYk0MQVtTblr2dNl5B4QT6QcLmGPc/eqIQ6028uquTnhrHA==\n-----END PUBLIC KEY-----\n"
      },
      "es256_c24d9feb-3afd-4fe1-8b06-0c20674ab904":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEbyBVnaSwQGwfpRI43DTQDQg9hcp8j/Vc\n69rrH/WOr397oe2dDzk4WcfmlgS7SUI0DUt+qf3AECxfStYrmMaiaw==\n-----END PUBLIC KEY-----\n"
      },
      "es256_e78a9ea4-1e95-477c-8ab8-9e5c7c2561d2":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAELS3yPKy+/KC2kN0KgFQ9EE3YNiyviKHy\nwhlKTPuNDQatJDCstYpGXidlGzSgCXB6Vzpm5/0QVUYGCHgYHeSWBw==\n-----END PUBLIC KEY-----\n"
      },
      "es256_f54ac750-efd7-40b5-9cea-1a0d7ad675cc":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEiySHgDD4CKx9drgWZ1FzF/+ICFBk0U5X\nAIr6v1Dzxir8bNm6yZLMb17A7iiRXejvOJTCHkw42SS6MDSJ0ZZKyA==\n-----END PUBLIC KEY-----\n"
      },
      "es256_fa844718-945a-470e-ae25-c96dbee57da6":{  
         "algorithm":"ES256",
         "key":"-----BEGIN PUBLIC KEY-----\nMFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE7AulD8kXdYPJlYnlXjtqFeZ67GQvjM1c\nTR0EgGbZtysWbdUIWoPVMaZEf7zRrr3xU2JybHg05s++Ikq0TRntJw==\n-----END PUBLIC KEY-----\n"
      }
   },
   "blockchain":{  
      "asset_code":"KIN",
      "asset_issuer":"GBC3SG6NGTSZ2OMH3FFGB7UVRQWILW367U4GSOOF4TFSZONV42UJXUH7",
      "horizon_url":"https://horizon-playground.kininfrastructure.com",
      "network_passphrase":"Kin Playground Network ; June 2018"
   },
   "bi_service":"https://kin-bi.appspot.com/eco_",
   "webview":"https://s3.amazonaws.com/assets.kinplayground.com/web-offers/cards-based/index.html",
   "environment_name":"playground",
   "ecosystem_service":"https://api.kinplayground.com"
}
```
