---
id: faq-blockchain
sidebar_label: Blockchain FAQ
title: Frequently Asked Questions
---
## Blockchain FAQ

### Which Blockchain solution are you using?

We started with Ethereum and then moved to Stellar testnet blockchain.
We are currently building a new Kin Blockchain which is a fork of Stellar.
 
### What does it mean when you say you are migrating Kin to Stellar or Kin Blockchain, or any other alternative blockchain? What is the purpose of it?

Migration refers to moving between systems in one direction. In this case, however, migration of Kin to Kin Blockchain means using Kin Blockchain as an alternative blockchain that runs in parallel. In the instance of Kin Blockchain, we would use both Ethereum and Kin Blockchain side-by-side. Tokens will be swappable between ERC20 and Kin Blockchain and will be completely equal in value. There are many benefits to the migration - fees are much lower on blockchains like Kin Blockchain, the blockchain is more predictable, and confirmation times for transactions are much faster, In addition, throughput is higher since the network is less congested. This will allow us to scale Kin as we build our ecosystem of digital services. Our migration process will be as follows:
Phase #1 - create KIN on Kin Blockchain that will only be available there
Phase #2 - add swap between Ethereum and Kin Blockchain, tokens will be available through both blockchains.

### How many transactions per second will the Blockchain support?

We currently support 125 tx/ sec

### How long do transactions take to settle on the Steller network?

Approximately 5 seconds on average

### What is the cost per transaction?

Since we are running on our network, there is no cost for transaction for now. We might use fractions of Kin for fees in order to avoid spamming the network.

### How do users transfer Kin into and out of their wallets?

Users can earn and spend Kin through different opportunities within the marketplace and within the native experiences you implement into your app. In the first phase, users will not be able to transfer their Kin out of their wallet. 

### When a user registers, do they get an address on the blockchain or is it managed within a group account?

For every user a private/public key pair is generated on the client side. The SDK will provide a method to get the public key. 

### Is there an update to how the Kin Rewards Engine works? 

We are working with MIT and running tests to see if we need to make further adaptations once there are real transactions running with our first few partners. We will start with a manual verification of the KRE prior to going fully automated and will slowly transition based on the number of transactions and validity of the model. 

### What resource do we have for securing accounts after a security breach?

There is no centralized place where we keep any personally identifiable information (PII) or user data or private keys. We have no control on the accounts once they are created since we have no access to the user’s private key. 

### Do you keep a record of every transaction off of the blockchain? What API access do we have? If a user wants to see transaction history is that done through your DB or by querying the blockchain?

We will provide a report of all transactions originated from each of the apps that the SDK is integrated in. The order history includes blockchain completed transactions. The details of the order history are received from the ecosystem server. The ecosystem server is open sourced and digital service can choose to provide their own offers directly (will be supported based on demand). We will provide tools for digital services to query stellar directly and filter all these transaction directly from stellar blockchain (based on demand).

### How does stellar works?

Stellar is a blockchain based on Federation unlike Ethereum and Bitcoin, which makes transactions faster and lower the cost. More information can be found here.

### Why is it more secure?

Its not. Security is not in our considerations when moving to Stellar or Kin Blockchain. Ethereum security is driven from permissionless nodes and PoW and Stellar security derives from pre-trusted nodes that approves transactions.

### Why do we need a federation?

It’s about x6 faster than ethereum because we pre-trust and can use a non PoW algorithm.
There are less entities in blockchain - permissioned not everyone can run a node. The node can run by anyone but it won’t be in a quorum, so no one will accept its blocks. We need you to join the federation, If only Kik runs nodes, we’re an MSB.

### How many nodes do we need in the federation?

SCP (Stellar Consensus Protocol) allows upto 20% failed nodes to continue to pass consensus. so we need 7 to allow 1 to go down and still work. all the nodes in the quorum agree on new  blocks.

### What does it mean to be in the federation, operation wise?

At least 1 core node + db, we also want you to run horizon + db. each core has it’s own signing key.

### What are the costs of operating a federation node?

These are our numbers from our deployment on AWS:  
500$ for horizon c5.4xl  
1000$ for postgres RDS m4.4xl  
30$ for core t2.medium  
50$ for core postgres RDS t2.medium  
\+ price for monitoring  
\+ price for redundancy + ELB  

### Do we need to join playground federation?

No need - there is no compliance here.

### What kind of operation/monitoring revolves around running a federation node, do we provide support?

On demand, we will share our monitoring scheme so you can duplicate it.
you are in charge of keeping the node up, we don’t have any access to your code and we shouldn’t because of legal obligations.
We might be able to provide/develop a status page on a node, so we can monitor individual nodes in the federation.
We will have a contact exchange and perhaps joined monitoring: phone exchange/ joined slack/ joined PD/ joined datadog.
We will review each other’s upgrades and new deployments, to keep everyone on track and ensure quality.

### Which node should our clients connect to?
to a DNS we provide which is geo-balanced and is health checked. We need to understand how to route different domains with different certs.