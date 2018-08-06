---
id: faq-sdk
sidebar_label: SDK FAQ
title: Frequently Asked Questions
---

## SDK FAQ

### What is the Kin account ID?
Kin account id is the public key (from the private/public key pair).
How do we get it after we are integrated?  
The Kin SDK provides an api to query the user public address. The public address will only be available after the SDK start method was called.

### How do we deal with user's transactions outside our app?
Currently a user can’t access or export his private key and therefore can’t signed any transaction outside of the app.

### If we display the user's Kin in our app, is it the total kin in Kik or only kin in our app?
If the user earns the kin from our app, and trades in other systems, is this user our activated user?
Each app has its own unique private/public key generated and securely stored on the device. Therefore each app have its own Kin account with separated balance, there is . 365 Kin balance will only be affected by transactions within 365 app and have no connection or affect on Kin user balance on any other installed app such as Kik and vice versa.

### Do you have backup and restore options?
The Kin SDk provide automatic backup functionality via Google backup on Android and iCloud on iOS. The backup will restore the private key if user reinstall the app under the same Google/icloud account (even on a different device). However the automatic  backup is not 100% guaranteed and a successful automatic backup and install depends on several external factor such as android os version, on whether the user has opted-out from application backup, and whether the user is constantly on data or is using WiFi or additional cases that we can’t control.  
We are currently research and test users activated backup/restore functionality and it will be added to future version of the SDK.

### After uninstalling and reinstalling the app will the user lose the earned KIN?

The SDK is doing its best effort to automatically backup and restore the user’s private key however the success is not guaranteed as it depends on external factors (Android OS version, user app backup opt out etc.). Therefore, in an event of app uninstall or cache clear, there is no 100% guarantee that the backup/restore succeeded and the user may lose his kin balance in such an event. We are well aware of this requirement and we are working on providing the right solution for user facing intuitive and safe backup/restore functionality  

### How do I set up a test (fake) account with an initial amount of KIN?

No need to fake account with initial Kin:
Playground environment provides  earn opportunities for testing environment where QA can quickly earn enough Kin for all testing( this will be the fastest way to achieve test)
You can use native earn option (ready on Android WIP on iOS)  to request Kin programmatically from within kik
Create a const earn JWT and use native earn API to request Kin for user - each JWT will only work once per user ID

### How do I move from production to test environment?
When initiating the client SDK you will have to provide an environment configuration: 
Each configuration will contain blockchain environment (horizon url, network passphrase, KIN issuer etc) configuration and ecosystem service configuration (ecosystem backend url, webview url etc.).

### What exactly happens when we start the SDK?

Client Wallet created locally (private and public keypair is created and private key is stored securely on client device)
Client register to server providing JWT credentials and public address
Ecosystem backend verify credential and create account for provided public address on Blockchain
Client receive auth token to be used with any network request to Ecosystem backend 
SDK watch blockchain and get notify when client account created
SDK create a trustline transaction (Account can’t receive Kin without establishing  trustline)
What happens when calling purchase/ getBalance/launchMarketPlace without ever calling the `Kin.start()` ?
Asynchronous request callback will be called with an error, MarketPlace will launch but will show error message to the user. Currently (iOS, 0.4.9) the launch mp function will just silently fail. Starting 0.5.0, its a throwing function.
 
### How can I know if `Kin.Start()` completed successfully?

There is no need to wait for `Kin.start()` to be completed. Any API call can be called immediately after calling Kin.start(). Make sure that from any thread `Kin.start()` is called before any other call.

`Kin.purchase(JWT, callBack)` (and other asynchronous calls) can be called immediately after the call to start. The SDK will completed the Purchase flow only after start process completed successfully and call callback upon success. If there were any failure in `Kin.start()` the call back will be called with an error specifying that there was a problem in the start process).  
`Kin.launch Marketplace(...)` can be called immediately, marketplace UI will handle “progress bar” UI/UX until start process completed  

### Can we get a list of items that were purchased?

Yes, but our server should not be used as part of the critical part of the Digital service.  
Our service can provide the order history and payment confirmation JWT but it should be used to receive it once and cache locally on your own servers. Don’t reconstruct your internal data using this API.

### What happens exactly when I call `purchase(JWT,callBack)`?

* Client send a create order request with offer JWT to Kin server.
* Kin server verify JWT and provides the client the amount, payment address and internal orderID.
* Client sign a transaction with provided information (the internal order id is set in the transaction memo).
* Client submit the order with the order id and kin server monitor blockchain for transaction with the order id in the memo field.
* Once transaction accepted by blockchain client request the order from server side.
* Kin server verify that the transaction was accepted by the blockchain and signs a JWT confirming the transaction.
* The JWT is returned to the purchase callback.
* Client can pass the JWT to digital service server to unlock the offer.

### Does the SDK store the balance in memory automatically or does each call to balance perform a network call?

When using the callback method - yes. Each call would make a request.  
Using the observable object: You would instantly get a pending balance which is indeed persisted by us, but the sdk would have to make a network call (to the blockchain layer) in order for you to have a verified balance.
It is the common practice at all blockchain infrastructures to always assume any balance is pending, unless you verified it with a call.

Opening an observer opens an SSE (Server Sent Event) connection to the blockchain network which should be closed when its no longer needed. Add the observer only when needed, anywhere else, callback api or the cached balance API should be used.