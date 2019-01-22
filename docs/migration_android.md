---
id: migration_android
sidebar_label: Android
title: Kin Blockchain Migration - Android
---

## Before We Begin
Moving to the new Kin Blockchain has been a top priority of Kin's for the past few months.
The Kin Blockchain opens a world of new possibilities for Kin developers, and we're very excited to onboard you.

This document will walk you through your migration process as a Kin developer. Please read it carefully.

### Terminology
- **Old Blockchain**: The old blockchain that you're currently using.
- **Kin Blockchain**: The new blockchain that you'll be migrating to.
- **Dev Platform SDK**: The SDK that you use

## Migration Overview - How Is It Going to Work?
Let's start by understanding what migrating an account actually means. Note that this process is similar for end-user accounts and digital service accounts (that's yours).
Migrating an account to the Kin Blockchain consists of the following three main steps:

1. **A new account is created on the Kin Blockchain -**
This new account has *the same* keypair (i.e., private and public addresses) as the account on the Old Blockchain.
The account is pre-created before the migration begins with a balance of zero.

2. **The Old Blockchain account is "deactivated" -**
Deactivating an account ("burning") means the account can no longer earn or spend Kins. 
However, the account's last known balance and transaction history are still avialable. 

3. **The Kin Blockchain account is funded -**
Once we've "burned" the old account, we can safely fund the new account with the last known account balance.
This step marks the end of the migration process.

#### The Migration Process
Now that we understand what migrating an account means, let's talk about what the migration process of your application looks like.

The new Dev Platform SDK supports the Kin Blockchain "silently", alongside the Old Blockchain.
This allows you to integrate the SDK and wait until the desired adoption rate of your app is reached before starting the migration process.

Once you reach your desired adoption rate, contact Kin and together we'll decide on a migration date.
On the migration date, we (Kin) will raise a flag on our servers indicating to your clients that the migration of your application has started.
From that moment on, your users will automatically migrate to the Kin Blockchain upon launching your application. 

>**Important note:** Once the migration start flag is raised, old application clients who have not integrated the new Dev Platform SDK will not be able to use Kin.

#### Migrating the Digital Service Accounts
While your users migrate automatically, your blockchain accounts will be migrated manually by Kin.
You need to create and send Kin a signed transaction for burning both DS Wallet and Shared Wallet. 
[TBD]
Before we migrate your and your users' accounts, you need to complete the following tasks:
1. Integrate the new Dev Platform SDK into your app. 
2. Publish your new, migration-ready app to the Play Store.
3. Notify Kin when your app reaches the desired adoption rate.
4. Decide together with Kin on a migration date (preferably, at low traffic hours).
5. Create and send Kin a signed transaction for burning both your wallet and the Shared Wallet.

> Note: DO NOT send this transaction by yourself to the old blockchain! We will send it when the migration starts.

#### Migrating Your End Users 
Once you release a new version of your application that implements the migration-enabled Dev Platform SDK, it will take care of migrating your users automatically when you first initate the SDK (continue reading for more details).

## Understanding the User Migration Flow
To support migration to the Kin Blockchain, additional step was added "under the hood" to the Dev Platform SDK instantiation flow.

In previous versions of the Dev Platform SDK, calling `Kin.start()` would immediately initiate the SDK. In the migration edition, this is no longer the case.
Now, when `Kin.start()` is called, the Dev Platform SDK will query our servers and check if the current application's migration process has been initiated. If true, the SDK will proceed to check if the current user has already migrated to the Kin Blockchain. If the current user has not migrated, the Dev Platform SDK will begin the user migration flow. Only once the user has successfully migrated to the Kin Blockchain will the SDK be initiated.

>**Important note:** `Kin.start()` is now working **asynchronously**. You cannot safely use the SDK before receiving a successful callback from the initiating process! Using the SDK before start is completed will result in an exception.

The chart below shows the new `Kin.start()` flow. 
**Continue reading this document to better understand the different steps shown in the chart.**

![](/kin-ecosystem-sdk-docs/img/migration_start.jpeg)

## Initiating the SDK
The migration-enabled Dev Platform SDK adds a new argument to the `Kin.start()` method - an optional `KinMigrationListener` interface.
This interface allows you to hook into migration-related events (continue reading for more information).

