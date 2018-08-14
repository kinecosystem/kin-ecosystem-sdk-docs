---
id: sdk-components
title: SDK Components
---

![](/kin-ecosystem-sdk-docs/img/sdk_components.png)

### Kin Server

* Create and activate users accounts on the Stellar blockchain.
* Serving out of the box KIN's earn and spend offers for the SDK marketplace.
* Manage and store user's earn and spend order history.
* Manage digital service wallets.
* Client authorization and secure communication between digital service and Kin server, see [Authorization and JWT](jwt.md).

### Application Server

* Client authorization and secure communication between digital service and Kin server, see [Authorization and JWT](jwt.md).
* Manage client native earn and spend offers.

### Client

* Communicate with Kin server.
* Manage client wallet.
* Provides SDK marketplace UI for both out of the box and native offers.
* Sending and receiving transaction on Kin blockchain.

### Kin Blockchain

* The Kin coin decentralized ledger.
* Accept transactions from Kin server and Kin client sdk.
