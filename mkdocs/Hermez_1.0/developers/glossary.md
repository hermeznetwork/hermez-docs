# Glossary

## Auction
Selection of the coordinator is done via an auction process managed by a smart contract. The node with the highest bid in an open slot will earn the right to forge new batches and collect the fees from the transactions included in the batch. This auction is the process by which Hermez Network reaches a consensus on which node shall play the role of coordinator in an upcoming slot.

## BabyJubJub
BabyJubJub is an elliptic curve defined over a large prime field. It's useful in zk-SNARKs proofs.

##  Batch
A batch is a rollup block. It is formed by a set of transactions that determines a state transition of the Hermez accounts and sets an exit tree. Batches can be:
- L2-Batch: The set of transactions are only L2
- L1-L2 Batch: The set of transactions are L1 or L2

##  Coordinator
A coordinator is our term for rollup block producer. At any one time there is one coordinator responsible for collecting transactions and creating blocks on the rollup chain.

## Data Availability
Hermez approach determines that anyone can reconstruct the full tree state by just collecting data from the mainnet. This is done by not having any dependency of third parties holding essential data to reconstruct the full state. This feature ensures liveness of the system, meaning that no third party needs to be active in order to provide data to rebuild the state tree.

##  Forging
Forging refers to the creation of a batch of layer 2 transactions (off-chain), creation of the proof and the subsequent (on-chain) verification of the attached zk-SNARK.

## Governance
The Hermez Network community intends to follow a strategy of “Governance minimization”. This model is intended to initially be a bootstrap governance mechanism to adjust and manage some network parameters mainly for security and stability purposes until the network reaches a sufficient degree of maturity to become fully decentralized; at that stage, the initial bootstrap Governance model will no longer be necessary and will eventually disappear.

The network will start with a governance based on a Community Council formed by some distributed and known Ethereum community members. This council will delegate some specific network parameters adjustments into a reduced Bootstrap Council, which is non custodial,  in order to be more operationally effective in the initial phase.

Some decisions that the initial Community Council will be able to make will be:

- Governance and policies related changes
- Upgrade, maintenance and updates of the smart contracts code and/or circuits.

The bootstrap Council will be enabled to change some of the initial parameters of the Hermez smart contracts such as:

- Minimum bidding amount for the slots auction series;
- Days an auction is open for and slots before closing auction;
- Value of the outbidding variable;
- Boot Coordinator maximum cap reward reduction.

## HEZ
Hermez has its own network token: HEZ.

HEZ is an ERC-20 utility token used to place bids in the Coordinators auction. Every time a rollup batch is created, a fraction of HEZ tokens placed during the proof-of-donation auction will be burned, and therefore permanently removed.

## L1
Ethereum Layer-1 blockchain

## L2
Hermez Layer-2 blockchain 

## Proof of Donation
Bidding mechanism to select the coordinator for upcoming batches. A fraction of the winning bid goes back to be reinvested in the protocols and services that run on top of Ethereum. 

## System Parameters
Set of parameters defined in the system that allow certain configuration from governance in order to modify the behavior of the network. 

## Transactions
Transactions is the generic name given to every operation in the Hermez Network. Transactions may be initiated by a user or by the coordinator. Transactions may also happen on L1 or L2. The coordinator node is in charge of collecting and processing transactions in batches generating a ZK-SNARK to prove that the transactions have been carried out according to some rules.

### Atomic Transactions
Hermez provides the capability for some transactions to be processed together. This feature is called Atomic Transactions.

## Trees
Hermez uses Sparse Merkle Trees to store the state of the Hermez Network. There are two main tree structures:
- State Tree
- Exit Tree

### State Tree
Merkle tree used to represent the whole zkRollup state which is summarized by its root. 
Each leaf of the state tree represents an account, and contains data such us balance, ethereum Address or type of token stored in this account.

### Exit Tree
Each batch has an associated exit tree with all the exits performed by the user (either L1 or L2 exit transactions). 

User needs to prove that it owns a leaf in the exit tree in order to perform a withdrawal and get the tokens back. This verification could be done either by submitting a Merkle tree proof or by submitting a zkProof. 

## Exit & Withdrawal
In order to transfer funds from the L2 account to the Ethereum account, two separate transactions are invoked. The first transaction is Exit, where funds are transferred to a smart contract. The second transaction is Withdrawal. If conditions are met, Withdrawal can be instant. If funds to be withdrawn exceed certain limits, the Withdrawal is delayed until the transaction is cleared.

## zk-Rollup
A zk-Rollup is a layer 2 construction  which uses the Ethereum blockchain for data storage instead of computation. 
All funds are held by a smart contract on the main-chain. For every batch, a zk-snark is generated off-chain and verified by this contract.
This snark proves the validity of every transaction in the batch.

## zk-SNARK
A zk-SNARK is a short (and efficiently checkable) cryptographic proof that allows to prove something specific without revealing any extra information.

