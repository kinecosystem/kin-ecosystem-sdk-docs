---
id: sdk-components
title: SDK Components
---

![](/img/sdk_components.png)

**Ecosystem server** - client authorisation, 2-way verifying of offers integrity and trusted origin using JWT, managing offers inventory, managing DS wallets.  

**Application server** - Authorise DS clients, 2-way verifying of offers integrity and trusted origin using JWT.  

(signs JWT request for client, validate ecosystem server JWT responses via client).  

**Client** - communicate with ecosystem server, marketplace UI, native earn/spend, send and validate server transaction.  

**Kin Blockchain** - Kin coin decentralized ledger, accept transactions from Ecosystem server and Ecosystem client sdk.
