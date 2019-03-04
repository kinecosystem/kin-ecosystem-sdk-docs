---
id: migration_ios
sidebar_label: iOS
title: Kin Blockchain Migration - iOS
---

## Before We Begin
Moving to the new Kin Blockchain has been a top priority of Kin's for the past few months.
The Kin Blockchain opens a world of new possibilities for Kin developers, and we're very excited to onboard you.

This document will walk you through your migration process as a Kin developer. Please read it carefully.

 >**Important note:** You should thoroughly test the migration process in the Playground environment before moving to Production.
 
### Terminology
- **Old Blockchain**: The old blockchain that you're currently using.
- **Kin Blockchain**: The new blockchain that you'll be migrating to.
- **Dev Platform SDK**: The SDK that you use.

## Migration Overview - How Is It Going to Work?
Let's start by understanding what migrating an account actually means. Note that this process is similar for end-user accounts and digital service accounts (that's yours).
Migrating an account to the Kin Blockchain consists of the following three main steps:

1. **A new account is created on the Kin Blockchain -**
This new account has *the same* keypair (i.e., private and public addresses) as the account on the Old Blockchain.
The account is pre-created before the migration begins with a balance of zero.

2. **The Old Blockchain account is "deactivated" -**
Deactivating an account ("burning") means the account can no longer earn or spend Kins. 
However, the account's last known balance and transaction history are still available.

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

Before we migrate you and your users' accounts, you need to complete the following tasks:
1. Integrate the new Dev Platform SDK into your app. 
2. Publish your new, migration-ready app to the App Store.
3. Notify Kin when your app reaches the desired adoption rate.
4. Decide together with Kin on a migration date (preferably, at low traffic hours).
5. Create and send Kin a signed transaction for burning both your wallet and the Shared Wallet.

>**Important note**: DO NOT send this transaction by yourself to the old blockchain! We will send it when the migration starts.

#### Migrating Your End Users 
Once you release a new version of your application that implements the migration-enabled Dev Platform SDK, it will take care of migrating your users automatically when you first initate the SDK (continue reading for more details).

## Understanding the User Migration Flow
To support migration to the Kin Blockchain an additional step was added "under the hood" to the Dev Platform SDK instantiation flow.

In previous versions of the Dev Platform SDK, calling `Kin.start()` would immediately initiate the SDK. In the migration edition, this is no longer the case.
Now, when `Kin.start()` is called, the Dev Platform SDK will query our servers and check if the current application's migration process has been initiated. If true, the SDK will proceed to check if the current user has already migrated to the Kin Blockchain. If the current user has not migrated, the Dev Platform SDK will begin the user migration flow. Only once the user has successfully migrated to the Kin Blockchain will the SDK be initiated.

>**Important note:** `Kin.start()` is now working **asynchronously**. You cannot safely use the SDK before receiving a successful delegate call!

The chart below shows the new `Kin.start()` flow. 
**Continue reading this document to better understand the different steps shown in the chart.**

![](/kin-ecosystem-sdk-docs/img/migration_start_ios.jpeg)

## Initiating the SDK
The migration-enabled Dev Platform SDK adds a new property to the `Kin` class `var migrationDelegate: KinMigrationDelegate?`.
This protocol allows you to hook into migration-related events, and requires stubs to be implemented (continue reading for more information).

## The `KinMigrationDelegate` Protocol
A protocol that allows you to hook into migration-related events.
The protocol exposes the following new, required methods, each of which *must be stubbed out* in the class you chose to implement the protocol:

#### `kinMigrationDidStart()`
This method is called once the Dev Platform SDK has determined that the current user should be migrated to the Kin Blockchain. It is important to note that this method will not be called after the user has been successfully migrated. If the user has already been migrated when the SDK is initiated (for instance, if the user was migrated in the previous app session), this method will not be called.

Since the migration process can take several seconds, we highly recommend to use the `kinMigrationDidStart()` method for communicating to your users that something is happening in the background and your application has not "frozen". Feel free to communicate to your users as you see fit; a message such as "Kin upgrade in progress, please be patient" should be sufficient.

#### `kinMigrationDidFinish()`
This method is called once the user has been migrated successfully. Now would be a good time to hide any popup or other UI elements you used to communicate with your users on `kinMigrationDidStart()`. If the user has already been migrated when the SDK is initiated (for instance if the user was migrated in the previous app session), this method will not be called.

>**Important note**: This is not an indication that the SDK was initiated successfully. You still need to wait for the `kinMigrationIsReady()` to be called.

#### `kinMigrationIsReady()`
This method is always called when the SDK is initiated successfully, regardless of `kinMigrationDidFinish()` being called.

#### `kinMigration(error: Error)`
This method will be called if the user migration has failed with an error.
We highly recommend using this method to log the error in your logging/analytics tools of choice.

## Example implementation of Delegate Protocol in the ExampleViewController:

import KinDevPlatform

class ExampleViewController: UIViewController, KinMigrationDelegate {
    override func viewDidLoad() {
           super.viewDidLoad()
           
        //set the delegate
        Kin.shared.migrationDelegate = self
        //start SDK
        
    }

    func kinMigrationDidStart() {
        // implement loading screen to let users know the migration has started
    }

    func kinMigrationDidFinish() {
        // stop loading screen and alert user the migration is complete
    }

    func kinMigrationIsReady() {
        // set flag to determine that app is ready to be used
    }

    func kinMigration(error: Error) {
        // log error and alert the user that there was a migration issue
    }

}

*Important note*: All of the above functions must be implemented / stubbed out when using the KinMigrationDelegate Protocol or you will receive an error.


## Important Things to Know
Migrating your users to the Kin Blockchain may pose a new temporary challenge. During the migration phase, your user base will be split between migrated users and users who have not been migrated yet (i.e., not all the users will be connected to the same blockchain). As a result, you may encounter the following situations: 

#### Migration flag raised during an active user session
A user launched your app while the migration flag (on our servers) is set to pre-migration (i.e., the user will not go through the migration process). During the user's session, the migration flag was changed to *migration started*. 

**Outcome:** The user will not be able to earn/spend Kins until `Kin.start()` is called again.

If you have your own earn/spend/P2P functionality (outside of the provided Kin Marketplace) the above scenario will throw a new type of error - `KinMigrationError.migrationNeeded` that you will need to handle.

For earning opportunities inside the Kin Marketplace, we handle this error for you in the form of an alert displayed to the user.

#### User attempts to transfer Kin from a migrated account to a non-migrated account 
User transfers Kin from account A, which has already been migrated, to account B, which has not been migrated yet.

**Outcome:** For account A, the transfer will succeed. In account B, the transfer will appear in the payment history, but its balance will not be updated until migrating to the Kin Blockchain.

## Testing Your Implementation
During the implementation of the new Dev Platform SDK you can safely test how your application will react to the *migration instatiated* flag. On the `Playground` environment, you can use the following POST HTTP request to change your application flag:

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/<your application id> -H "Content-type: Application/json" -d '{"blockchain_version": "<desired blockchain vesion>"}'`

`<desired blockchain vesion>` can have a value of "2" or "3" as a string. 
- "2" - Set your application flag to pre-migration
- "3" - Set your application flag to migration started

For example, the following HTTP request sets the flag to pre-migration for application ID "ABC1":

`curl -XPOST https://api.developers.kinecosystem.com/v1/config/blockchain/ABC1 -H "Content-type: Application/json" -d '{"blockchain_version": "2"}'`
