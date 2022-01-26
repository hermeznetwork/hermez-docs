#  Integrators

This FAQ is addressed to developers integrating Hermez Network into their service, such as exchanges.

## Overview & Getting Started 

### Where do I start?

The first place to start is the [Developer Guide](../developers/dev-guide.md) that provides an introduction to Hermez and the protocol. Additionally, there are some 
code examples in the [SDK section](../developers/sdk.md) and in the [Exchanges section](../users/exchanges.md) that may be useful to get a deeper understanding of Hermez.

### Is there an SDK available?

HermezJS is an open-source SDK to interact with Hermez Rollup network.  It can be downloaded as an [npm package](https://www.npmjs.com/package/@hermeznetwork/hermezjs), or via [github](https://github.com/hermeznetwork/hermezjs). 
Additionally, there are some examples of how to interact with Hermez written in Golang. You can find these examples [here](https://github.com/hermeznetwork/hermez-integration)

### What are the different account types in Hermez?

An account in Hermez is represented by a leaf in the Merkle tree. Each account can only store one token type. There are two account types:
- Normal accounts include a hezEthereum and a Baby Jubjub address. These accounts can be used to do deposits from L1, transfers within L2 and withdrawals to L1.
- Internal accounts include only a Baby Jubjub address. These accounts can be only be used to do transfers within L2.

### Is it possible to send a transfer to a non-existing Hermez account?

Yes, it is. The receiver of the transfer needs to have previously authorized the Coordinator to create the account at the moment of the transfer. This authorization is done by opening the account with the Hermez wallet.

### Do I need to run a Coordinator node?

You don't unless you want to. However, as an integrator offering some service on top of Hermez Network, you may want to spin a Hermez node in synchronizer mode to directly access the Hermez data directly without an intermediary. 

### How do I check the status of a transaction?

Whenever you send an L2 transaction to Hermez, it will be added to a transaction-pool queue. This transaction will remain there until it has been processed or expires. The possible states of a transaction in the 
transaction pool include `forged`, `forging`, `pending` and `invalid`.To check the status of a transaction, you can query the API using the returned transaction id by sending a GET /transactions-pool/{transaction-id}. 

### What happens if a transaction is not processed?

Coordinators select the transactions to process according to some internal rules configured by the Coordinator. If the transaction is not processed, it will expire and be removed from the transaction pool.


### What is the timeout for a transaction in the transaction pool?

Currently, this timeout is set to 24 hours.

### What are the reasons a transaction may not be processed?

A valid transaction should always be processed within 15 minutes. There are several reasons why a transaction may be invalid and therefore not processed by any Coordinator; insufficient balance in the sender account, nonexistent sender account, destination account hasn't given permission to create an account, fees lower than suggested Coordinator fees,... Checking the transaction status will provide some feedback on the reason why the transaction wasn't forged.

### Can I cancel a transaction in the pool?

Transactions cannot be cancelled once submitted. 

### How do I set the fee when sending a transaction?

Coordinators select the recommended fee depending on the different transaction types. These fees can be queried in the `/state` endpoint and are in USD. There are three types of fees:
- existingAccount used for transfers to existing Hermez accounts
- createAccount used for deposits or transfers to nonexistent normal accounts (`transferToEthAddress` transaction)
- creteAccountInternal used for transfers to nonexistent internal accounts (`transfertoBJJ` transaction)

These USD fees need to be converted to the equivalent fee in the token used for the transaction.

### How do I set the nonce when sending a transaction?

Each Hermez transaction includes a nonce value that prevents replay attacks. The Coordinator will forge transactions with the expected nonce (that is, the nonce of the transaction to be processed needs to be one more than the nonce of the last transaction processed from the same sender account). These nonces are computed by the SDK automatically unless specifically initialized. Recommended practice is to let the SDK compute these nonces.

## Troubleshooting 

### Where can I submit a bug report or contact Hermez for additional help?

As always, please report bugs to hello@hermez.network. Additionally, you can always contact us in [Discord](https://bit.ly/hermez-discord). You can also find more information in the [Hermez documentation page](https://docs.hermez.io)

