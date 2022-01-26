#  Coordinators

## Overview

### What is a Hermez Coordinator?
A Coordinator is our term for rollup block producer. At any one time, there is one Coordinator responsible for creating blocks on the rollup chain selected among a number of registered nodes via an auction process.

### How many Coordinators are there?

There is no limit to the number of registered Coordinators. Becoming a Coordinator is entirely permissionless via an auction process and it will be enabled shortly after launch.

Although the first Coordinator will be the Hermez Boot Coordinator which will forge blocks when there are no alternative bids in the auction, it’s important for us that the market for coordinators becomes open over time.

### How is the Coordinator Node selected?

The Coordinator Node is selected using an auction process, where registered nodes bid for the right to forge batches during a time slot. The highest bidder in a given slot will become the Coordinator

### Will there be a competitive market for Coordinators?

We are committed to creating a competitive market for Coordinators and we will open-source software to allow anyone to run a Coordinator node.

## Auction Process

### How do I become a Coordinator?

To become a Coordinator you need to prepare the system infrastructure, take part in the auction, and win a bid for a time slot.

### How are bids placed in the auction?

Coordinators will participate in the auction by sending an on-chain transaction to the auction smart contract. Bids are always paid in HEZ.

### Where do I get HEZ?

HEZ can be easily obtained in [Uniswap](https://app.uniswap.org/)

### Is there a minimum amount of HEZ tokens you have to hold to be a Coordinator?

You need to have at least as many tokens as your bid ammount.

### The Coordinator I am running has lost a bid. Where does the bid go?

In case your bid didn't win the slot, the amount bid is transferred to the Auction smart contract under your account. You can use this amount for future bids, or withdraw it to the Coordinator account.

### How long are Coordinator time slots?

Slots will be 40 Ethereum blocks long (10 minutes). During this time, a Coordinator can forge as many batches as possible.

### What happens if the Coordinator goes offline?

If the Coordinator has not done anything in the first part a slot, then anyone can jump in and replace it by forging blocks (first come first served).

### How do Coordinators make money?

Coordinators set and collect the transaction fees. They can expect a revenue per transaction and each coordinator will select to forge the more profitable transactions from the transaction pool.

They profit from these fees minus the operational costs and the bid price for the slot.

## Running a Coordinator

### What are the infrastructure requirements to run a Coordinator?

To run a Coordinator you need the following:
- PostgreSQL database
- Ethereum Node
- Coordinator Server running the [hermez-node repository](https://github.com/hermeznetwork/hermez-node). A reasonable server spec is AWS instance c5ad.2xlarge with 8 vCPU and 16GB RAM
- Proof Server running the [rapidsnark repository](https://github.com/iden3/rapidsnark). A server spec able to run the ~2000 transaction circuit is AWS instance c5ad.24xlarge with 96 vCPU and 192GB RAM. 


### Is there a step by step guide to configure a Coordinator node?

Yes. You can find this guide [here](../developers/coordinator.md).

### How do you register as a Coordinator?

You can use a JavaScript tool, [cli-bidding](https://github.com/hermeznetwork/cli-bidding) to register your node as a Coordinator.

### How many transactions can be processed per batch?
 
The number of transactions per batch depends on the circuit used. Hermez accepts two different circuit sizes: 400 transactions and 2048 transactions. 


### I am having technical issues running a Coordinator node. Who can I contact for support?

We recommend to go over this FAQ first, and if you still have issues, please send an email to hello@hello@hermez.network, or send a message to our [Discord](https://bit.ly/hermez-discord). You can also find more information in the [Hermez documentation page](https://docs.hermez.io)


