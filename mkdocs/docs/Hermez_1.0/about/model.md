# Hermez Network Model

## Hermez Protocol

Hermez provides the decentralized components in the form of smart contracts and open source tools (SDK, Block Explorer, Wallet,...) to enable a new 
ecosystem of actors to participate in the network. Hermez Network interacts with the rest of the ecosystem either via smart contracts or via a
 standardized REST API exported by all operative coordinators.

## Coordinators

Coordinators are Hermez Network's version of block producers. This means they are the ones who effectively
 run the network by computing the zero-knowledge proof of validity of the transactions made by the users.

Coordinators use systems infrastructure to:
* Synchronize with the Hermez Network  fetching data (historical and updates) from layer 1
* Receive transaction requests from users
* Process transaction requests in order to build rollup batches
* Save the  Merkle root together with a ZK-proof of correctness and the data necessary to reconstruct the full Merkle tree on Ethereum.

The boot coordinator is a special seed coordinator which will be assigned to create batches of transactions by default, until an alternative permissionless operator will place a bid in an attempt to obtain the right to run the network in a future slot of time.


### Auction
Multiple coordinators can coexist in the network running Hermez nodes.
They will compete in a continuous decentralized auction process managed by smart contract for the right to
 build rollup batches and collect the fees associated with each user transaction.  The right to forge is structured in time slots, which are defined to be 10 minutes long.

The winning bid is the one with the highest amount of HEZ, an ERC-20 utility token whose value is not pre-determined
 or pegged to a reference asset.

All bids will be processed by the auction smart contract , and all of the HEZ tokens placed will be used as follows:

- 30% will be burned (permanently removed).
- 40% will be automatically and permanently transferred to a donations account controlled by the Ethereum Foundation. These donations will be initially sent to Gitcoin quadratic funding grants but with the future-proof ability to donate also to other quadratic funding matching pools as they become available;
- 30% will be allocated as Hermez Network usage incentives, compensating active engagement and network adoption, e.g. rewarding transaction and rewarding the holding of specific tokens in Hermez L2 addresses, instead of on L1 Ethereum addresses.


The auction process incentivises efficiency, as coordinators need to include as many transactions as possible in each time slot to compensate for their bidding costs and operative expenses.

To prevent bidders from buying up all the slots in one go, nobody will be able to bid on a specific slot more than one month in advance. And the auction will be closed two time slots before the time of creating the slot.

The auction will be structured in a series of six time slots to cover one hour (%0-5), with 10 HEZ as the initial minimal bidding price for all of them.

The first bid in each time slot must be over the minimum bidding price in order to be accepted as valid. Thereafter, any bid placed in the auction should outbid the previous bid by at least 10%.


The minimum bidding amount for each slot in an auction series will be decided by the network governance (see section "Hermez Governance"), and it will be possible to change it dynamically, affecting future, even already open, auctions.

Also, governance will be able to implement the effective decentralisation of the network by locking the price of specific slots of the auction to 0 HEZ and therefore wouldn't require any minimum bid, this being an irreversible configuration.

## Users
Network users will be provided with easy-to-use interfaces to register their L1 Ethereum addresses as Hermez 
L2 accounts. They will then be able to deposit and withdraw their funds, ETH or ERC-20 tokens, into or out of these L2 accounts.

Initially, users will access the Hermez Network through an interface based on a non-custodial individually
 owned wallet solution that relies on Metamask for the management of private keys.

Through this interface, users will be able to:

- Register their Ethereum L1 address into the Hermez Network and obtain an internal address - one for each type of token they wish to deposit;
- Deposit L1 tokens into their Hermez Network addresses with a simple transaction;
- Transfer tokens between Hermez addresses much faster and for very low fees;
- Withdraw tokens from Hermez Network addresses back to their chosen L1 addresses.

Hermez provides protection mechanisms (enforced in the smart contract) that guarantee all tokens locked in 
the L2 solution can always be recovered by the users, even in the unlikely event that the auction-winning coordinator is malicious — in other words, stealing, censoring or blocking transactions is impossible.

The Hermez Network does not provide in any way custodial or exchange services. Hermez only and exclusively provides a L2 scaling solution for faster, low fee Ethereum tokens transfers.

## Third-Party Volume Aggregators
Third-Party Exchanges and other volume aggregators will use an SDK to connect to the network to access L1 smart contracts and coordinator REST API endpoints and thus exploit Hermez's full potential.
Hermez will provide an interesting feature of atomic transactions, which implements a link between transactions that need to be executed together, and it's very useful for token swaps.

