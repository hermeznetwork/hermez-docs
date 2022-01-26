# Hermez Wallet Guide

### Welcome to the Hermez Wallet

Hermez Wallet provides a simple user interface to get started with the Hermez Network. It supports depositing, transferring, and withdrawing ETH and ERC-20 tokens on Hermez Network. 

## Getting Started 

When opening the wallet, there's a button to log in with Metamask. This will automatically derive a Hermez account from the Ethereum account.

![](wallet/hw-login.png)

This will then lead to an empty wallet:

![](wallet/hw-empty.png)

The next step is to make a deposit

## Transactions

There are 3 kinds of transactions:

- Deposits. Sends ETH or an ERC-20 token (must be registered in Hermez) from your Ethereum account to your Hermez account.
- Transfers. Sends ETH or an ERC-20 token from a Hermez account to another Hermez account.
- Withdrawals. Sends ETH or an ERC-20 token from a Hermez account to its corresponding Ethereum account.

They all follow a similar flow. First, we select a token. If it's a `Deposit`, the token must be in the Ethereum account. Otherwise, it must be in the Hermez account.

![](wallet/hw-deposit-accounts.png)

Then there's a form to select the amount:

![](wallet/hw-deposit-form.png)

In the case of a `Transfer`, there will also be a **receiver** input. This input also supports scanning a QR code or pasting the Receiver's address directly.

![](wallet/hw-tx-form.png)

If everything is valid, the next step is the confirmation with a look at all of the transaction parameters. With a `Deposit`, as it is a Layer 1 transaction, it will require signing with your Ethereum Wallet (e.g. Metamask).

This leads to the confirmation screen if everything went well:

![](wallet/hw-deposit-confirm)

## Accounts

Making `Deposits` creates accounts that now appear on the home screen.

![](wallet/hw-home.png)

Opening an account shows all the transactions related to that account.

![](wallet/hw-account.png)

Opening a transaction shows information related to that transaction. There's also a button to open the Batch Explorer with all the information.

![](wallet/hw-tx.png)

## Withdrawals

Withdrawals are a two-part process. The first part requires you to select the Token account you want to withdraw from (for example HEZ) and on the next screen click on the `Withdraw` button, enter the amount to withdraw, and click on `Continue`.

After completing the first part explained above, a card with the withdrawal details appears on the Home screen or on the respective account page. When ready, it will show a button to finalize the withdrawal.

*Withdrawals require paying an Ether gas fee on L1, insufficient gas in your L1 account will cause the withdrawal to stall. 

*Withdrawals are final and cannot be stopped, reversed, or altered in any way after initiated.  

![](wallet/hw-withdraw.png)

Alternatively, you can transfer your funds to a different account in Hermez and perform the withdrawal from that account. This may be helpful in situations where you don't have enough Ether in the original L1 account but do in a different account.

## My Account

There's a My Account page. Current options are:

- Copying your Hermez Address or displaying it as a QR code.
- Changing the default FIAT currency between EUR and USD.
- Making a `Force Withdrawal`. This is a L1 equivalent of the first step of the withdrawal as explained above. This uses more Gas but forces the Coordinator to pick the transaction up. It's only a security measure and shouldn't be needed.
- View the Hermez account in the Batch Explorer.
- Logging out.

![](wallet/hw-settings.png)
