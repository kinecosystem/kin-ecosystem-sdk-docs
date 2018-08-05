---
id: getting-started
title: Getting Started
---

## Obtaining Authentication Credentials

### Whitelist

Receive your app key and app-id.

> **Note:** whitelist registration should be used only for a quick first-time integration or sanity test, see [Whitelist limitations](jwt#WhitelistNote).

### JWT

* Generate your JWT key pair.
* Provide Ecosystem with your JWT public keys.
* Receive your JWT issuer jss/app-id, that will uniquely identify your application.
  
## Integrate Mobile SDK

Integrate Mobile SDK into your app, see [Android](android#IntegrateSDK) and [iOS](ios#IntegrateSDK) instructions.

## Starting The SDK

### Android

Call ```Kin.start(…)```, passing the user’s unique ID and your chosen authentication credentials (either whitelist or JWT credentials).

**Whitelist mode:**

```java
whitelistData = new WhitelistData(<userID>, <appID>, <apiKey>);
try {
   Kin.start(getApplicationContext(), whitelistData,
             Environment.getPlayground());
} 
catch (ClientException | BlockchainException e) {
   // Handle exception…
}
```

userID - your application unique identifier for the user  
appID - your application unique identifier as provided by Kin.  
apiKey - your secret apiKey as provided by Kin.

**JWT mode:**

For starting the SDK your server will need to generate a [registration JWT token](jwt#RegisterPayload), The fastest way for building JWT tokens is to use the [JWT Service](jwt#JWTService).

(See [Building the JWT Token](jwt#BuildJWT) to learn how to build the JWT token.)

```java
try {
   Kin.start(getApplicationContext(), jwt, Environment.getPlayground());
} 
catch (ClientException | BlockchainException e) {
   // Handle exception…
} 
```
