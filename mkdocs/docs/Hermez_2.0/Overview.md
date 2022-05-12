# About Polygon Hermez

Integrated seamlessly with the Ethereum ecosystem, Polygon Hermez is a powerful, decentralized technology that provides Layer 2 scalability solutions to blockchain users. With the tremendous increase in the number of transactions taking place on-chain (i.e. base layer Ethereum), the Layer 1 solution is already facing throughput, scalability, and speed issues. It is where Polygon Hermez steps in. 

By providing zk-rollups (zero-knowledge rollups) that sit on the top of Ethereum Maninnet, the scalability and the transactions per second (TPS) can be dramatically improved. To prove that the off-chain computations are correct, Polygon Hermez employs zero-knowledge proofs that can be easily verified. The Layer 2 zero-knowledge proofs are based on the complex polynomial computations that help in leveraging Ethereum scaling and provide fast finality to the off-chain transactions.

## What is Polygon Hermez 2.0? 

Polygon Hermez 1.0 version was launched in March 2021. The main focus of Polygon Hermez 1.0 was to scale payments and transfer ERC-20 tokens. It focussed mainly on decongesting Ethereum main chain by taking transactions off from the main chain and executing them off-chain; this resulted in a considerable increase in the number of transactions to be executed per second to 2000, which was a big improvement over Layer 1 Ethereum. See [Ethereum Live TPS](https://ethtps.info/Network/Ethereum) for viewing Ethereum's live TPS. 

Polygon Hermez 2.0, henceforth called Hermez 2.0, has been developed to emulate Ethereum Virtual Machine(EVM), a machine that executes Ethereum transactions with zero-knowledge proof validations. This has been accomplished by developing zero-knowledge EVM, which has been designed to recreate all the existing EVM opcodes so that these can be deployed as smart contracts. Although taking on this revolutionary design approach was a hard decision to make, the objective is to minimise the users and dApps friction when using the solution. It is an approach that requires the recreation of all EVM opcodes for the transparent deployment of existing Ethereum smart contracts. For this purpose, a new set of technologies and tools are being created and engineered.

This document presents a high-level description of the upcoming Polygon Hermez 2.0 solution, which includes its main components and design. It also seeks to highlight how Polygon Hermez 2.0 departs from the original design of Hermez 1.0.


## Architecture


Over and above what its predecessor was designed to do, the main functionality of Hermez 2.0 is to provide smart contract support. It performs the task of state transition resulting from the Ethereum Layer 2 transaction executions (transactions that users send to the network). Subsequently, by employing zero-knowledge functionality, it generates validity proofs that attest to the correctness of these state change computations carried out off-chain.

The major components of Hermez 2.0 are:

- Proof of Efficiency Consensus Mechanism
- zkNode Software
- zkProver
- LX-to-LY Bridge
- Sequencers
- Aggregators 
- Active users of the Hermez 2.0 network who create transactions.

The skeletal architecture of Hermez 2.0 is therefore as follows.


<p align="center"><img src="IMAGES/fig1-simpl-arch.png" width="600" /></p>
<div align="center"><b> Figure 1 : Skeletal Overview Hermez 2.0 </b></div>



### Consensus Algorithm: Proof of Efficiency

Our earlier version, Hermez 1.0, is based on the Proof of Donation(PoD) consensus mechanism. This model decides who would be the next batch creator. PoD is a decentralised auction that is conducted automatically and the participants (coordinators) bid a number of tokens so that they have the chance to create the next batch.

However, for the implementation of the current 2.0, PoD needed to be replaced with a much simpler Proof of Efficiency (PoE) model. Let us see why PoE is preferable to PoD.



#### Why is PoD not the Best Option?

The PoD model fell out of our preferable options for the reasons listed below:

- The PoD, being an auction model, has proved to be quite complex for both the coordinators and the validators. Besides, it has also proven to be less viable economically. More so considering that not every competing validator gets rewarded but only the most effective.
- This consensus mechanism is vulnerable to attacks, especially during the bootstrapping phases. At any given point in time, the network is controlled by a permissionless participant. This raises the risk for the network to suffer service level delays should such a third party turn malicious or experience operational issues.
- PoD assigns the right to produce batches in a specific timeframe and validators need to be very competitive if they are to gain any economic incentives.
- The efficacy of selecting “the best” operator amounts to a winner-takes-all model, which turns out to be unfair to competitors with slightly less performance. Consequently, only a few select operators validate batches more often than others, defeating the concept of network decentralization.
- Another drawback is that the auction protocol is very costly and complex for validators, while at the same time only the most effective will be rewarded. The auction requires bidding some time in advance.


#### Why is PoE a Better Model?

The Proof of Efficiency (PoE) model leverages the existing Proof of Donation mechanism and supports the permissionless participation of multiple coordinators to produce batches in Layer L2. These batches are created from the rolled-up transactions of Layer 1. As compared to PoD, PoE employs a much simpler mechanism and is preferred owing to its better efficiency to solve the problems inherent in PoD.   


The strategic implementation of PoE promises to ensure that the network: 

- Maintains its "permissionless" feature to produce L2 batches 
- Is efficient, a criterion which is key for the overall network performance
- Attains an acceptable degree of decentralization
- Is protected from malicious attacks, especially by validators
- Keeps a proportionate balance between the overall validation effort and the value in the network.

> Note: Possibilities of coupling PoE with a PoS (Proof of Stake) are being explored.

A detailed description of Hermez 2.0's PoE is found [here](https://ethresear.ch/t/proof-of-efficiency-a-new-consensus-mechanism-for-zk-rollups/11988).


#### Hybrid Mode for On-Chain Data Availability

A full zk-rollup schema requires that both the data (which is required by users to reconstruct the full state) and the validity proofs (zero-knowledge proofs) be published on-chain. However, given the Ethereum setting, publishing data on-chain incurs gas fees, which is an already-existing problem with Layer 1. This makes it difficult to choose between a full zk-rollup configuration and a hybrid one. 

Under a hybrid schema, either of the following is possible:
 - **Validium**: Data is stored off-chain and only the validity proofs are published on-chain.
 - **Volition**: For some transactions, both the data and the validity proofs remain on-chain while for the remaining ones, only proofs go on-chain.

Unless, among other things, the proving module can be highly accelerated to mitigate costs for the validators, a hybrid schema remains viable. The team is yet to finalise the best consensus configuration.


### The PoE Smart Contract 

The underlying protocol in Hermez 2.0 ensures that the state transitions are correct by employing a validity proof. To ensure that a set of pre-determined rules have been followed for allowing transitioning of the state, a smart contract is employed. The verification of the validity proofs by a smart contract checks if each transition is done correctly. This is achieved by using zk-SNARK circuits. Such a mechanism entails two processes: batching of transactions and validation of the batched transactions. Hermez 2.0 uses two types of participants to carry out these processes: Sequencers and Aggregators. Under this two-layer model, 

- Sequencers propose transaction batches to the network, i.e. they roll-up the transaction requests to batches and add them to the PoE smart contract.

- Aggregators check the validity of the transaction batches and provide validity proofs. Any permissionless Aggregator can submit the proof to demonstrate the correctness of the state transition computation.

The PoE smart contract, therefore, makes two basic calls: A call to receive batches from Sequencers, and another call to Aggregators, requesting batches to be validated. See **Figure 2** below.



### Proof of Efficiency Tokenomics: Sequencers and Aggregators

The PoE smart contract imposes a few requirements on Sequencers and Aggregators.

**Sequencers**

A Sequencer receives L2 transactions from the users, preprocesses them as a new L2 batch, and then proposes the batch to the PoE smart contract as a valid L2 transaction.

- Anyone with the software necessary for running a Hermez 2.0 node can be a Sequencer. 
- Every Sequencer must pay a fee in form of MATIC tokens to earn the right to create and propose batches. 
- A Sequencer that proposes valid batches (which consist of valid transactions), is incentivised with fees paid by transaction-requestors or the users of the network. 


**Aggregators**

An Aggregator receives all the transaction information from the Sequencer and sends it to the prover which provides a small zk-proof after complex polynomial computations. The smart contract validates this proof. This way, an aggregator collects the data, sends it to the prover, receives its output and finally, sends the information to the smart contract to check that the validity proof from the prover is correct. 

- An Aggregator's task is to provide validity proofs for the L2 transactions proposed by Sequencers.
- In addition to running Hermez 2.0's zkNode software, Aggregators need to have specialised hardware for creating the zero-knowledge validity proofs. We herein call it the zkProver (You will read about it in the later sections).
- For a given batch or batches, an Aggregator that submits a validity proof first earns the Matic fee (which is being paid by the Sequencer(s) of the batch(es)).
- The Aggregators need to indicate their intention to validate transactions and then they compete to produce the validity proofs based on their own strategy.



<p align="center"><img src="IMAGES/fig2-simple-poe.png" width="650" /></p>
<div align="center"><b> Figure 2: Simplified Proof of Efficiency </b></div>



### zkNode

A zkNode is the software needed to run a Hermez 2.0 node. It is a client that the network requires to implement the synchronization and governs the roles of participants (Sequencers or Aggregators). 

#### zkNode Architecture

The zkNode Architecture is composed of:

1. **Sequencers and Aggregators**: Polygon Hermez 2.0 participants will choose how they participate; either as simply a node, to know the state of the network; or as a participant in the process of batch production in any of the two roles: Sequencer or Aggregator. An Aggregator will be running the zkNode but also performs validation using the core part of the zkEVM, called the zkProver (this is labelled **Prover** in Figure 3 below.)

2. **Synchronizer**: Other than the sequencing and the validating processes, the zkNode also enables synchronisation of batches and their validity proofs, which happens only after these have been added to L1. It uses a subcomponent called the Synchronizer. A Synchronizer is in charge of getting all the data from smart contracts, which includes the data posted by the sequencers (transactions) and the data posted by the coordinators (which is the validity proof). All this data is stored in a huge database and served to third parties through a service called "JSON-RPC".

The Synchronizer is responsible for reading the events from the Ethereum blockchain, including new batches to keep the state fully synced. The information read from these events must be stored in the database. The Synchronizer also handles possible reorgs, which will be detected by checking if the last `ethBlockNum` and the last `ethBlockHash` are synced.


<p align="center"><img src="IMAGES/fig3-zkNode-arch.png" width="600" /></p>
<div align="center"><b> Figure 3 : Hermez 2.0 zkNode Diagram </b></div>


The architecture of zkNode is modular and implements a set of functions as depicted in Figure 3 above.

 3. **RPC**: (Remote Procedure Calls) is a JSON RPC interface compatible with Ethereum. For a software application to interact with the Ethereum blockchain (by reading blockchain data and/or sending transactions to the network), it must connect to an Ethereum node. RPC enables integration of the zkEVM with existing tools, such as Metamask, Etherscan and Infura. It adds transactions to the **Pool** and interacts with the **State** using read-only methods. 

4. **State**: A subcomponent that implements the Merkle Tree and connects to the DB backend. It checks integrity at the block level (i.e., information related to; gas, block size, etc.) and some transaction-related information (e.g., signatures, sufficient balance, etc.). State also stores smart contract (SC) code into the Merkle tree and processes transactions using the EVM.

5. **zkProver**: All the rules a transaction must follow to be valid are implemented and enforced in the zkProver. A zkProver performs complex mathematical computations in the form of polynomials and assembly language, which are then verified on Smart Contract. Those rules could be seen as constraints that a transaction must accomplish in order to be able to modify the state tree or the exit tree. The zkProver is the most complex module. It was necessary to develop two new programming languages to implement the needed elements. Read below to know more about zkProver. 


### zkProver

Hermez 2.0 employs advanced zero-knowledge technology to create validity proofs. It uses a zero-knowledge prover (zkProver), which is intended to run on any server and is being engineered to be compatible with most consumer hardware. Every Aggregator will use this zkProver to validate batches and provide validity proofs. zkProver has its own detailed architecture which is outlined below. It consists of a Main State Machine Executor, a collection of secondary State Machines, each with its own executor, a STARK-proof builder, and a SNARK-proof builder. See **Figure 4** below for a simplified diagram of the Hermez 2.0 zkProver.

<p align="center"><img src="IMAGES/fig4-zkProv-arch.png" width="650" /></p>
<div align="center"><b> Figure 4: A Simplified zkProver Diagram </b></div>



In a nutshell, the zkEVM expresses state changes in polynomial form. Therefore, the constraints that each proposed batch must satisfy are, in fact, polynomial constraints or polynomial identities. That is, all the valid batches must satisfy certain polynomial constraints.

#### zkProver Architecture


The zkNode Architecture is composed of:

1. **Main State Machine Executor**: The Main Executor handles the execution of the zkEVM. This is where EVM Bytecodes are interpreted using a new **zero-knowledge Assembly language** (or zkASM), specially developed by the team. The executor also sets up the polynomial constraints that every valid batch of transactions must satisfy. Another new language, also a specially developed language by the team, called the **Polynomial Identity Language** (or PIL) is used to encode all the polynomial constraints.

2. **Secondary State Machines**: Every computation required in proving the correctness of transactions is represented in the zkEVM as a state machine. The zkProver, being the most complex part of the whole project, consists of several state machines; from those carrying out bitwise functionalities (e.g., XORing, padding, etc.) to those performing hashing (e.g., Keccak, Poseidon), even to verifying signatures (e.g., ECDSA).

The collection of the secondary state machines, therefore, refers to a collection of all state machines in the zkProver. It is not a subcomponent per se, but a collection of various executors for individual secondary state machines. The set of state machines are: 

- Binary SM
- Memory SM
- Storage SM 
- Poseidon SM
- Keccak SM 
- Arithmetic SM

See **Figure 5** below for dependencies among these SMs.

Depending on the specific operations each SM is responsible for, some use both zkASM and PIL, while others rely only on one of these languages.

<p align="center"><img src="IMAGES/fig5-col-sm-zkprov.png" width="800" /></p>
<div align="center"><b> Figure 5: Hermez 2.0 State Machines </b></div>


3. **STARK Proof Builder**: STARK, which stands for "Scalable Transparent Argument of Knowledge", is a proof system that enables provers to produce verifiable proofs, without the need for a trusted setup. A STARK-proof Builder refers to the subcomponent used to produce zero-knowledge STARK-proofs, which are zk-proofs attesting to the fact that all the polynomial constraints are satisfied.



<div align="left">State machines generate polynomial constraints, and zk-STARKs are used to prove that batches satisfy these constraints. In particular, zkProver utilises Fast Reed-Solomon Interactive Oracle Proofs of Proximity (RS-IOPP), colloquially called [FRI](https://drops.dagstuhl.de/opus/volltexte/2018/9018/pdf/LIPIcs-ICALP-2018-14.pdf), to facilitate fast zk-STARK proving.</div>


4. **SNARK Proof Builder**: SNARK, which stands for "Succinct Non-interactive Argument of Knowledge", is a proof system that produces verifiable proofs. Since STARK proofs are bigger than SNARK proofs, Hermez 2.0 zkProver uses SNARK proofs to prove the correctness of these STARK proofs. Consequently, the SNARK proofs, which are much cheaper to verify on L1, are published as validity proofs.

The aim is to generate a [CIRCOM](https://www.techrxiv.org/articles/preprint/CIRCOM_A_Robust_and_Scalable_Language_for_Building_Complex_Zero-Knowledge_Circuits/19374986/1) circuit which can be used to generate or verify a SNARK proof. As to whether a PLONK or a GROTH16 SNARK proof will be used for Hermez 2.0, is a question that needs to be discussed. 


### The LX-to-LY Bridge

An LX-LY bridge is a smart contract that lets users transfer their assets between two layers, LX and LY. The L1-L2 in Hermez 2.0 is a decentralised bridge for secure deposits and withdrawal of assets; it is a combination of two smart contracts, one deployed on one chain and the second on the other.

 The L1 and L2 contracts in Hermez 2.0 are identical except for where each is deployed. **Bridge L1 Contract** is on the Ethereum mainnet in order to manage asset transfers between rollups, while **Bridge L2 Contract** is on a specific rollup and it is responsible for asset transfers between mainnet and the rollup (or rollups). Layer 2 Interoperability allows a native mechanism to migrate assets between different L2 networks. This solution is embedded in the Bridge Smart Contract.



#### Bridge L1 Contract

Bridge L1 Contract carries out two operations: `bridge` and `claim`. The `bridge` operation transfers assets from one rollup to another, while the `claim` operation applies when the contract makes a claim from any rollup.

Bridge L1 Contract requires two Merkle trees in order to perform the above operations: `globalExitTree` and `mainnet exit tree`. The `globalExitTree` contains all the information of exit trees of all rollups, whereas the `mainnet exit tree` has information on transactions made by users who interact with the mainnet. A contract named the **global exit root manager L1** is responsible for managing exit roots across multiple networks. 

The exit tree structure is depicted in **Figure 6** below:


<p align="center"><img src="IMAGES/fig6-exit-tr-strct.png" width="700" /></p>
<div align="center"><b> Figure 6: The Exit Tree Structure </b></div>



#### Bridge L2 Contract

Bridge L2 Contract is deployed on Layer L2 with ether on it. The ether will be set on the genesis in order to enable the minting/burning of the native ether.

Bridge L2 Contract also requires all the information of exit trees of all rollups contained in the `globalExitTree` Merkle tree. In this case, a smart contract named the **global exit root manager L2** is responsible for managing the exit roots across multiple networks.

> Note: When a batch is verified in the PoE smart contract in L1, the rollup exit root is updated in the global exit root manager L1. Bridge L2 Contract handles the rollup side of the `bridge` and the `claim` operations, as well as interacting with the `globalExitTree` and the `rollup exit tree`, mainly to update exit roots.


#### LX-to-LY Bridge

Typically, a bridge smart contract is an L2-to-L1 Bridge, but the Hermez 2.0 Bridge is more flexible and interoperable. It can function as a bridge between any two arbitrary Layer 2 chains, L2_A and L2_B, or between any Layer 2 (say L2_X) and L1 (Ethereum blockchain). It consequently allows asset transfers among multiple rollups. Hence the term "LX-to-LY Bridge".



## Hermez 2.0 Design Characteristics


The architectural details (for engineering and implementation) described in the sections above will help Hermez 2.0 attain its design goals. That would mean a network which is: permissionless, decentralized, secure, efficient and comes with verifiable block data.

Development efforts aim at **permissionless-ness**, that is, allowing anyone with the Hermez 2.0 software to participate in the network. For instance, the consensus algorithm will give everyone the opportunity to be a Sequencer or an Aggregator.

Data availability is most crucial for **decentralization**, where every user has sufficient data needed to rebuild the full state of a rollup. As discussed above, the team still has to decide on the best configuration for data availability. The aim is to ensure that there is no censorship and that no one party can control the network.

Hermez 2.0 was designed with **security** in mind. As an L2 solution, most of the security is inherited from Ethereum. Smart contracts will warrant that anyone who executes state changes must, firstly, do it correctly; secondly, create a proof that attests to the validity of a state change; and thirdly, avail validity proofs on-chain for verification.


### Efficiency and Overall Strategy

Efficiency is key to network performance. Hermez 2.0, therefore, applies several implementation strategies to guarantee efficiency. A few of them are listed below:

1. The first strategy is to deploy PoE, which incentivizes the most efficient aggregators to participate in the proof generation process.

2. The second strategy is to carry out all computations off-chain while keeping only the necessary data and zk-proofs on-chain.

There are other strategies too that are implemented within specific components of the Hermez 2.0 system. For instance:

1. The way in which the bridge smart contract is implemented, such as settling accounts in a UTXO manner, by only using the Exit Tree Roots.

2. Utilisation of specialised cryptographic primitives within the zkProver in order to speed up computations and minimise proof sizes, as seen in:

   (a) Running a special zero-knowledge Assembly language (zkASM) for interpretation of byte codes

   (b) Using zero-knowledge tools such as zk-STARKs for proving purposes, which are[very fast, though yielding hefty proofs](https://docs.google.com/presentation/d/1gfB6WZMvM9mmDKofFibIgsyYShdf0RV_Y8TLz3k1Ls0/edit#slide=id.p). 

   So, instead of publishing the sizeable zk-STARK proofs as validity proofs, a zk-SNARK is used to attest to the correctness of the zk-STARK proofs. These zk-SNARKs are in turn published as the validity proofs to state changes. The gas costs reduce from 5M to 350K.


## Conclusion 


Given the EVM opcode compatibility, Hermez 2.0 is designed to process smart contracts seamlessly and verify state changes efficiently. It is promising not only to be secure and efficient but also to accomplish competitive decentralization. In an effort to achieve high-speed proving and succinct proofs for quick verification, the team is focused on the optimization of the zkProver.

The team also leverages the synergies among the different Polygon teams that are also looking into zk-rollups solutions for achieving Ethereum scalability. Although development is still underway, it was important for this document to be released early so as to align with the goal of transparency set by open-source projects, as well as keep the Polygon community of developers and users of Hermez 1.0 updated with the upcoming changes.

The next step is to prepare for a public testnet. Although it is difficult to set a fixed date for that, the plan is to launch it by mid-2022.