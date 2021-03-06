---
id: jwt-service
sidebar_label: JWT Service
title: JWT Service
---

A helper tool for managing Kin SDK [JWT Tokens](jwt).  
Using JWT service, a developer can:

* Generate a JWT key-pairs.
* Generate a signed JWT token for SDK [requests](jwt#JWTRequests).  
* Validate Kin server response JWT - [PaymentConfirmation](jwt#PaymentConfirmation).

## Keys
The repo contains private/public RSA keys and private/public stellar keys.  
These keys are for testing porpuses only.

## Install

### Clone jwt-service repo:
```bash
$ git clone https://github.com/kinecosystem/kin-devplatform-jwt-service.git
```

### Generate your application JWT key pairs:

```bash
$ ./create_keys.sh
```

This script will generate 10 key pairs inside keys folder. generating multiple key pairs is a best practice for security measures.
for each key, two files will be created, for the public key:  
`es256_<key-id>.pem`  
An example:  
`es256_EEA3AFB1-4708-44B6-A5B4-5875A4B0423A.pem`  
And for the private key:  
`es256_<key-id>-priv.pem`  
An example:  
`es256_EEA3AFB1-4708-44B6-A5B4-5875A4B0423A-priv.pem`

<a name="KeyIdNote"></a>
> **NOTE:** The generated unique random `key-id` is an important piece of information. on each JWT request generation, JWT Service will choose randomly one of the keys to sign the request with. each JWT request will include the `key-id` it was signed with, and will be use by the other party for determines which public key to use for validating the signature.

### Point the service to the created private keys:

Edit `config/default.json` file, under `private_keys` object, delete the existing keys and add the generated private keys.  
The `key-id` is the unique random key id included in the filename, see above [note](#KeyIdNote).
```json
"private_keys": {
    "es256_<key-id>": {
        "algorithm": "ES256",
        "file": "keys/<file name>.pem"
        },
        ...
}
```

For the above generated key:  
```json
"private_keys": {
    "es256_EEA3AFB1-4708-44B6-A5B4-5875A4B0423A": {
        "algorithm": "ES256",
        "file": "keys/es256_EEA3AFB1-4708-44B6-A5B4-5875A4B0423A-priv.pem"
        },
        ...
}
```

### Point the service to Kin public keys:

To enable JWT Service to verify [JWT order confirmations](jwt#PaymentConfirmation) received from Kin server, config file should point to [Kin public keys](jwt#PublicKeys). For playground environment, no need for manual configuration, the default config in JWT Service already set to the playground public keys.

### Enter your app-id
App-id is the unique application identifier as provided by Kin, it serves as the `iss` - issuer field for each JWT token, for identify the origin of each request.  
Edit `config/default.json` file, under `app_id` object, and change it to your app-id

<a name="Predefine"></a>
### Add your pre-defined custom offers (Optional)

Edit `config/default.json` file, under `offers` object, add your custom offers.

> Note: Offer id should be unique when sending multiple offers for a single user. pre-defined offers use the same offer id and can be used only once per user. For creating multiple offers per user or define offers parameters dynamically, see [sign](#Sign) request.

```json
{
    "id": "offer1", //Unique offer id
    "title": "first offer", //Offer title
    "type" : "spend", //Offer type
    "description": "the 1st test offer", //Offer description
    "type": "spend", //Offer type: "spend"/"earn"/"pay_to_user"
    "amount": 1, //Kin amount
    "wallet_address": "GDC6UAWZETQRXDFJHHZS2O7JK2OQ3F5MNOZLHJP4TD7J6MB5Q2NH5YGU"
}
```
#### Install the repo:

```bash
$ npm i
```

#### Build from source:

```bash
$ npm run build
```

#### Run the service
```bash
$ npm run start
```

## Using the service
The service supports 4 endpoints:

### offers
Returns a collection of the configured offers:  
`GET SERVICE_URL/offers`

Result:
```json
{
    "offers": [
        {
            "id": "offer1",
            "title": "first offer",
            "type" : "earn",
            "amount": 10,
            "description": "the 1st test offer"
        },
        {
            "id": "offer2",
            "title": "second offer",
            "type" : "spend",
            "amount": 20,
            "description": "the 2nd test offer"
        },
        {
            "id": "offer3",
            "title": "third offer",
            "type" : "pay_to_user",
            "amount": 30,
            "description": "the 3rd test offer"
        }
    ]
}
```

<a name="Sign"></a>
### sign
Returns a token for a posted arbitrary payload (and subject), can be used to create a repetitive and/or dynamic offers with arbitrary values, as opposed to the [pre-defined offers](#Predefine). see [Signing Custom Offers](#CustomOffers) for how to construct a custom offer.


`POST '{"subject":"a subject", "payload":{"key1":"value1"}}' SERVICE_URL/sign`

- `"subject"` - JWT `sub` field, see [JWT Requests](jwt#JWTRequests) for the available requests subjects.
- `"payload"` - Kin server fields for JWT requests, see [JWT Requests](jwt#JWTRequests) for the available parameters for each request.

<a name="CustomOffers"></a>
#### Signing Custom Offers

Creating a token for dynamic offer is done by specifying offers requests data as follows:

> Pay attention to creating a unique offer id for each offer, Kin server will block a repetitive offer with the same values for the same user. Offer id serves for distinguishing each offer from the previous one and preventing the client from sending a single generated JWT multiple times.

* "subject" - one of the available offer types: `"spend"`/`"earn"`/`"pay_to_user"`
* "payload" - the offer request body.

`POST '<offer json>' SERVICE_URL/sign`

**Spend offer json**

```json
{
    "subject": "spend",
    "payload": {
        "offer": {
            "id": "offer unique id - generated by application server",
            "amount": "amount of kin for this offer"
        },
         "sender": {
            "user_id": "the user_id who will perform the spend",
            "title": "order title - appears in sender order history",
            "description": "order description - appears in sender order history"
        }
    }
}
```
**Earn offer json**

```json
{
    "subject": "earn",
    "payload": {
        "offer": {
            "id": "offer unique id - generated by application server",
            "amount": "amount of kin for this offer"
        },
        "recipient": {
            "user_id": "user_id who will receive the order",
            "title": "offer title - appears in recipient order history",
            "description": "offer description - appears in recipient order history"
        }
    }
}
```

**Pay to user offer json**

```json
{
    "subject": "pay_to_user",
    "payload": {
        "offer": {
            "id": "offer unique id - generated by application server",
            "amount": "amount of kin for this offer"
        },
        "sender": {
            "user_id": "the user_id who will perform the spend",
            "title": "order title - appears in sender order history",
            "description": "order description - appears in sender order history"
        },
        "recipient": {
            "user_id": "user_id who will receive the order",
            "title": "offer title - appears in recipient order history",
            "description": "offer description - appears in recipient order history"
        }
    }
}
```

Result:
```json
{
    "jwt":"eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ.eyJ0eXAiOiJKV1QiLCJrZXkxIjoidmFsdWUxIiwiaWF0IjoxNTIzODczMjEzLCJleHAiOjE1MjU0MTg2ODY5MTQsInN1YiI6ImEgc3ViamVjdCJ9.KzkD8VSHNZmo0H6Mb5a83OEiaDKUugO3R7Z2JN4GJh7YepH_gz0-sZ0YlLffvYnohwhciysJ9wtcwJ8YwbO7sedObmdZbezEYaBBowaezGzIMJeZc9erfTWu7aYP_-je-DpyVbY1lLvoFF8AufF7xPmYQQweYqFGhIp-9AHtKds"
}
```

### validate
Validates an input token using the configured public key, can be use for veriyfing [Payment Confirmation JWT](jwt#PaymentConfirmation) received from Kin Server.
`GET SERVICE_URL/validate?jwt=anInputJWT`

Result:
```json
{"is_valid":true}
```

### earn <a name="Earn"/>
Returns a token which can be used to create an earn order:

> Note: The token will be generated from the [pre-defined](#Predefine) offers, and can be use only once per user. For creating offer dynamically see [sign](#Sign) request

`GET SERVICE_URL/earn/token?offer_id=offer1&user_id=aUserID`

Result:
```json
{
    "jwt": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6InJzXzUxMl8wIn0.eyJpc3MiOiJ0ZXN0IiwiZXhwIjoxNTMyNjExNjI5NDg1LCJpYXQiOjE1MzI1OTAwMjk0OTIsInN1YiI6ImVhcm4iLCJvZmZlciI6eyJpZCI6Im9mZmVyNCIsImFtb3VudCI6MX0sInJlY2lwaWVudCI6eyJ1c2VyX2lkIjoieW9zIiwidGl0bGUiOiJmb3VydGggb2ZmZXIiLCJkZXNjcmlwdGlvbiI6InRoZSA0dGggdGVzdCBvZmZlciJ9fQ.GGRnkIOa2PmwDws1jmySHC9Amb12QsNKwxMeD__gGjLUi4UNKPqyVIn4T_jsuwfHQGuvTGBQzZW3AV_9O_sb6uDnwE5c2k0ClL5IOdzV48NvjFJg2kT8e_h_IHDoCEMbVu0hU_UiOryjhoKb3pp7zDsymRcwhXNS2Hhzb6BOFb0"
}
```

### spend <a name="Spend"/>
Returns a token which can be used to create a spend order:  

> Note: The token will be generated from the [pre-defined](#Predefine) offers, and can be use only once per user. For creating offer dynamically see [sign](#Sign) request

`GET SERVICE_URL/spend/token?offer_id=offer1`

Result:
```json
{
    "jwt": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ.eyJ0eXAiOiJKV1QiLCJpZCI6Im9mZmVyMSIsInRpdGxlIjoidGhpcmQgb2ZmZXIiLCJhbW91bnQiOjMwLCJkZXNjcmlwdGlvbiI6InRoZSAzcmQgdGVzdCBvZmZlciIsIndhbGxldF9hZGRyZXNzIjoiR0RDNlVBV1pFVFFSWERGSkhIWlMyTzdKSzJPUTNGNU1OT1pMSEpQNFREN0o2TUI1UTJOSDVZR1UiLCJpYXQiOjE1MjM4NzI4MDYsImV4cCI6MTUyNTQxODI3OTE3MCwic3ViIjoic3BlbmQifQ.d2pEsXzWMr-XXNfnKYL52C-GscRMdIqtrETdpGc2R_TOnLcScXMLFU62HshP3hxZW88vi5JY42MszVApNmCQ_XI9XgVcZcAIYx6Ef63sO-e1WG8_oPRFFLwHf1p8VylArtkvaz2JkWbHVPQuCNdcwf31JUMVSqJZHGk6ez3KaSQ"
}
```

The `offer_id` needs to match one of the offers which returns from the `SERVICE_URL/offers` endpoint.

### pay to user <a name="PayToUser"/>
Returns a token which can be used to create a pay to user order:  

> Note: The token will be generated from the [pre-defined](#Predefine) offers, and can be use only once per user. For creating offer dynamically see [sign](#Sign) request

`GET SERVICE_URL/paytouser/token?offer_id=offer1&sender_id=user_id1&recipient_id=user_id2`

Result:
```json
{
    "jwt": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ.eyJ0eXAiOiJKV1QiLCJpZCI6Im9mZmVyMSIsInRpdGxlIjoidGhpcmQgb2ZmZXIiLCJhbW91bnQiOjMwLCJkZXNjcmlwdGlvbiI6InRoZSAzcmQgdGVzdCBvZmZlciIsIndhbGxldF9hZGRyZXNzIjoiR0RDNlVBV1pFVFFSWERGSkhIWlMyTzdKSzJPUTNGNU1OT1pMSEpQNFREN0o2TUI1UTJOSDVZR1UiLCJpYXQiOjE1MjM4NzI4MDYsImV4cCI6MTUyNTQxODI3OTE3MCwic3ViIjoic3BlbmQifQ.d2pEsXzWMr-XXNfnKYL52C-GscRMdIqtrETdpGc2R_TOnLcScXMLFU62HshP3hxZW88vi5JY42MszVApNmCQ_XI9XgVcZcAIYx6Ef63sO-e1WG8_oPRFFLwHf1p8VylArtkvaz2JkWbHVPQuCNdcwf31JUMVSqJZHGk6ez3KaSQ"
}
```

The `offer_id` needs to match one of the offers which returns from the `SERVICE_URL/offers` endpoint.

### register <a name="Register"/>
Returns a token which can be used to register a user:  
`GET SERVICE_URL/register/token?user_id=aUserID`

Result:
```json
{
    "jwt": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ.eyJ0eXAiOiJKV1QiLCJ1c2VyX2lkIjoiYVVzZXJJRCIsImlhdCI6MTUyMzg3Mjk5NSwiZXhwIjoxNTI1NDE4NDY4Mzc2LCJzdWIiOiJyZWdpc3RlciJ9.RenNzwLk7DQCi-U3vXZO4d-4pM4nA1X-RW3JHGWq3BNRLOrzePQZ7O6VgW7wV2-YBPjR9UEAU9XrKs8FT7DYDzgfQA5iFOpEHRlDoiILmwPrje601BE5LGvoNPR4HI4bIPgDiw2-XuIqqNXRVCx4oWqR_p_ex8GEA95ty0aoP4Q"
}
```

## Configuration
The service can be configured by changing the `config/default.json` file.

The most important of which are the:

* `app_id`: the unique application identifier as provided by Kin, serves as the `iss` - issuer field for each JWT token.
* `port`: on which port this service will be bound to
* `offers`: the collection of offers which are returned in the `/offer` endpoint and which are signed when calling the `/spend?offer_id={ ID }` endpoint
* `private_keys`: application private keys which will be used for signing the tokens
* `public_keys`: kin server public keys which will be used for validate tokens (order confirmation requests)