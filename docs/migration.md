---
id: migration
sidebar_label: Kin Blockchain Migration
title: Kin Blockchain Migration
---

## Before we begin
Moving to the new Kin Blockchain has been a top priority of Kin's for the past few months.
The Kin Blockchain opens a world of new possibilities for Kin developers, and we're very excited to onboard you.

This document will walk you through your migration process as a Kin developer. Please read it carefully.

### Terminology
- **Old Blockchain**: The old blockchain that you're currently using.
- **Kin Blockchain**: The new blockchain that you'll be migrating to.
- **Dev Platform SDK**: The SDK which you use.

## Migration overview - How's it going to work?
Let's start by understanding what migrating an account actually means.
Migrating an account to the Kin Blockchain consists of three main steps. Please note that this process is similar between end-user accounts, 
and digital service accounts (that's you).

1. **A new account is created on the Kin Blockchain -**
This new account has *the same* keypair (i.e. private and public addresses) as the account on the Old Blockchain.
The account is pre-created before the migration begins with a balance of zero.

2. **The Old Blockchain account is "deactivated" -**
Deactivating an account ("burning") means the account can no longer earn or spend Kin's. 
However, the accounts last known balance and transaction history are still avialable. 

3. **The Kin Blockchain account is funded -**
Once we've "burned" the old account, we can safely fund the new account with the last known account balance.
This step marks the end of the migration process.

#### The migration process
Now that we understand what migrating an account means, let's talk about how the migration process of your application looks like.

The new Dev Platform SDK supports the Kin Blockchain "silently", alongside the Old Blockchain.
This allows you to integrate the SDK and wait until the desired adoption rate of your app is reached before starting the migration process.

Once you reach your desired adoption rate, contact Kin and together we'll decide on a migration date.
On the migration date, we (Kin) will raise a flag on our servers indicating to your clients that the migration has started.
From that moment on, your users will automatically migrate to the Kin Blockchain upon launching your application. 

>**Important note:** Once the migration start flag is raised, old clients who do not integrate the new Dev Platform SDK will not be able to use Kin.


#### Migrating your end users
Once you release a new version of your application that implements the migration-enabled Dev Platform SDK, it will take care of migrating your users automatically 
when you first initate the SDK (continue reading for more details).

#### Migrating the digital service accounts
While your users migrate automatically, your blockchain accounts will be migrated manually by Kin.
You need to create and send Kin a signed transaction for burning both DS Wallet and Shared Wallet. 
[TBD]
Before we migrate your accounts and your users you need to complete the following tasks:
- Integrate the new Dev Platform SDK into your app.
- Publish your new, migration ready app to the Play Store.
- Notify Kin when your app reaches the desired adoption rate.
- Decide togther with Kin on a migration date (preferably at low traffic hours).
- Create and send Kin a signed transaction for burning both your wallet and the Shared Wallet. 
> Note: DO NOT send this transaction by yourself to the old blockchain! We will send it when the migration will start.

## Understanding the user migration flow
To support migrating to the Kin Blockchain, additional functionality was added "under the hood" to the Dev Platform SDK instantiation flow.

In previous versions of the Dev Platform SDK, calling `Kin.start()` was immediately initiating the SDK. In the migration edition, this is no longer the case.
Now, when calling `Kin.start()` the Dev Platform SDK will query our servers and check if the current application's migration process was initiated. If true, the SDK will proceed to check if the current user was already migrated to the Kin Blockchain. If the current user wasn't migrated, the Dev Platform SDK will begin the user migration flow. Only once the user was successfully migrated to the Kin Blockchain will the SDK be initiated.

>**Important note:** `Kin.start()` is now working **asynchronously**. You cannot use the SDK before receiving a successful callback from the initiating process! Using the SDK before start is completed will result in an exception.

The chart below showcase the new `Kin.start()` flow. 
**Continue reading this document to better understand the different steps showed in the chart.**

![](/kin-ecosystem-sdk-docs/img/migration_start.jpeg)

## Initiating the SDK
The migration enabled Dev Platform SDK adds a new argument to the `Kin.start()` method: an optional `KinMigrationListener` interface.
This interface allows you to hook into migration-related events (continue reading for more information).

>**Important note:** As of [SDK version 0.9.0](https://github.com/kinecosystem/kin-devplatform-android/releases/tag/0.9.0), a new optional callback was added to `Kin.start()` that allows you to know when the Dev Platform SDK was successfully initiated or if an error occurred. 
In the migration edition, this callback is no longer optional and must be implemented.

## The `KinMigrationListener` interface
 An interface that allows you to hook into migration-related events.
 The interface exposes the following methods:

#### `onStart()`
Is called once the Dev Platform SDK has determined that the current user should be migrated to the Kin Blockchain. It's important to note that this method will not be called after the user successfully migrates. If the user was already migrated when the SDK is initiated (for instance if the user was migrated in previous app session) this method won't be invoked.

Since the migration process can take several seconds, we highly recommend to use the `onStart()` method for communicating to your users that something is happening in the background and your application did not "freeze". Feel free to communicate to your users as you see fit, a message like "Kin upgrade in progress, please be patient" should be sufficient.

#### `onFinish()`
This method will be invoked once the user has been migrated successfully. Now would be a good time to hide any popup or other UI elements you used to communicate with your users on `onStart()`. If the user has already been migrated when the SDK is initiated (for instance if the user was migrated in a previous app session) this method won't be invoked.

>**Important note:** This is not an indication that the SDK was initiated successfully. You still need to wait for the `KinCallback` to return a response or failure. (see example below)

#### `onError()`
This method will be invoked if the user migration has failed with an exception.
We highly recommend using this method to log the exception in your logging/analytics tools of choice.

## Kin.start() code sample
The following code sample shows the usage of the new `Kin.start()` method with the following arguments:
- A `Context`
- A JWT token required for authorizing the user
- The `Environment` we wish to use
- `KinCallback` interface that allows us to know if the SDK initiation was succeeded or failed
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

## Important things to know
Migrating your users to the Kin Blockchain introduces a new temporary challenge. During the migration phase your user-base will be split between migrated users and users who have not migrated yet (i.e. not all the users will be connected to the same blockchain).

Consider the following scenarios:

#### Migration flag raised during user active session
A user launched your app while the migration flag (in our servers) is set to pre-migration (i.e. the user will not go through the migration process). During the user session the migration flag was changed to migration started. 

**Outcome:** The user will not be able to earn/spend Kin's until migrating to the Kin Blockchain.

If you have your own earn/spend/P2P functionality (outside of the provided Kin Marketplace) 
the above scenario will throw a new type of exception - `MigrationNeededException` that you'll need to handle.


For earning opportunities inside the Kin Marketplace - we handle this exception for you in the form of an error dialog to the user.

#### Migrated users who attempt to transfer Kin to non-migrated users
User-A was already migrated and is attempting to transfer Kin to User-B who didn't migrate yet.

**Outcome:** User-A's Kin transfer will successed. User-B will see the transfer in the payment history screen, but will not see their balance updated until migrating to the Kin Blockchain.


## Testing your implementation
During the implementation of the new Dev Platform SDK you can safely test how your application will react to the migration instatiated flag. On the `Playground` environment you can use the following POST HTTP request to change your application flag:

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/<your application id> -H "Content-type: Application/json" -d '{"blockchain_version": "<desired blockchain vesion>"}'`

`<desired blockchain vesion>` can have a value of "2" or "3" as a string. 
- "2" - Set your application flag to pre-migration
- "3" - Set your application flag to migration started

For example, the following HTTP request sets the flag to pre-migration for application id "ABC1":

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/ABC1 -H "Content-type: Application/json" -d '{"blockchain_version": "2"}'`
