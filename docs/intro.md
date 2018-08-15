---
id: intro
title: Introduction
---

## What is Kin?

Kin is the world’s most consumer-ready cryptocurrency built with a blockchain solution that supports more than 1M daily transactions. Place Kin at the center of your user engagement strategy and use it to solve real customer problems.

Give your users fun ways to earn and spend Kin in your app, and help us build a whole new digital world. [Here](https://developers.kinecosystem.com/playbook.pdf) is a suggested Playbook on how to build with Kin.. To see an example of how Kik integrated Kin, check out [this video](https://www.youtube.com/watch?v=hzWsLFI2Fnk).

## Kin SDK Overview

The Kin SDK allows you to quickly and easily integrate with the Kin platform. This enables you to provide your users with new opportunities to earn and spend the Kin digital currency from inside your app or from the Kin Marketplace offer window. For each user, you will create a Kin account and wallet. By calling the appropriate SDK functions, you application performs buy and sell transactions using the user’s wallet objects. Your users may also view their account balance as well as their transaction history.


### Marketplace

The Kin Marketplace is a UI component that you can optionally launch in your app. It displays Earn and Spend offers which may be added to by your app or by the Kin Server itself.to it by your app or by the Kin Server. When a user selects one of these offers, the Kin Marketplace notifies the app that created the offer. The app can then launch the Earn or Spend activity for the user to complete. You may choose to add your custom Earn and Spend offers to the Kin Marketplace so that there is a convenient, visible place where the user can access all offers. Some offers displayed in-app might require that the user choose to navigate to a specific page and therefore might not be so readily visible.

<img src="/kin-ecosystem-sdk-docs/img/marketplace.png" alt="marketplace" height="600px"/>

### Kin Server

The Kin Server handles SDK requests from client apps. This includes the following tasks:

* Creating user accounts.
* Funding user accounts with an initial balance.
* Performing Earn and Spend actions.
* Managing and storing transaction history including the required blockchain management tasks.

Only apps with access credentials can direct requests to the Kin Server.

### Kin Wallets and Accounts

Every user who transacts with Kin owns the following two items:

* Wallet – contains the user’s authentication keys (does not contain currency, unlike a real-world
wallet).
* Account – an identity recognized by the Kin blockchain which is associated with a Kin balance
and a transaction history log.
