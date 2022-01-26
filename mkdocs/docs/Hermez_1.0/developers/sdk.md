# Examples
Some Golang and Javascript integration examples are provided as a reference. They can be found at:
- [Hermez Golang examples](https://github.com/hermeznetwork/hermez-go-sdk/tree/main/examples)
- [Hermez Javascript examples](https://github.com/hermeznetwork/hermezjs/tree/main/examples)

Additionally, [Hermez Mobile SDK example](https://github.com/hermeznetwork/hermez_flutter_sdk/tree/main/example) provides an example of how to import and use the Mobile SDK.

# SDK
A full Javascript SDK and a Flutter Plugin for Hermez Mobile SDK are provided as part of the tools for integration with Hermez Network.

HermezJS is an open-source SDK to interact with Hermez Rollup network.  It can be downloaded as an [npm package](https://www.npmjs.com/package/@hermeznetwork/hermezjs), or via [github](https://github.com/hermeznetwork/hermezjs).

[Hermez Flutter SDK](https://github.com/hermeznetwork/hermez_flutter_sdk) is a Flutter Plugin for Hermez Mobile SDK, and provides a cross-platform tool (iOS, Android) to communicate with the Hermez API and network. 

There is an additional [Golang SDK](https://github.com/hermeznetwork/hermez-go-sdk) to interact with Hermez using Golang.

## SDK How-To (Javascript)
In this tutorial we will walk through the process of using the SDK to:
1. [Installing Hermezjs](#install-hermezjs)
2. [Initializing Hermezjs](#initialization)
3. [Check registered tokens](#check-token-exists-in-hermez-network)
4. [Creating a wallet](#create-a-wallet)
5. [Making a deposit from Ethereum into the Hermez Network](#deposit-tokens-from-ethereum-into-hermez-network)
6. [Verifying the balance in a Hermez account](#verify-balance)
7. [Withdrawing funds back to Ethereum network](#withdrawing)
8. [Making transfers](#transfers)
9. [Verifying transaction status](#verifying-transaction-status)
10. [Authorizing the creation of Hermez accounts](#create-account-authorization)
11. [Internal accounts](#create-internal-accounts)


## Install Hermezjs
```bash
npm i @hermeznetwork/hermezjs
```

## Import modules
Load Hermezjs library

```js
const hermez = require('@hermeznetwork/hermezjs')
```

## Initialization
### Create Transaction Pool
Initialize the storage where user transactions are stored. This needs to be initialized at the start of your application.

```js
  hermez.TxPool.initializeTransactionPool()
```

### Configure Hermez Environment
In these examples, we are going to connect to `Hermez Testnet` which is deployed in Rinkeby Ethereum Network. To configure `Hermezjs` to work with the Testnet, we need to configure a Rinkeby Ethereum node, the Hermez API URL, and the addresses of the Hermez and Withdrawal Delayer smart contracts.

Hermez Testnet API URL is deployed at https://api.testnet.hermez.io/v1. 

>**NOTE:** In order to interact with Hermez Testnet, you will need to supply your own Rinkeby Ethereum node. You can check these links to help you set up a Rinkeby node (https://blog.infura.io/getting-started-with-infura-28e41844cc89, https://blog.infura.io/getting-started-with-infuras-ethereum-api).

Currently, Testnet Hermez smart contract is deployed at address `0x14a3b6f3328766c7421034e14472f5c14c5ba090` and Withdrawal Delayer contract is deployed at address `0x6ea0abf3ef52d24427043cad3ec26aa4f2c8e8fd`. These addresses could change in the future, so please check these addresses with a query of the [API](https://api.testnet.hermez.io/v1/config) using the browser.

For the remainder of the examples, we will configure the basic Hermezjs parameters

```js
const EXAMPLES_WEB3_URL = 'https://rinkeby.infura.io/v3/80496a41d0a134ccbc6e856ffd034696'
const EXAMPLES_HERMEZ_API_URL = 'https://api.testnet.hermez.io'
const EXAMPLES_HERMEZ_ROLLUP_ADDRESS = '0x14a3b6f3328766c7421034e14472f5c14c5ba090'
const EXAMPLES_HERMEZ_WDELAYER_ADDRESS = '0x6ea0abf3ef52d24427043cad3ec26aa4f2c8e8fd'

hermez.Providers.setProvider(EXAMPLES_WEB3_URL)
hermez.Environment.setEnvironment({
        baseApiUrl: EXAMPLES_HERMEZ_API_URL,
        contractAddresses: {
          [hermez.Constants.ContractNames.Hermez]: EXAMPLES_HERMEZ_ROLLUP_ADDRESS,
          [hermez.Constants.ContractNames.WithdrawalDelayer]: EXAMPLES_HERMEZ_WDELAYER_ADDRESS
        }
})

```

## Check token exists in Hermez Network
Before being able to operate on the Hermez Network, we must ensure that the token we want to operate with is listed. For that we make a call to the Hermez Coordinator API that will list all available tokens. All tokens in Hermez Network must be ERC20.

We can see there are 2 tokens registered. `ETH` will always be configured at index 0. The second token is `HEZ`. For the rest of the examples we will work with `ETH`. In the future, more tokens will be included in Hermez.

```js
  const token = await hermez.CoordinatorAPI.getTokens()
  const tokenERC20 = token.tokens[0]
  console.log(token)

>>>>
{
  tokens: [
    {
      itemId: 1,
      id: 0,
      ethereumBlockNum: 0,
      ethereumAddress: '0x0000000000000000000000000000000000000000',
      name: 'Ether',
      symbol: 'ETH',
      decimals: 18,
      USD: 1787,
      fiatUpdate: '2021-02-28T18:55:17.372008Z'
    },
    {
      itemId: 2,
      id: 1,
      ethereumBlockNum: 8153596,
      ethereumAddress: '0x2521bc90b4f5fb9a8d61278197e5ff5cdbc4fbf2',
      name: 'Hermez Network Token',
      symbol: 'HEZ',
      decimals: 18,
      USD: 5.365,
      fiatUpdate: '2021-02-28T18:55:17.386805Z'
    }
  ],
  pendingItems: 0

```

## Create a Wallet 
We can create a new Hermez wallet by providing the Ethereum private key of an Ethereum account. This wallet will store the Ethereum and Baby JubJub keys for the Hermez account. The Ethereum address is used to authorize L1 transactions, and the Baby JubJub key is used to authorize L2 transactions. We will create two wallets.

> **NOTE** You will need to supply two Rinkeby private keys to initialize both accounts. The keys provided here are invalid and are shown as an example.

```js
  const EXAMPLES_PRIVATE_KEY1 = 0x705d123e707e25fa37ca84461ac6eb83eb4921b65680cfdc594b60bea1bb4e52
  const EXAMPLES_PRIVATE_KEY2 = 0x3a9270c05ac169097808da4b02e8f9146be0f8a38cfad3dcfc0b398076381fdd

  // load first account
  const wallet = await hermez.HermezWallet.createWalletFromEtherAccount(EXAMPLES_WEB3_URL, { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY1 })
  const hermezWallet = wallet.hermezWallet
  const hermezEthereumAddress = wallet.hermezEthereumAddress

  // load second account
  const wallet2 = await hermez.HermezWallet.createWalletFromEtherAccount(EXAMPLES_WEB3_URL, { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY2 })
  const hermezWallet2 = wallet2.hermezWallet
  const hermezEthereumAddress2 = wallet2.hermezEthereumAddress

```


## Deposit Tokens from Ethereum into Hermez Network
Creating a Hermez account and depositing tokens is done simultaneously as an L1 transaction.  In this example we are going to deposit 1 `ETH` tokens into the newly created Hermez accounts. 

```js
  // set amount to deposit
  const amountDepositString = '1.0'
  const amountDeposit = hermez.Utils.getTokenAmountBigInt(amountDepositString, 18)
  const compressedDepositAmount = hermez.HermezCompressedAmount.compressAmount(amountDeposit)

  // perform deposit account 1
  await hermez.Tx.deposit(
    compressedDepositAmount,
    hermezEthereumAddress,
    tokenERC20,
    hermezWallet.publicKeyCompressedHex,
    { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY1 }
  )

  // perform deposit account 2
  await hermez.Tx.deposit(
    compressedDepositAmount,
    hermezEthereumAddress2,
    tokenERC20,
    hermezWallet2.publicKeyCompressedHex,
    { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY2 }
  )

```
Internally, the deposit funcion calls the Hermez smart contract to add the L1 transaction.

## Verify Balance
A token balance can be obtained by querying the API and passing the `hermezEthereumAddress` of the Hermez account.

```js
    // get sender account information
    const infoAccountSender = (await hermez.CoordinatorAPI.getAccounts(hermezEthereumAddress, [tokenERC20.id]))
      .accounts[0]

    // get receiver account information
    const infoAccountReceiver = (await hermez.CoordinatorAPI.getAccounts(hermezEthereumAddress2, [tokenERC20.id]))
      .accounts[0]

    console.log(infoAccountSender)
    console.log(infoAccountReceiver)

>>>>>
{
  accountIndex: 'hez:ETH:4253',
  balance: '1099600000000000000',
  bjj: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  hezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  itemId: 4342,
  nonce: 1,
  token: {
    USD: 1789,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}
{
  accountIndex: 'hez:ETH:4254',
  balance: '1097100000000000000',
  bjj: 'hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U',
  hezEthereumAddress: 'hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4',
  itemId: 4343,
  nonce: 6,
  token: {
    USD: 1789,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}

```
We can see that the field `accountIndex` is formed by the token symbol it holds and an index. A Hermez account can only hold one type of token.
Account indexes start at 256. Indexes 0-255 are reserved for internal use.
Note that the balances do not match with the ammount deposited of 1 `ETH` because accounts already existed in Hermez Network before the deposit, so we performed a `deposit on top` instead.

Alternatively, an account query can be filtered using the assigned `accountIndex`

```js
    const account1ByIdx = await hermez.CoordinatorAPI.getAccount(infoAccountSender.accountIndex)
    const account2ByIdx = await hermez.CoordinatorAPI.getAccount(infoAccountReceiver.accountIndex)

    console.log(account1ByIdx)
    console.log(account2ByIdx)

>>>>>

{
  accountIndex: 'hez:ETH:4253',
  balance: '1099600000000000000',
  bjj: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  hezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  itemId: 4342,
  nonce: 1,
  token: {
    USD: 1789,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}
{
  accountIndex: 'hez:ETH:4254',
  balance: '1097100000000000000',
  bjj: 'hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U',
  hezEthereumAddress: 'hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4',
  itemId: 4343,
  nonce: 6,
  token: {
    USD: 1789,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}

```
## Withdrawing
Withdrawing funds is a two step process:
1. Exit
2. Withdrawal

### Exit

The `Exit` transaction is used as a first step to retrieve the funds from `Hermez Network` back to Ethereum.
There are two types of `Exit` transactions:
- Normal Exit, referred as `Exit` from now on. This is a L2 transaction type.
- `Force Exit`, an L1 transaction type which has extended guarantees that will be processed by the Coordinator. We will
talk more about `Force Exit` [here](#force-exit)

The `Exit` is requested as follows:

```js
  // set amount to exit
  const amountExit = hermez.HermezCompressedAmount.compressAmount(hermez.Utils.getTokenAmountBigInt('1.0', 18))

  // set fee in transaction
  const state = await hermez.CoordinatorAPI.getState()
  const userFee = state.recommendedFee.existingAccount

  // generate L2 transaction
  const l2ExitTx = {
    type: 'Exit',
    from: infoAccountSender.accountIndex,
    amount: amountExit,
    fee: userFee
  }

  const exitResponse = await hermez.Tx.generateAndSendL2Tx(l2ExitTx, hermezWallet, infoAccountSender.token)
  console.log(exitResponse)

>>>>
{
  status: 200,
  id: '0x0257305cdc43060a754a5c2ea6b0e0f6e28735ea8e75d841ca4a7377aa099d91b7',
  nonce: 2
}

```

After submitting our `Exit` request to the Coordinator, we can check the status of the transaction by calling
the Coordinator's Transaction Pool. The Coordinator's transaction pool stores all those transactions 
that are waiting to be forged.

```js
  const txPool = await hermez.CoordinatorAPI.getPoolTransaction(exitResponse.id)
  console.log(txPool)

>>>>>
{
  amount: '1000000000000000000',
  fee: 204,
  fromAccountIndex: 'hez:ETH:4253',
  fromBJJ: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  fromHezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  id: '0x0257305cdc43060a754a5c2ea6b0e0f6e28735ea8e75d841ca4a7377aa099d91b7',
  info: null,
  nonce: 2,
  requestAmount: null,
  requestFee: null,
  requestFromAccountIndex: null,
  requestNonce: null,
  requestToAccountIndex: null,
  requestToBJJ: null,
  requestToHezEthereumAddress: null,
  requestTokenId: null,
  signature: '38f23d06826be8ea5a0893ee67f4ede885a831523c0c626c102edb05e1cf890e418b5820e3e6d4b530386d0bc84b3c3933d655527993ad77a55bb735d5a67c03',
  state: 'pend',
  timestamp: '2021-03-16T12:31:50.407428Z',
  toAccountIndex: 'hez:ETH:1',
  toBjj: null,
  toHezEthereumAddress: null,
  token: {
    USD: 1781.9,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  },
  type: 'Exit'
}


```
We can see the `state` field is set to `pend` (meaning pending). There are 4 possible states: 
1. **pend** : Pending
2. **fging** : Forging
3. **fged** : Forged
4. **invl** : Invalid

If we continue polling the Coordinator about the status of the transaction, the state will eventually be set to `fged`.


We can also query the Coordinator to check whether or not our transaction has been forged. `getHistoryTransaction` reports those transactions
that have been forged by the Coordinator.

```js
  const txExitConf = await hermez.CoordinatorAPI.getHistoryTransaction(txExitPool.id)
  console.log(txExitConf)

```

And we can confirm our account status and check that the correct amount has been transfered out of the account.

```js
  console.log((await hermez.CoordinatorAPI.getAccounts(hermezEthereumAddress, [tokenERC20.id]))
    .accounts[0])

```

### Withdrawing Funds from Hermez

After doing any type of `Exit` transaction, which moves the user's funds from their token account to a specific Exit Merkle tree, one needs to do a `Withdraw` of those funds to an Ethereum L1 account.
To do a `Withdraw` we need to indicate the `accountIndex` that includes the Ethereum address where the funds will be transferred, the amount and type of tokens, and some information
to verify the ownership of those tokens. Additionally, there is one boolean flag. If set to true, the `Withdraw` will be instantaneous.

```js
    const exitInfoN = (await hermez.CoordinatorAPI.getExits(infoAccountSender.hezEthereumAddress, true)).exits
    const exitInfo = exitInfoN[exitInfoN.length - 1]
    // set to perform instant withdraw
    const isInstant = true

    // perform withdraw
    await hermez.Tx.withdraw(
      exitInfo.balance,
      exitInfo.accountIndex,
      exitInfo.token,
      hermezWallet.publicKeyCompressedHex,
      exitInfo.batchNum,
      exitInfo.merkleProof.siblings,
      isInstant,
      { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY1 }
    )

```

The funds should now appear in the Ethereum account that made the withdrawal.

### Force Exit

This is the L1 equivalent of an Exit. With this option, the smart contract forces Coordinators to pick up L1 transactions before they pick up L2 transactions to ensure that L1 transactions will eventually be picked up.

This is a security measure. We don't expect users to need to make a Force Exit.

```js
  // set amount to force-exit
  const amountForceExit = hermez.HermezCompressedAmount.compressAmount(hermez.Utils.getTokenAmountBigInt('1.0', 18))

  // perform force-exit
  await hermez.Tx.forceExit(
    amountForceExit,
    infoAccountSender.accountIndex,
    tokenERC20,
    { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY1 }
  )
```

The last step to recover the funds will be to send a new `Withdraw` request to the smart contract as we did after the regular `Exit` request.

```js 
  ```js
    const exitInfoN = (await hermez.CoordinatorAPI.getExits(infoAccountSender.hezEthereumAddress, true)).exits
    const exitInfo = exitInfoN[exitInfoN.length - 1]
    // set to perform instant withdraw
    const isInstant = true

    // perform withdraw
    await hermez.Tx.withdraw(
      exitInfo.balance,
      exitInfo.accountIndex,
      exitInfo.token,
      hermezWallet.publicKeyCompressedHex,
      exitInfo.batchNum,
      exitInfo.merkleProof.siblings,
      isInstant,
      { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY1 }
    )

```

## Transfers

First, we compute the fees for the transaction. For this we consult the recommended fees from the Coordinator.

```js
  // fee computation
  const state = await hermez.CoordinatorAPI.getState()
  console.log(state.recommendedFee)

>>>>
{
  existingAccount: 96.34567219671051,
  createAccount: 192.69134439342102,
  createAccountInternal: 240.86418049177627
}

```

The returned fees are the suggested fees for different transactions:
- **existingAccount** : Make a transfer to an existing account
- **createAccount**   : Make a transfer to a non-existent account, and create a regular account
- **createAccountInternal** : Make a transfer to an non-existent account and create internal account

The fee amounts are given in USD. However, fees are payed in the token of the transaction. So, we need to do a conversion.

```js
  const usdTokenExchangeRate = tokenERC20.USD
  const fee = fees.existingAccount / usdTokenExchangeRate
```

Finally we make the final transfer transaction.

```js
  // set amount to transfer
  const amountTransfer = hermez.HermezCompressedAmount.compressAmount(hermez.Utils.getTokenAmountBigInt('1.0', 18))
  // generate L2 transaction
  const l2TxTransfer = {
    from: infoAccountSender.accountIndex,
    to: infoAccountReceiver.accountIndex,
    amount: amountTransfer,
    fee: fee
  }
  const transferResponse = await hermez.Tx.generateAndSendL2Tx(l2TxTransfer, hermezWallet, infoAccountSender.token)
  console.log(transferResponse)
 
>>>>>
{
  status: 200,
  id: '0x02e7c2c293173f21249058b1d71afd5b1f3c0de4f1a173bac9b9aa4a2d149483a2',
  nonce: 3
}

```
The result status 200 shows that transaction has been correctly received. Additionally, we receive the nonce matching the transaction we sent,
and an id that we can use to verify the status of the transaction either using `hermez.CoordinatorAPI.getHistoryTransaction()` or `hermez.CoordinatorAPI.getPoolTransaction()`.

As we saw with the `Exit` transaction, every transaction includes a ´nonce´. This `nonce` is a protection mechanism to avoid replay attacks. Every L2 transaction will increase the nonce by 1.

## Verifying Transaction Status
Transactions received by the Coordinator will be stored in its transaction pool while they haven't been processed. To check a transaction in the transaction pool we make a query to the Coordinator node.

```js
  const txXferPool = await hermez.CoordinatorAPI.getPoolTransaction(transferResponse.id)
  console.log(txXferPool)

>>>>>
{
  amount: '100000000000000',
  fee: 202,
  fromAccountIndex: 'hez:ETH:4253',
  fromBJJ: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  fromHezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  id: '0x02e7c2c293173f21249058b1d71afd5b1f3c0de4f1a173bac9b9aa4a2d149483a2',
  info: null,
  nonce: 3,
  requestAmount: null,
  requestFee: null,
  requestFromAccountIndex: null,
  requestNonce: null,
  requestToAccountIndex: null,
  requestToBJJ: null,
  requestToHezEthereumAddress: null,
  requestTokenId: null,
  signature: 'c9e1a61ce2c3c728c6ec970ae646b444a7ab9d30aa6015eb10fb729078c1302978fe9fb0419b4d944d4f11d83582043a48546dff7dda22de7c1e1da004cd5401',
  state: 'pend',
  timestamp: '2021-03-16T13:20:33.336469Z',
  toAccountIndex: 'hez:ETH:4254',
  toBjj: 'hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U',
  toHezEthereumAddress: 'hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4',
  token: {
    USD: 1786,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  },
  type: 'Transfer'
}


```

We can also check directly with the Coordinator in the database of forged transactions.

```js
  const transferConf = await hermez.CoordinatorAPI.getHistoryTransaction(transferResponse.id)
  console.log(transferConf)

>>>>>
{
  L1Info: null,
  L1orL2: 'L2',
  L2Info: { fee: 202, historicFeeUSD: 182.8352, nonce: 3 },
  amount: '100000000000000',
  batchNum: 4724,
  fromAccountIndex: 'hez:ETH:4253',
  fromBJJ: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  fromHezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  historicUSD: 0.17855,
  id: '0x02e7c2c293173f21249058b1d71afd5b1f3c0de4f1a173bac9b9aa4a2d149483a2',
  itemId: 14590,
  position: 1,
  timestamp: '2021-03-16T13:24:48Z',
  toAccountIndex: 'hez:ETH:4254',
  toBJJ: 'hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U',
  toHezEthereumAddress: 'hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4',
  token: {
    USD: 1787.2,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  },
  type: 'Transfer'
}

```

At this point, the balances in both accounts will be updated with the result of the transfer

```js 
  // check balances
  console.log((await hermez.CoordinatorAPI.getAccounts(wallet.hermezEthereumAddress, [tokenERC20.id])).accounts[0])
  console.log((await hermez.CoordinatorAPI.getAccounts(wallet2.hermezEthereumAddress2, [tokenERC20.id])).accounts[0])
>>>>>

{
  accountIndex: 'hez:ETH:4253',
  balance: '477700000000000000',
  bjj: 'hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO',
  hezEthereumAddress: 'hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6',
  itemId: 4342,
  nonce: 4,
  token: {
    USD: 1793,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}
{
  accountIndex: 'hez:ETH:256',
  balance: '1874280899837791518',
  bjj: 'hez:YN2DmRh0QgDrxz3NLDqH947W5oNys7YWqkxsQmFVeI_m',
  hezEthereumAddress: 'hez:0x9F255048EC1141831A28019e497F3f76e559356E',
  itemId: 1,
  nonce: 2,
  token: {
    USD: 1793,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-02-28T18:55:17.372008Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}

```
## Create Account Authorization
Imagine that Bob wants to send a transfer of Ether to Mary using Hermez, but Mary only has an Ethereum account but no Hermez account. To complete this transfer, Mary could open a Hermez account and proceed as the previous transfer example.
Alternatively, Mary could authorize the Coordinator to create a Hermez account on her behalf so that she can receive Bob's transfer. 

First we create a wallet for Mary:
```js
  // load second account
  const wallet3 = await hermez.HermezWallet.createWalletFromEtherAccount(EXAMPLES_WEB3_URL, { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY3 })
  const hermezWallet3 = wallet3.hermezWallet
  const hermezEthereumAddress3 = wallet3.hermezEthereumAddress
```

The authorization for the creation of a Hermez account is done using the private key stored in the newly created Hermez wallet. 
> Note that the account is not created at this moment. The account will be created when Bob performs the transfer. Also, it is Bob  that pays for the fees associated with the account creation.

```js
  const EXAMPLES_PRIVATE_KEY3 = '0x3d228fed4dc371f56b8f82f66ff17cd6bf1da7806d7eabb21810313dee819a53'
  const signature = await hermezWallet3.signCreateAccountAuthorization(EXAMPLES_WEB3_URL, { type: 'WALLET', privateKey: EXAMPLES_PRIVATE_KEY3 })
  const res = await hermez.CoordinatorAPI.postCreateAccountAuthorization(
    hermezWallet3.hermezEthereumAddress,
    hermezWallet3.publicKeyBase64,
    signature
  )
```

We can find out if the Coordinator has been authorized to create a Hermez account on behalf of a user by:

```js
  const authResponse = await hermez.CoordinatorAPI.getCreateAccountAuthorization(wallet3.hermezEthereumAddress)
  console.log(authResponse)

>>>>

{
  hezEthereumAddress: 'hez:0xd3B6DcfCA7Eb3207905Be27Ddfa69453625ffbf9',
  bjj: 'hez:ct0ml6FjdUN6uGUHZ70qOq5-58cZ19SJDeldMH021oOk',
  signature: '0x22ffc6f8d569a92c48a4e784a11a9e57b840fac21eaa7fedc9dc040c4a45d502744a35eeb0ab173234c0f687b252bd0364647bff8db270ffcdf1830257de28e41c',
  timestamp: '2021-03-16T14:56:05.295946Z'
}

```

Once we verify the receiving Ethereum account has authorized the creation of a Hermez account, we can proceed with the transfer from Bob's account to Mary's account. For this, we set the destination address to Mary's Ethereum address and set the fee using  the `createAccount` value.

```js
  // set amount to transfer
  const amountTransferAuth = hermez.HermezCompressedAmount.compressAmount(hermez.Utils.getTokenAmountBigInt('1.0', 18))
  // generate L2 transaction
  const l2AuthTxTransfer = {
    from: infoAccountSender.accountIndex,
    to: hermezWallet3.hermezEthereumAddress
    amount: amountTransferAuth,
    fee: state.recommendedFee.createAccount / usdTokenExchangeRate
  }
  const accountAuthTransferResponse = await hermez.Tx.generateAndSendL2Tx(l2AuthTxTransfer, hermezWallet, infoAccountSender.token)
  console.log(accountAuthTransferResponse)
 
>>>>>
{
  status: 200,
  id: '0x025398af5b69f132d8c2c5b7b225df1436baf7d1774a6b017a233bf273b4675c8f',
  nonce: 0
}

```

After the transfer has been forged, we can check Mary's account on Hermez
```js
    // get receiver account information
    const infoAccountAuth = (await hermez.CoordinatorAPI.getAccounts(hermezWallet3.hermezEthereumAddress, [tokenERC20.id]))
      .accounts[0]
    console.log(infoAccountAuth)

>>>>>

{
  accountIndex: 'hez:ETH:265',
  balance: '1000000000000000',
  bjj: 'hez:ct0ml6FjdUN6uGUHZ70qOq5-58cZ19SJDeldMH021oOk',
  hezEthereumAddress: 'hez:0xd3B6DcfCA7Eb3207905Be27Ddfa69453625ffbf9',
  itemId: 10,
  nonce: 0,
  token: {
    USD: 1795.94,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-03-16T14:56:57.460862Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}
```

## Create Internal Accounts
Until now we have seen that accounts have an Ethereum address and a Baby JubJub key. This is the case for normal accounts. However, there is a second type of account that only requires a Baby JubJub key. These accounts are called `internal accounts`.

The advantage of these accounts is that they are much more inexpensive to create than a `normal account`, since these accounts only exist on Hermez. The downside is that one cannot perform deposits or withdrawals from this type of account. However, there are some scenarios where these accounts are useful. For example, in those scenarios where one requires a temporary account. (for example, Exchanges could use these accounts to receive a transfer from users).


```js
  // Create Internal Account
  // create new bjj private key to receive user transactions
  const pvtBjjKey = Buffer.allocUnsafe(32).fill('1')

  // create rollup internal account from bjj private key
  const wallet4 = await hermez.HermezWallet.createWalletFromBjjPvtKey(pvtBjjKey)
  const hermezWallet4 = wallet4.hermezWallet

  // set amount to transfer
  const amountTransferInternal = hermez.HermezCompressedAmount.compressAmount(hermez.Utils.getTokenAmountBigInt('1.0', 18))
  // generate L2 transaction
  const transferToInternal = {
    from: infoAccountSender.accountIndex,
    to: hermezWallet4.publicKeyBase64,
    amount: amountTransferInternal,
    fee: state.recommendedFee.createAccountInternal / usdTokenExchangeRate
  }
  const internalAccountResponse = await hermez.Tx.generateAndSendL2Tx(transferToInternal, hermezWallet, tokenERC20)
  console.log(internalAccountResponse)


>>>>>

{
  status: 200,
  id: '0x02ac000f39eee60b198c85348443002991753de912337720b9ef85d48e9dcfe83e',
  nonce: 0
}
```

Once the transaction is forged, we can check the account information
```js
    // get internal account information
    const infoAccountInternal = (await hermez.CoordinatorAPI.getAccounts(hermezWallet4.publicKeyBase64, [tokenERC20.id]))
      .accounts[0]
    console.log(infoAccountInternal)


>>>>>>

{
  accountIndex: 'hez:ETH:259',
  balance: '1000000000000000000',
  bjj: 'hez:KmbnR34pOUhSaaPOkeWbaeZVjMqojfyYy8sYIHRSlaKx',
  hezEthereumAddress: 'hez:0xFFfFfFffFFfffFFfFFfFFFFFffFFFffffFfFFFfF',
  itemId: 4,
  nonce: 0,
  token: {
    USD: 1798.51,
    decimals: 18,
    ethereumAddress: '0x0000000000000000000000000000000000000000',
    ethereumBlockNum: 0,
    fiatUpdate: '2021-03-16T15:44:08.33507Z',
    id: 0,
    itemId: 1,
    name: 'Ether',
    symbol: 'ETH'
  }
}


```
We can verify it is in fact an `internal account` because the associated `hezEthereumAddress` is `hez:0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`.

