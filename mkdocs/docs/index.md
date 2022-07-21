<div align="center">
<img src="logo-purple.png" align="center" width="128px"/>
<br /><br />
</div>

[![Chat on Twitter][ico-twitter]][link-twitter]
[![Chat on Telegram][ico-telegram]][link-telegram]
<!-- [![Website][ico-website]][link-website] -->
<!-- [![GitHub repo][ico-github]][link-github] -->
<!-- ![Issues](https://img.shields.io/github/issues-raw/hermeznetwork/zkevmdoc-public?color=blue) -->
<!-- ![GitHub top language](https://img.shields.io/github/languages/top/hermeznetwork/zkevmdoc-public) -->
<!-- ![Contributors](https://img.shields.io/github/contributors-anon/hermeznetwork/zkevmdoc-public) -->

[ico-twitter]: https://img.shields.io/twitter/url?color=blueviolet&label=Polygon%20Hermez&logoColor=blueviolet&style=social&url=https%3A%2F%2Ftwitter.com%2F0xPolygonHermez
[ico-telegram]: https://img.shields.io/badge/telegram-telegram-blueviolet
<!-- [ico-website]: https://img.shields.io/website?up_color=blueviolet&up_message=hermez.io&url=https%3A%2F%2Fhermez.io -->
<!-- [ico-github]: https://img.shields.io/github/last-commit/hermeznetwork/zkevmdoc-public?color=blueviolet -->

[link-twitter]: https://twitter.com/0xPolygonHermez
[link-telegram]: https://t.me/polygonhermez
<!-- [link-website]: https://hermez.io -->
<!-- [link-github]: https://github.com/hermeznetwork/zkevmdoc-public -->

---

# About Polygon zkEVM

Integrated seamlessly with the Ethereum ecosystem, Polygon zkEVM is a powerful, decentralized technology that provides Layer 2 scalability solutions to blockchain users. With the tremendous increase in the number of transactions taking place on-chain (i.e. base layer Ethereum), the Layer 1 solution is already facing blockchain trilemma: decentralization, scalability, and security. It is where Polygon zkEVM steps in. 

By providing zk-rollups (zero-knowledge rollups) that sit on the top of Ethereum Mainnet, the scalability and the transactions per second (TPS) can be dramatically improved. To prove that the off-chain computations are correct, Polygon zkEVM employs zero-knowledge proofs that can be verified easily. The Layer 2 zero-knowledge proofs are based on the complex polynomial computations that help in leveraging Ethereum scaling and provide fast finality to the off-chain transactions.

## What is Polygon zkEVM? 

Polygon Hermez 1.0 version was launched in March 2021. The goal of Polygon Hermez 1.0 was to scale payments and transfer ERC-20 tokens. It focussed mainly on decongesting Ethereum main chain by taking transactions off from the main chain and executing them off-chain; this resulted in an increase in the number of transactions that can be executed per second to up to 2000, which was a big improvement over Layer 1 Ethereum. See [Ethereum Live TPS](https://ethtps.info/Network/Ethereum) to keep track of Ethereum's live transactions per second. 

Polygon zkEVM, henceforth called zkEVM, has been developed to emulate Ethereum Virtual Machine(EVM) that executes Ethereum transactions with zero-knowledge proof validations. This has been accomplished by developing an EVM based on zero-knowledge; the machine is designed to recreate all the existing EVM opcodes that can be deployed as smart contracts. Although taking on this revolutionary design approach was a hard decision to make, the objective is to minimise the users and dApps friction when using the solution. It is an approach that requires the recreation of all EVM opcodes for the transparent deployment of existing Ethereum smart contracts. For this purpose, a new set of technologies and tools are being created and engineered by the team.

This document presents a high-level description of the upcoming Polygon zkEVM solution including its main components and design. It also seeks to highlight how Polygon zkEVM departs from the original design of Polygon Hermez 1.0.


## Architecture


Over and above what its predecessor was designed to do, the main functionality of zkEVM is to provide smart contract support. It performs the task of state transition resulting from the Ethereum Layer 2 transaction executions (transactions that users send to the network). Subsequently, by employing zero-knowledge functionality, it generates validity proofs that attest to the correctness of these state change computations carried out off-chain.

The major components of zkEVM are:

- Proof of Efficiency Consensus Mechanism
- zkNode Software
- zkProver
- LX-to-LY Bridge
- Sequencers
- Aggregators 
- Active users of the zkEVM network who create transactions.

The skeletal architecture of zkEVM is shown below: 


<p align="center"><img src="../images/fig1-simpl-arch.png" width="600" /></p>
<div align="center"><b> Figure 1 : Skeletal Overview of zkEVM </b></div>



### Consensus Algorithm: Proof of Efficiency

Our earlier version, Polygon Hermez 1.0, is based on the Proof of Donation(PoD) consensus mechanism. This model decides who would be the next batch creator. PoD is a decentralised auction that is conducted automatically and the participants (coordinators) bid a number of tokens so that they have the chance to create the next batch.

However, for the implementation of the current 2.0, PoD needed to be replaced with a much simpler Proof of Efficiency (PoE) model. Let us see why PoE is preferable to PoD.



#### Why is PoD not the Best Option?

The PoD model fell out of our preferable options for the reasons listed below:

- The PoD, being an auction model, has proved to be quite complex for both the coordinators and the validators. Besides, it has also proven to be less viable economically.  
- This consensus mechanism is vulnerable to attacks, especially during the bootstrapping phases. At any given point in time, the network is controlled by a permissionless participant. This raises the risk for the network to suffer service level delays should such a third party turn malicious or experience operational issues.
- PoD assigns the right to produce batches in a specific timeframe and validators need to be very competitive if they are to gain any economic incentives.
- The efficacy of selecting “the best” operator amounts to a "winner-takes-all" model, which turns out to be unfair to competitors with slightly less performance. Consequently, only a few select operators validate batches more often than others, defeating the concept of network decentralization.
- Another drawback is that the auction protocol is very costly and complex for validators. The auction requires bidding some time in advance.


#### Why is PoE a Better Model?

The Proof of Efficiency (PoE) model leverages the existing Proof of Donation mechanism and supports the permissionless participation of multiple coordinators to produce batches in Layer L2. These batches are created from the rolled-up transactions of Layer 1. As compared to PoD, PoE employs a much simpler mechanism and is preferred owing to its better efficiency to solve the problems inherent in PoD.   


The strategic implementation of PoE promises to ensure that the network: 

- Maintains its "permissionless" feature to produce L2 batches 
- Is efficient, a criterion which is key for the overall network performance
- Attains an acceptable degree of decentralization
- Is protected from malicious attacks, especially by validators
- Keeps a proportionate balance between the overall validation effort and the value in the network.

> Note: Possibilities of coupling PoE with a PoS (Proof of Stake) are currently being explored.

A detailed description of zkEVM's PoE is found [here](https://ethresear.ch/t/proof-of-efficiency-a-new-consensus-mechanism-for-zk-rollups/11988).


#### Hybrid Mode for On-Chain Data Availability

A full zk-rollup schema requires that both the data (which is required by users to reconstruct the full state) and the validity proofs (zero-knowledge proofs) be published on-chain. However, given the Ethereum setting, publishing data on-chain incurs gas fees, which is an already-existing problem with Layer 1. This makes it difficult to choose between a full zk-rollup configuration and a hybrid one. 

Under a hybrid schema, either of the following is possible:
 - **Validium**: Data is stored off-chain and only the validity proofs are published on-chain.
 - **Volition**: For some transactions, both the data and the validity proofs remain on-chain while for the remaining ones, only proofs go on-chain.

Unless, among other things, the proving module can be highly accelerated to mitigate costs for the validators, a hybrid schema remains viable. The team is yet to finalise the best consensus configuration.


### The PoE Smart Contract 

The underlying protocol in zkEVM ensures that the state transitions are correct by employing a validity proof. To ensure that a set of pre-determined rules have been followed for allowing transitioning of the state, a smart contract is employed. The verification of the validity proofs by a smart contract checks if each transition is done correctly. This is achieved by using zk-SNARK circuits. Such a mechanism entails two processes: batching of transactions and validation of the batched transactions. zkEVM uses two types of participants to carry out these processes: Sequencers and Aggregators. Under this two-layer model: 

- Sequencers propose transaction batches to the network, i.e. they roll-up the transaction requests to batches and add them to the PoE smart contract.

- Aggregators check the validity of the transaction batches and provide validity proofs. Any permissionless Aggregator can submit the proof to demonstrate the correctness of the state transition computation.

The PoE smart contract, therefore, makes two basic calls: A call to receive batches from Sequencers, and another call to Aggregators, requesting batches to be validated. See **Figure 2** below:

<p align="center"><img src="../images/fig2-simple-poe.png" width="650" /></p>
<div align="center"><b> Figure 2: Simplified Proof of Efficiency </b></div>



### Proof of Efficiency Tokenomics: Sequencers and Aggregators

The PoE smart contract imposes a few requirements on Sequencers and Aggregators.

**Sequencers**

A Sequencer receives L2 transactions from the users, preprocesses them as a new L2 batch, and then proposes the batch to the PoE smart contract as a valid L2 transaction.

- Anyone with the software necessary for running a zkEVM node can be a Sequencer. 
- Every Sequencer must pay a fee in form of MATIC tokens to earn the right to create and propose batches. 
- A Sequencer that proposes valid batches (which consist of valid transactions), is incentivised with the fee paid by transaction-requestors or the users of the network. 


**Aggregators**

An Aggregator receives all the transaction information from the Sequencer and sends it to the prover which provides a small zk-proof after complex polynomial computations. The smart contract validates this proof. This way, an aggregator collects the data, sends it to the prover, receives its output and finally, sends the information to the smart contract to check that the validity proof from the prover is correct. 

- An Aggregator's task is to provide validity proofs for the L2 transactions proposed by Sequencers.
- In addition to running zkEVM's zkNode software, Aggregators need to have specialised hardware for creating the zero-knowledge validity proofs. We, herein, call it the zkProver. (You will read about it later in this document).
- For a given batch or batches, an Aggregator that submits a validity proof first earns the Matic fee (which is being paid by the Sequencer(s) of the batch(es)).
- The Aggregators need to indicate their intention to validate transactions and then they compete to produce the validity proofs based on their own strategy.







### zkNode

A zkNode is the software needed to run a zkEVM node. It is a client that the network requires to implement the synchronization and govern the roles of the participants (Sequencers or Aggregators). 

#### zkNode Architecture

The zkNode Architecture is composed of:

1. **Sequencers and Aggregators**: Polygon zkEVM participants will choose how they participate; either as a node to know the state of the network; or as a participant in the process of batch production in any of the two roles: Sequencer or Aggregator. An Aggregator runs the zkNode but also performs validation using the core part of the zkEVM, called the zkProver (this is labelled **Prover** in **Figure 3** below.)

2. **Synchronizer**: Other than the sequencing and the validating processes, the zkNode also enables synchronisation of batches and their validity proofs, which happens only after these have been added to L1. This is accomplished using a subcomponent called the Synchronizer. A Synchronizer is in charge of getting all the data from smart contracts, which includes the data posted by the sequencers (transactions) and the data posted by the coordinators (which is the validity proof). All this data is stored in a huge database and served to third parties through a service called "JSON-RPC".
<br><br>The Synchronizer is responsible for reading the events from the Ethereum blockchain, including new batches to keep the state fully synced. The information read from these events must be stored in the database. The Synchronizer also handles possible reorgs, which will be detected by checking if the last `ethBlockNum` and the last `ethBlockHash` are synced.</br></br>


<p align="center"><img src="../images/fig3-zkNode-arch.png" width="600" /></p>
<div align="center"><b> Figure 3: zkEVM zkNode Diagram </b></div>

<br>

The architecture of zkNode is modular and implements a set of functions as depicted in **Figure 3** above.</br>

 3. **RPC**: RPC (Remote Procedure Call) is a JSON RPC interface compatible with Ethereum. For a software application to interact with the Ethereum blockchain (by reading blockchain data and/or sending transactions to the network), it must connect to an Ethereum node. RPC enables integration of the zkEVM with existing tools, such as Metamask, Etherscan and Infura. It adds transactions to the **Pool** and interacts with the **State** using read-only methods. 

4. **State**: A subcomponent that implements the Merkle Tree and connects to the DB backend. It checks integrity at the block level (information related to gas and block size, among others) and some transaction-related information (signatures, sufficient balance). State also stores smart contract code into the Merkle tree and processes transactions using the EVM.

5. **zkProver**: All the rules for a transaction to be valid are implemented and enforced in the zkProver. A zkProver performs complex mathematical computations in the form of polynomials and assembly language; these are then verified on a smart contract. Those rules could be seen as constraints that a transaction must accomplish in order to be able to modify the state tree or the exit tree. The zkProver is the most complex module; it required developing two new programming languages to implement the needed elements. Read below to know more about zkProver. 


### zkProver

zkEVM employs advanced zero-knowledge technology to create validity proofs. It uses a zero-knowledge prover (zkProver), which is intended to run on any server and is being engineered to be compatible with most consumer hardware. Every Aggregator will use this zkProver to validate batches and provide validity proofs. zkProver has its own detailed architecture which is outlined below. It consists of a Main State Machine Executor, a collection of secondary State Machines (each with its own executor), a STARK-proof builder, and a SNARK-proof builder. See **Figure 4** below for a simplified diagram of the zkEVM zkProver:

<p align="center"><img src="../images/fig4-zkProv-arch.png" width="650" /></p>
<div align="center"><b> Figure 4: A Simplified zkProver Diagram </b></div>

<br>

In a nutshell, the zkEVM expresses state changes in a polynomial form. Therefore, the constraints that each proposed batch must satisfy are, in fact, polynomial constraints or polynomial identities. That is, all the valid batches must satisfy certain polynomial constraints.</br>

#### zkProver Architecture


The zkNode Architecture is composed of:

1. **Main State Machine Executor**: The Main Executor handles the execution of the zkEVM. This is where EVM Bytecodes are interpreted using a new **zero-knowledge Assembly language** (or zkASM), specially developed by the team. The executor also sets up the polynomial constraints that every valid batch of transactions must satisfy. Another language, specially developed by the team, called  **Polynomial Identity Language** (or PIL), is used to encode all the polynomial constraints.

2. **Secondary State Machines**: Every computation required in proving the correctness of transactions is represented in the zkEVM as a state machine. The zkProver, being the most complex part of the whole project, consists of several state machines; from those carrying out bitwise functionalities (e.g., XORing, padding, etc.) to those performing hashing (e.g., Keccak, Poseidon), even to verifying signatures (e.g., ECDSA).
<br><br>The collection of the secondary state machines, therefore, refers to a collection of all state machines in the zkProver. It is not a subcomponent per se, but a collection of various executors for individual secondary state machines. The set of state machines are:</br>
- Binary SM
- Memory SM
- Storage SM 
- Poseidon SM
- Keccak SM 
- Arithmetic SM

See **Figure 5** below for dependencies among these SMs.

While some SMs use both zkASM and PIL, others rely only on one of these languages depending upon the specific operations each SM is responsible for.

<p align="center"><img src="../images/fig5-col-sm-zkprov.png" width="800" /></p>
<div align="center"><b> Figure 5: zkEVM State Machines </b></div>


3. **STARK Proof Builder**: STARK, which stands for "Scalable Transparent Argument of Knowledge", is a proof system that enables provers to produce verifiable proofs without the need for a trusted setup. A STARK Proof Builder refers to the subcomponent used to produce zero-knowledge STARK proofs, which are zk-proofs attesting to the fact that all the polynomial constraints are satisfied.<br><br>State machines generate polynomial constraints and zk-STARKs are used to prove that batches satisfy these constraints. In particular, zkProver utilises "Fast Reed-Solomon Interactive Oracle Proofs of Proximity (RS-IOPP)", colloquially called [FRI](https://drops.dagstuhl.de/opus/volltexte/2018/9018/pdf/LIPIcs-ICALP-2018-14.pdf), to facilitate fast zk-STARK proving.</br></br>


4. **SNARK Proof Builder**: SNARK, which stands for "Succinct Non-interactive Argument of Knowledge", is a proof system that produces verifiable proofs. Since STARK proofs are bigger than the SNARK proofs, zkEVM zkProver uses SNARK proofs to prove the correctness of these STARK proofs. Consequently, the SNARK proofs, which are much cheaper to verify on L1, are published as validity proofs.<br><br>The aim is to generate a [CIRCOM](https://www.techrxiv.org/articles/preprint/CIRCOM_A_Robust_and_Scalable_Language_for_Building_Complex_Zero-Knowledge_Circuits/19374986/1) circuit which can be used to generate or verify a SNARK proof. Whether a PLONK or a GROTH16 SNARK proof will be used for zkEVM is a question that needs to be discussed.</br></br> 


### The LX-to-LY Bridge

An LX-LY bridge is a smart contract that lets users transfer their assets between two layers, LX and LY. The L1-L2 in zkEVM is a decentralised bridge for secure deposits and withdrawal of assets; it is a combination of two smart contracts, one deployed on one chain and the second on the other.

 The L1 and L2 contracts in zkEVM are identical except for where each is deployed. **Bridge L1 Contract** is on the Ethereum mainnet in order to manage asset transfers between rollups, while **Bridge L2 Contract** is on a specific rollup and it is responsible for asset transfers between mainnet and the rollup (or rollups). Layer 2 Interoperability allows a native mechanism to migrate assets between different L2 networks. This solution is embedded in the bridge smart contract.



#### Bridge L1 Contract

Bridge L1 Contract carries out two operations: `bridge` and `claim`. The `bridge` operation transfers assets from one rollup to another, while the `claim` operation applies when the contract makes a claim from any rollup.

Bridge L1 Contract requires two Merkle trees in order to perform the above operations: `globalExitTree` and `mainnet exit tree`. The `globalExitTree` contains all the information of exit trees of all rollups, whereas the `mainnet exit tree` has information on transactions made by users who interact with the mainnet. A contract named the **global exit root manager L1** is responsible for managing exit roots across multiple networks. 

The exit tree structure is depicted in **Figure 6** below:


<p align="center"><img src="../images/fig6-exit-tr-strct.png" width="700" /></p>
<div align="center"><b> Figure 6: The Exit Tree Structure </b></div>



#### Bridge L2 Contract

Bridge L2 Contract is deployed on Layer L2 with ether on it. The ether will be set on the genesis in order to enable the minting/burning of the native ether.

Bridge L2 Contract also requires all the information of exit trees of all rollups contained in the `globalExitTree` Merkle tree. In this case, a smart contract named the **global exit root manager L2** is responsible for managing the exit roots across multiple networks.

> Note: When a batch is verified in the PoE smart contract in L1, the rollup exit root is updated in the global exit root manager L1. Bridge L2 Contract handles the rollup side of the `bridge` and the `claim` operations, as well as interacting with the `globalExitTree` and the `rollup exit tree`, mainly to update exit roots.


#### LX-to-LY Bridge

Typically, a bridge smart contract is an L2-to-L1 Bridge, but the zkEVM Bridge is more flexible and interoperable. It can function as a bridge between any two arbitrary Layer 2 chains, L2_A and L2_B, or between any Layer 2 (say L2_X) and L1 (Ethereum blockchain). It consequently allows asset transfers among multiple rollups. Hence the term "LX-to-LY Bridge".



## zkEVM Design Characteristics


The architectural details (for engineering and implementation) described in the sections above will help zkEVM attain its design goals. That would mean a network which is: permissionless, decentralized, secure, efficient and comes with verifiable block data.

Development efforts aim at **permissionless-ness**, that is, allowing anyone with the zkEVM software to participate in the network. For instance, the consensus algorithm will give everyone the opportunity to be a Sequencer or an Aggregator.

Data availability is most crucial for **decentralization**, where every user has sufficient data needed to rebuild the full state of a rollup. As discussed above, the team still has to decide on the best configuration for data availability. The aim is to ensure that there is no censorship and that no one party can control the network.

zkEVM was designed with **security** in mind. As an L2 solution, most of the security is inherited from Ethereum. Smart contracts will warrant that anyone who executes state changes must, firstly, do it correctly; secondly, create a proof that attests to the validity of a state change; and thirdly, avail validity proofs on-chain for verification.


### Efficiency and Overall Strategy

Efficiency is key to network performance. zkEVM, therefore, applies several implementation strategies to guarantee efficiency. A few of them are listed below:

1. The first strategy is to deploy PoE, which incentivizes the most efficient aggregators to participate in the proof generation process.

2. The second strategy is to carry out all computations off-chain while keeping only the necessary data and zk-proofs on-chain.

There are other strategies too that are implemented within specific components of the zkEVM system. For instance:

1. The way in which the bridge smart contract is implemented, such as settling accounts in a UTXO manner, by only using the Exit Tree Roots.

2. Utilisation of specialised cryptographic primitives within the zkProver in order to speed up computations and minimise proof sizes, as seen in:

   (a) Running a special zero-knowledge Assembly language (zkASM) for interpretation of byte codes

   (b) Using zero-knowledge tools such as zk-STARKs for proving purposes; these proofs are very fast though they are bigger in size. [See](https://docs.google.com/presentation/d/1gfB6WZMvM9mmDKofFibIgsyYShdf0RV_Y8TLz3k1Ls0/edit#slide=id.p). 

   So, instead of publishing the sizeable zk-STARK proofs as validity proofs, a zk-SNARK is used to attest to the correctness of the zk-STARK proofs. These zk-SNARKs are, in turn, published as the validity proofs to state changes. This helps in reducing the gas costs from 5M to 350K.


## Conclusion 


Given the EVM opcode compatibility, zkEVM is designed to process smart contracts seamlessly and verify state changes efficiently. It promises not only to be secure and efficient but also to accomplish competitive decentralization. In an effort to achieve high-speed proving and succinct proofs for quick verification, the team is focused on the optimization of the zkProver.

The team also leverages the synergies among the different Polygon teams that are also looking into zk-rollup solutions for achieving Ethereum scalability. Although development is still underway, it was important for this document to be released early so as to align with the goal of transparency set by the open-source projects, as well as keep the Polygon community of developers and users of Polygon Hermez 1.0 updated with the upcoming changes.

Our next step will be to develop a public testnet. Although it is difficult to set a fixed date for the same, our plan is to launch it by mid-2022.