>**Important note:** As of [SDK version 0.9.0](https://github.com/kinecosystem/kin-devplatform-android/releases/tag/0.9.0), a new optional callback was added to `Kin.start()` that notifies you whether the Dev Platform SDK was successfully initiated or an error occurred. 
In Version 1.0, this callback is no longer optional and must be implemented.

## The `KinMigrationListener` Interface
 An interface that allows you to hook into migration-related events.
 The interface exposes the following methods:

#### `onStart()`
This method is called once the Dev Platform SDK has determined that the current user should be migrated to the Kin Blockchain. It is important to note that this method will not be called after the user has been successfully migrated. If the user has already been migrated when the SDK is initiated (for instance, if the user was migrated in the previous app session), this method will not be invoked.

Since the migration process can take several seconds, we highly recommend to use the `onStart()` method for communicating to your users that something is happening in the background and your application has not "frozen". Feel free to communicate to your users as you see fit; a message such as "Kin upgrade in progress, please be patient" should be sufficient.

#### `onFinish()`
This method will be invoked once the user has been migrated successfully. Now would be a good time to hide any popup or other UI elements you used to communicate with your users on `onStart()`. If the user has already been migrated when the SDK is initiated (for instance if the user was migrated in the previous app session), this method will not be invoked.

>**Important note:** This is not an indication that the SDK was initiated successfully. You still need to wait for the `KinCallback` to return a response of success/failure (see example below).

#### `onError()`
This method will be invoked if the user migration has failed with an exception.
We highly recommend using this method to log the exception in your logging/analytics tools of choice.

## Kin.start() Code Sample
The following code sample shows the usage of the new `Kin.start()` method with the following arguments:
- A `Context`
- A JWT token required to authorize the user
- The `Environment` you wish to use
- `KinCallback` interface that allows us to know if the SDK initiation succeeded or failed
- `KinMigrationListener` to hook into migration-related events

```java
Kin.start(getApplicationContext(), jwt, Environment.getPlayground(), new KinCallback<Void>() {
    @Override
    public void onResponse(Void response) {
        // The SDK is ready. Now, you can safely use Kin.
    }

    @Override
    public void onFailure(KinEcosystemException error) {
        // Something went wrong when initiating the SDK.
    }

}, new KinMigrationListener() {
    @Override
    public void onStart() {
        // The user migration has started.
    }

    @Override
    public void onFinish() {
        // The user migration was completed successfully.
    }

    @Override
    public void onError(Exception e) {
        // The user migration has failed.
    }
});
```

## Important Things to Know
Migrating your users to the Kin Blockchain may pose a new temporary challenge. During the migration phase, your user base will be split between migrated users and users who have not been migrated yet (i.e., not all the users will be connected to the same blockchain). As a result, you may encounter the following situations: 

#### Migration flag raised during an active user session
A user launched your app while the migration flag (on our servers) is set to pre-migration (i.e., the user will not go through the migration process). During the user's session, the migration flag was changed to *migration started*. 

**Outcome:** The user will not be able to earn/spend Kins until `Kin.start()` is called again.

If you have your own earn/spend/P2P functionality (outside of the provided Kin Marketplace) the above scenario will throw a new type of exception - `MigrationNeededException` that you will need to handle.

For earning opportunities inside the Kin Marketplace, we handle this exception for you in the form of an error dialog displayed to the user.

#### Migrated users attempt to transfer Kin to non-migrated users
User A has already been migrated and is attempting to transfer Kin to User B, who has not been migrated yet.

**Outcome:** User A's Kin transfer will succeed. User B will see the transfer in the payment history screen, but will not see their balance updated until migrating to the Kin Blockchain.

## Testing Your Implementation
During the implementation of the new Dev Platform SDK you can safely test how your application will react to the *migration instatiated* flag. On the `Playground` environment, you can use the following POST HTTP request to change your application flag:

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/<your application id> -H "Content-type: Application/json" -d '{"blockchain_version": "<desired blockchain vesion>"}'`

`<desired blockchain vesion>` can have a value of "2" or "3" as a string. 
- "2" - Set your application flag to pre-migration
- "3" - Set your application flag to migration started

For example, the following HTTP request sets the flag to pre-migration for application ID "ABC1":

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/ABC1 -H "Content-type: Application/json" -d '{"blockchain_version": "2"}'`