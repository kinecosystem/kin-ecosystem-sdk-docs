---
id: faq-sdk
sidebar_label: SDK FAQ
title: Frequently Asked Questions
---

## SDK FAQ

### What is the Kin account ID?
Kin account id is the public key (from the private/public key pair). Once integrated, the Kin SDK provides an api to query the user public address. The public address will only be available after the SDK start method was called.

### How do we deal with user's transactions outside our app?
Currently a user can’t access or export his private key and therefore cannot sign any transaction outside of the app.

### If we display the user's Kin in our app, is it the total kin in Kik or only kin in our app? If the user earns the kin from our app, and trades in other systems, is this user our activated user?
Each app has its own unique private/public key generated and securely stored on the device. Therefore each app has its own Kin account with separated balance, there is . Your app Kin balance will only be affected by transactions within your app and have no connection or affect on Kin user balance on any other installed app such as Kik and vice versa.

### Do you have backup and restore options?
The Kin SDK provides automatic backup functionality via Google backup on Android and iCloud on iOS. The backup will restore the private key if users reinstall the app under the same Google/icloud account (even on a different device). However the automatic backup is not 100% guaranteed and a successful automatic backup and install depends on several external factors such as android os version, on whether the user has opted-out from application backup, and whether the user is constantly on data or is using WiFi, or additional cases that we can’t control.
We are currently researching and testing user activated backup/restore functionality, and it will be added to future version of the SDK.

### After uninstalling and reinstalling the app will the user lose the earned KIN?

The SDK is doing its best effort to automatically backup and restore the user’s private key. However, the success is not guaranteed as it depends on external factors (Android OS version, user app backup opt out, etc.). Therefore, in an event that the app is uninstalled or the cache is cleared, there is not a 100% guarantee that the backup/restore function will be successful. he user in this event may lose their Kin balance. We are well aware of this requirement and we are working on providing the right solution for user facing, intuitive, and safe backup/restore functionality.

### How do I set up a test (fake) account with an initial amount of KIN?

There is no need to create a fake account with any amount of Kin. Playground environment provides earn opportunities without the limits that will be applied in production environment. For testing and debugging purposes, earn task can be performed multiple times in order to fund your account for performing a spend opportunities.

### How do I move from production to test environment?
When initiating the client SDK you will have to provide an environment configuration. Each configuration will contain blockchain environment configuration (horizon url, network passphrase, KIN issuer, etc.) and Kin service configuration (Kin server url, webview url, etc.).

### What exactly happens when we start the SDK?

- Client Wallet is created locally (private and public keypair is created and private key is stored securely on client device).
- Client register to server providing JWT credentials and public address.
- Kin server verify credential and create account for provided public address on Blockchain.
- Client receive auth token to be used with any network request to Kin server.
- SDK watch blockchain and get notified when client account created.
- SDK create a trustline transaction (Account can’t receive Kin without establishing trustline).

### How can I know if `Kin.Start()` completed successfully?

There is no need to wait for `Kin.start()` to be completed. Any API call can be called immediately after calling Kin.start(). Make sure that from any thread `Kin.start()` is called before any other call.

`Kin.purchase(JWT, callBack)` (and other asynchronous calls) can be called immediately after the call to start. The SDK will complete the Purchase flow only after the start process completed successfully and calls callback upon success. If there were any failure in `Kin.start()` the call back will be called with an error specifying that there was a problem in the start process).

`Kin.launchMarketplace(...)` can be called immediately. Marketplace UI will handle “progress bar” UI/UX until start process completed.

### Can we get a list of items that were purchased?
Kin server saves the order history of items that were purchased, but as this data can be large, the best practice is to send each payment confirmation JWT that is provided by Kin server, to your app server and save it in application server DB for your app needs.

### What happens exactly when I call `purchase(JWT,callBack)`?

* Client sends a create order request with offer JWT to Kin server.
* Kin server verifies JWT and provides the client the amount, payment address and internal orderID.
* Client signs a transaction with provided information (the internal orderID is set in the transaction memo).
* Client submits the order with the orderID and kin server monitors blockchain for transactions with the orderID in the memo field.
* Once transaction is accepted by the blockchain, the client requests the order from server side.
* Kin server verifies that the transaction was accepted by the blockchain and signs a JWT confirming the transaction.
* The JWT is returned to the purchase callback.
* Client can pass the JWT to digital service server to unlock the offer.

### Does the SDK store the balance in memory automatically or does each call to balance perform a network call?

Yes, when using the callback method. Each call would make a request.

Using the observable object, You would instantly get a pending balance which is indeed persisted by us, but the SDK would have to make a network call (to the blockchain layer) in order for you to have a verified balance. It is the common practice with all blockchain infrastructures to always assume any balance is pending; unless you verified it with a call.

Opening an observer opens an SSE (Server Sent Event) connection to the blockchain network which should be closed when its no longer needed. Add the observer only when needed. Anywhere else, callback api or the cached balance API should be used.