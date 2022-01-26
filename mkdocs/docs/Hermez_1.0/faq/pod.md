# Proof-of-donation

> Hermez is currently under development. Some of the details in the answers can be modified before network launch.

## How exactly does proof-of-donation work?

We have an auction where everyone bids the amount of Hermez network tokens (HEZ) they're willing to donate in order to obtain the right to create the next block.

The winning bid is the highest amount of HEZ. And this address is assigned the right to create the next block.

We refer to this mechanism as **proof-of-donation** because **40%** of this bid goes back to be reinvested in the protocols and services that run on top of Ethereum.

For more on the details how it works, see this [ethresearch post](https://ethresear.ch/t/spam-resistant-block-creator-selection-via-burn-auction/5851) (though you should replace all instances of burn with donation when reading).

## Where are the funds from the proof-of-donation sent?

They will be sent initially to the Gitcoin quadratic funding pool, but with future governance, other funding pools might be enabled as they become available.
