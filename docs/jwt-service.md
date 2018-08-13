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

#### Clone jwt-service repo:
```bash
$ git clone https://github.com/kinecosystem/kin-devplatform-jwt-service.git
```

#### Generate your application JWT key pairs:

```bash
$ ./create_keys.sh
```

This script will generate 10 key pairs inside keys folder, for each key 2 files will be created for the private and the public key. Generating multiple key pairs is a best practice for security measures.

#### Point the service to the created key pairs:

Edit `config/default.json` file, under `private_keys` and `public_keys` objects, delete the existing keys and add the generated private and public keys file accordingly.
```json
"<keyId>": {
    "algorithm": "ES256",
    "file": "keys/<file name>.pem"
    },
```

### Enter your app-id
App-id is the unique application identifier as provided by Kin, it serves as the `iss` - issuer field for each JWT token, for identify the origin of each request.  
Edit `config/default.json` file, under `app_id` object, and change it to your app-id

### Add your custom offers

Edit `config/default.json` file, under `offers` object, add your custom offers.

```json
{
    "id": "offer1", //Unique offer id
    "title": "first offer", //Offer title
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
            "amount": 10,
            "description": "the 1st test offer",
            "wallet_address": "GDC6UAWZETQRXDFJHHZS2O7JK2OQ3F5MNOZLHJP4TD7J6MB5Q2NH5YGU"
        },
        {
            "id": "offer2",
            "title": "second offer",
            "amount": 20,
            "description": "the 2nd test offer",
            "wallet_address": "GDC6UAWZETQRXDFJHHZS2O7JK2OQ3F5MNOZLHJP4TD7J6MB5Q2NH5YGU"
        },
        {
            "id": "offer3",
            "title": "third offer",
            "amount": 30,
            "description": "the 3rd test offer",
            "wallet_address": "GDC6UAWZETQRXDFJHHZS2O7JK2OQ3F5MNOZLHJP4TD7J6MB5Q2NH5YGU"
        }
    ]
}
```

### earn <a name="Earn"/>
Returns a token which can be used to create an earn order:  
`GET SERVICE_URL/earn/token?offer_id=offer1&user_id=aUserID`

Result:
```json
{
    "jwt": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6InJzXzUxMl8wIn0.eyJpc3MiOiJ0ZXN0IiwiZXhwIjoxNTMyNjExNjI5NDg1LCJpYXQiOjE1MzI1OTAwMjk0OTIsInN1YiI6ImVhcm4iLCJvZmZlciI6eyJpZCI6Im9mZmVyNCIsImFtb3VudCI6MX0sInJlY2lwaWVudCI6eyJ1c2VyX2lkIjoieW9zIiwidGl0bGUiOiJmb3VydGggb2ZmZXIiLCJkZXNjcmlwdGlvbiI6InRoZSA0dGggdGVzdCBvZmZlciJ9fQ.GGRnkIOa2PmwDws1jmySHC9Amb12QsNKwxMeD__gGjLUi4UNKPqyVIn4T_jsuwfHQGuvTGBQzZW3AV_9O_sb6uDnwE5c2k0ClL5IOdzV48NvjFJg2kT8e_h_IHDoCEMbVu0hU_UiOryjhoKb3pp7zDsymRcwhXNS2Hhzb6BOFb0"
}
```

### spend <a name="Spend"/>
Returns a token which can be used to create a spend order:  
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

### sign
Returns a token for a posted arbitrary payload (and subject):  
`POST '{"subject":"a subject", "payload":{"key1":"value1"}}' SERVICE_URL/sign`

Result:
```json
{
    "jwt":"eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ.eyJ0eXAiOiJKV1QiLCJrZXkxIjoidmFsdWUxIiwiaWF0IjoxNTIzODczMjEzLCJleHAiOjE1MjU0MTg2ODY5MTQsInN1YiI6ImEgc3ViamVjdCJ9.KzkD8VSHNZmo0H6Mb5a83OEiaDKUugO3R7Z2JN4GJh7YepH_gz0-sZ0YlLffvYnohwhciysJ9wtcwJ8YwbO7sedObmdZbezEYaBBowaezGzIMJeZc9erfTWu7aYP_-je-DpyVbY1lLvoFF8AufF7xPmYQQweYqFGhIp-9AHtKds"
}
```

### validate
Validates an input token using the configured public key:  
`GET SERVICE_URL/validate?jwt=anInputJWT`

Result:
```json
{"is_valid":true}
```

## Configuration
The service can be configured by changing the `config/default.json` file.

The most important of which are the:

* `app_id`: the unique application identifier as provided by Kin, serves as the `iss` - issuer field for each JWT token.
* `port`: on which port this service will be bound to
* `offers`: the collection of offers which are returned in the `/offer` endpoint and which are signed when calling the `/spend?offer_id={ ID }` endpoint
* `private_keys`: the keys which will be used for signing the tokens
* `public_keys`: the public keys which will be used for validate tokens