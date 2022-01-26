# Developer Guide

This document is an overview of the Hermez Protocol. Its objective is to provide an introduction to developers on the Hermez Protocol so that the use of the tools which interact with Hermez Network, such as [`HermezJS`](../sdk#dk) (javascript SDK) and the [`REST API`](../api#api), become simpler.  This document assumes you are familiar with the Ethereum ecosystem and L2 Rollups (in particular ZK-Rollups). 

For a more in depth analysis, read the [`protocol`](../protocol/hermez-protocol/protocol) section.

Hermez smart contracts can be downloaded from [`here`](https://github.com/hermeznetwork/contracts).

## Overview
Hermez is a [`zk-rollup`](../glossary#zk-rollup) solution that allows scaling payments and token transfers on top of the Ethereum public blockchain. It uses Ethereum for data storage but not for computation. In addition, by using zero-knowledge proofs, it is easy to verify on-chain that computations have been carried out correctly.

All accounts and balances in Hermez Network are stored off-chain in a [`state tree`](../glossary#state-tree). Incoming user [`transactions`](../glossary#transactions) are [`batched`](../glossary#batch) together, and through a [`zk-SNARK`](../glossary#zk-snark) that proves that those transactions meet certain rules specified in a smart contract, the state tree transitions to a new verifiable valid state.

The [`coordinator`](../glossary#coordinator) is the entity that collects and codifies these transactions, calculates the ZK-SNARK proof and submits the result to the smart contract that validates the transition. Transactions are made public to provide [`data availability`](../glossary#data-availability) to the protocol so that anyone can rebuild the state tree from on-chain data.

Users typically send transactions to Hermez via a wallet. The purpose of this tool is to improve the experience of using Hermez by hiding the internal interactions between the different Hermez components and simplifying the usage. 

The [`governance`](../glossary#governance) is the  entity that oversees the sustainability and evolution of the network. Some functions delegated to the governance include the upgrade of smart contracts, the modification of [`system parameters`](../glossary#system-parameters), or the execution of the [`withdrawal delay`](#withdrawal) mechanism among others.

Hermez deploys three main smart contracts:
1. **Hermez smart contract**: Manages the [`forging`](../glossary#forging) stage by checking the zk-proofs provided by the selected coordinator, and updates the state and exit trees. It also interacts with users by collecting L1 transactions and adding them to the transaction queue.
2. **Consensus smart contract**: Manages the selection of a coordinator node via an auction process.
3. **WithdrawalDelayer smart contract**: Manages a withdrawal protection mechanism embedded into the system.

The overall picture of Hermez can be seen in the diagram below.

![](hermez_zkrollup.png)

Users send L1 transactions (such as Create account, Deposit or Withdrawal requests) using a UI. These transactions are collected by the Hermez smart contract and added into a queue of pending transactions.  Users may also send L2 transactions (Transfer, Exit) directly to the coordinator node. The UI hides all the unnecessary complexities from the user, who just selects the type of operation and the input data for a given operation (source account, destination account, amount to transfer,...).
At the time of processing a batch, the coordinator takes the pending L1 transactions from the Hermez smart contract and the received L2 transactions, and generates a proof showing that these transactions have been carried out correctly. This proof is given to the smart contract that verifies it and updates the state of the network.  In the meantime, an auction process is ongoing to select the coordinator node for a given amount of time. In this auction, nodes bid for the right to forge upcoming batches and thus collecting the fees associated to those transactions. The proceedings of these bids will be sent to different accounts, including a  Gitcoin grants account.


Hermez functionalities can be summarized in 4 major groups:
1. Handling L1-user and L2-user transactions
2. Forging batches
3. Reaching [`consensus`](../glossary#auction) to select a coordinator
4. Withdrawal of funds.


![](L1diagram.png)

## Accounts
Hermez stores accounts as leaves in the Hermez state tree. Each account stores a single type of token. A user may own multiple rollup accounts.

There are two types of accounts to operate in Hermez Network:
1. **Regular**: Regular accounts can be used in both L1 and L2 transactions. Regular accounts include an Ethereum and a [`babyjubjub`](../glossary#BabyJubJub) public key. An Ethereum key is used to authorize L1 transactions and the Baby Jubjub key is used to authorize L2 transactions. An Ethereum address may authorize the creation of a Regular account containing that same Ethereum address plus a Baby Jubjub public key. Typically, this is done via a UI.

2. **Internal**: Internal accounts only have a Baby Jubjub key, and thus may only be used in L2 transactions. Since there is no Ethereum address, the account creation does not require an authorization and will only require a Baby Jubjub key.


## Transactions

There are two types of Hermez transactions:
- **L1 transactions** are those that are executed through the smart contract. These transactions may be started by the user or by the coordinator.
- **L2 transactions** are those that are executed exclusively on L2.

### L1 Transactions
L1 transactions can be divided in two groups depending the originator of the transaction:
- **L1 User Transactions**: originate from an Hermez end-user using some form of UI. 
- **L1 Coordinator Transactions**: originate from the coordinator.

#### L1 User Transactions
L1 user transactions (L1UserTxs) are received by the smart contract. These transactions are concatenated and added in queues to [`force the coordinator`](#L1L2-Batches) to process them as part of the batch. The queue that will be forged in the next L1L2-batch is always frozen, and the L1 Transactions will be added in the following queues. In case a transaction is invalid (e.g. attempts to send an amount greater than the account balance) it will be processed by the circuit but will be nullified.

This system allows the L1 transactions to be uncensorable

Examples of L1 User transactions include `CreateAccountDeposit`, `Deposit`, `DepositTransfer`... All the transactions details are handled by the UI.
#### L1 Coordinator Transactions
L1 Coordinator Transactions (L1CoordinatorTxs) allow the coordinator to create regular or internal [`accounts`](#accounts) when forging a batch so that a user can transfer funds to another user that doesn't own an account yet.

### L2 Transactions
L2 transactions (L2Txs) are executed exclusively on L2. Examples of L2 transactions include `Transfer` of funds between rollup accounts or `Exit` to transfer funds to the exit tree. All L2 transactions are initiated by the user, who sends the transactions directly to the coordinator via a [`REST API`](../api#api). Depending on the UI capabilities, the user may be able to select among a number of coordinators (the one currently forging, the ones that have already won the right to forge in upcoming slots,...).

Fees are payed on L2 transactions in the same token used in the transaction. The coordinator collects these fees from up to 64 different tokens per batch. If more than 64 tokens are used in the same batch, no fees will be collected for the excess number of tokens. 

### Transaction ID
Transaction ID (TxID) allows tracking transactions from the time they are sent to the coordinator to the time they are made available on
chain.

TxID is computed differently depending on the type of transaction (L1UserTxs, L1CoordinatorTxs or L2Txs).
Padding is used to make all TxID (no matter which type) have the same length of 33 bytes.

- TxID on L1UserTx:
  ```
  bytes:   | 1 byte |             32 bytes                |
                      SHA256(    8 bytes      |  2 bytes )
  content: |   0    | SHA256([ToForgeL1TxsNum | Position ])
  ```
- TxID on L1CoordinatorTx:
  ```
  bytes:   | 1 byte |             32 bytes        |
                      SHA256( 8 bytes  |  2 bytes )
  content: |   1    | SHA256([BatchNum | Position ])
  ```
- TxID on L2Tx:
  ```
  bytes:   | 1 byte |             32 bytes        |
                      SHA256( 6 bytes | 4 bytes | 2 bytes| 5 bytes | 1 byte )
  content: |   2    | SHA256([FromIdx | TokenID | Amount |  Nonce  | Fee    ])
  ```
## Forging
In this section we will describe how consensus to select a coordinator with the permission to forge batches and collect fees from the processed transactions is reached. We will also describe some of the embedded security mechanisms that discourage these coordinators from acting maliciously.

### Consensus

In Hermez zkRollup, time is divided into slots of a certain duration:
  - Ethereum Block = ~ 15s.
  - Slot = 40 Ethereum Blocks ~ 10 min.

![](consensus-1.png)

Hermez reaches a consensus on who will play the role of coordinator by running an auction managed by a smart contract. This auction is held among the existing nodes for every slot. The node that places the highest bid for a given slot while the auction is open will claim the role of coordinator for that slot. 

The coordinator node is allowed to forge batches during the awarded slot, which is the mechanism by which an authorized coordinator processes a batch of transactions, produces a ZK-SNARK attesting to the correctness of the operation and is able to reclaim the processing fees.

Auction bids are placed only in [`HEZ`](../glossary#hez). The auction of future slots opens up to **S1** slots in advance. Auction closes **S2** slots before the beginning the slot. Tentative **S1** and **S2** values are 1 month and 2 slots respectively. Additionally, these parameters can be changed by governance at any time.

Bids placed during the auction should be at least greater than the minimal bidding price if it's the first bid in a slot, or a premium bid factor **P** % higher than the previous bid. Both the minimum bidding price and the premium bid factor(**P**) can be modified by the network governance. Tentative values for minimum bid and premium factor are 10 HEZ and 10% respectively. Bids not meeting these conditions will not be valid and bidders will receive their HEZ when the slot is forged.


![](consensus-2.png)

![](consensus-3.png)

#### Allocation of Bids
All bids are deposited in the consensus smart contract the moment they are placed. 

Once the slot is forged, the tokens bid are assigned to three different accounts:
- Part will be **burnt**. 
- Part will be assigned to a **donations account** with Gitcoin grants, which will decide how to allocate this funds into different projects.
- Remaining tokens will be allocated to an **incentives account**, compensation active engagement and network adoption.

### Protection Mechanisms

There are some rules on how coordinators must process transactions. These rules ensure that a coordinator will behave correctly and efficiently, attempting to forge as many transactions as possible during its allocated slots. 

#### L1/L2 Batches
There are 2 types of batches:
- **L2-batch**: Only L2 transactions are mined.
- **L1L2-batch**: Both L1 and L2 transactions can be mined. In these batches, the coordinator must forge the last L1 queue.

In both cases the coordinator may include L1-coordinator-transactions.

Coordinators must process L1 user transactions periodically. The smart contract establishes a deadline for the L1L2-batches. This deadline indicates the maximum time between two consecutive L1L2 batches. Once this deadline is reached, the coordinator cannot forge any L2 batches until the deadline is reset which only happens after a L1L2 batch is forged.

This mechanism is summarized in the diagram below.

![](forgeL1L2.png)

#### Coordinator Override
If for some reason the coordinator of the current slot doesn't forge any batch in the N first available blocks inside the slot, any available coordinator may forge batches without bidding. This maximum idle time is called **Slot deadline**, and defines the amount of time that any coordinator must wait to start forging without bidding, provided that the coordinator that won the current slot action hasn't forged anything during that time.

This mechanism ensures that as long as there is one honest working coordinator, Hermez Network will be running and all funds will be recoverable. 

#### Boot Coordinator
Hermez includes the role of Boot coordinator managed by the network. The Boot coordinator acts as the bootstrap mechanism and its mission is to guarantee that there is always coordinator available in the early stages of the project.

#### Slot Grouping
Auction is structured in groups of **6 slots** (i.e, slots are sequentially indexed 0,1,2,3,4,5,0,1,...). Each slot index has an independent minimum bidding price. This grouping allows certain flexibility to the governance to influence behavior of coordinators. If slots are frequently wasted (meaning that elected coordinators chose not to forge batches), governance may increase the minimum bid amount for certain slots to make slot wasting less efficient for coordinators, and thus allowing the boot coordinator to forge the batches.

When the minimum bidding price is set to **0 HEZ value** for a given slot index, the value will be locked and governance will not be able to modify it anymore for that slot. 


## Withdrawal
Funds are recovered from Hermez network by executing two transactions back-to-back:
1. **Exit transaction**: Funds are transferred from the state tree to the exit tree.
2. **Withdrawal**: Funds are transferred from Hermez smart contract to the user Ethereum address. The limit and rate at which funds can be transferred from the smart contract is regulated by a [`leaky bucket`](https://en.wikipedia.org/wiki/Leaky_bucket) algorithm. Depending on the amount of available credits in the smart contract, withdrawal may be instantaneous or delayed.

### Hermez Withdrawal Limit
Withdrawals are classified in one of several buckets depending on the USD amount to be withdrawn. Every bucket contains some amount of credits indicating the maximum amount that can be withdrawn at any point in time. Buckets are re-filled with credits at a specific rate (depending on bucket). When a user attempts to withdraw funds, credits in the selected bucket are subtracted. If the withdrawal amount exceeds the existing value of credits, the instant withdrawal cannot be performed and a delayed withdrawal will be done instead. Delayed withdrawal is handled by the WithdrawalDelayer smart contract.

Figure below depicts how the different buckets are structured depending on the amount.

![](buckets.png)

### Withdrawal Resolution
The amount above the `withdrawal limit` set by the available credits wont be withdrawn instantly. In this case, excess tokens will be sent to the `WithdrawalDelayer` smart contract. 

The WithdrawalDelayer smart contract can be in one of two states:
1. **Normal Mode**: Amount above withdrawal limit is available for withdrawal, but with a delay D. This is the standard state.
2. **Emergency Mode**: The Hermez Foundation is the only body that may change the 
WithdrawalDelayer mode to Emergency in case of an attack. In this scenario, funds can only be 
withdrawn by the governance under the tutelage of an emergency council that will return the funds to the users. 

## Adding New Tokens
Hermez contains a list with all tokens supported. The following list includes some requirements on the token listing:
- Tokens must be ERC20
- Governance can decide the fee cost of adding tokens and therefore can regulate the token listing.
- There can be up to 2^{32} different tokens.
- Contracts maintain a list of all tokens registered in the rollup and each token needs to be listed before using it.
- The token 0 will be reserved for `ether`
- A token only can be added once

The list of supported tokens can be retrieved though the [`REST API`](../developers/api?=api) 
