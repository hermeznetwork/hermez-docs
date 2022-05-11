[ToC]



# About Polygon Hermez

Intergrated seamlessly with the Ethereum ecosystem, Polygon Hermez is a powerful, decentralized technology that provides Layer 2 scalability solutions to blockchain users. With the tremendous increase in the number of transactions taking place on-chain (i.e. base layer Ethereum), Layer 1 solution is already facing throughput, scalability, and speed issues. It is where Polygon Hermez steps in. By providing zk-rollups (zero knowedge rollups) that sit on the top of Ethereum Maninnet, the scalability and the transactions per second (TPS) can be dramatically improved. To prpve that the off-chain computations are correct, Polygon Hermez employs zero-knowldege proofs that can be easily verified. The Layer 2 zero knowldege proofs are based on the complex polynomial computations that help in leveraging Ethereum scaling and provide fast finality to the off-chain transactions.

## What is Polygon Hermez 2.0? 

Polygon Hermez 1.0 version was launched in March 2021. The main focus of Polygon Hermez 1.0 was to sacle payments and transfer ERC-20 tokens. It focussed mainly on decongesting Ethereum main chian by taking transactions off from the main chain and executimng them off-chain; this resulted in considerable increase in number of trandactions to be executed per second to 2000, which was a big improvemnet over layer 1 Ethereum. See https://ethtps.info/Network/Ethereum for viewing Ethereum's live TPS. 

Polygon Hermez 2.0, henceforth called Hermez 2.0,has been developed to emulate Ethereum Virtual Machine(EVM),a machine that executes Ethereum transactions with zero-knowledge prrof validations. . This has been accompalished by developing zero knowledge EVM  which has been designed to recreate all the existing EVM opcodes so that these can be deployed as smart contracts. Although taking on this revolutionary design approach was a hard decision to make, the objective is to minimise the users and dApps friction when using the solution. It is an approach that requires recreation of all EVM opcodes for transparent deployment of existing Ethereum smart contracts. For this purpose a new set of technologies and tools are being created and engineered.

This document presents a high-level description of the upcoming Polygon Hermez 2.0 solution, which includes its main components and design. It also seeks to highlight how Polygon Hermez 2.0 departs from the original design of Hermez 1.0.


//The team has been working on the development of Hermez 2.0 as a zero-knowledge Ethereum Virtual Machine (zkEVM), a virtual machine that executes Ethereum transactions in a transparent way, including smart contracts with zero-knowledge-proof validations. 

//is a decentralized Ethereum Layer 2 scalability solution utilising cryptographic zero-knowledge technology to provide validation and fast finality of off-chain transaction computations.

//Hermez 1.0, which has been live since March 2021, is decentralized, permissionless and scales up to 2000 /////transactions per second (tps). It has therefore successfully accomplished the purpose it was designed for, //which was scaling payments and transfers of ERC-20 tokens.


## Architecture


Over and above what its predecessor was designed to do, the main functionality of Hermez 2.0 is to provide smart contract support. It performs the task of state transtioning resulting from the  Ethereum Layer 2 transaction executions (transactions that users send to the network). Subsequently, by employing zero-knowledge functionality, it generates validity proofs that attest to the correctness of these state change computations carried out off-chain.

The major components of Hermez 2.0 are:

- Proof of Efficiency Algorithm 
- zkNode Software
- zkProver
- LX-to-LY Bridge
- Sequencers
- Aggregators //(who are the participants requisite in reaching network consensus)
- Active users of the Hermez 2.0 network who create transactions.


The skeletal architecture of Hermez 2.0 is therefore as follows.

![Hermez Architecture](/IMAGES/fig1-simpl-arch.png)
<p align="center"><img src="IMAGES/fig1-simpl-arch.png" width="600" /></p>
<div align="center"><b> Figure 1 : Skeletal Overview Hermez 2.0 </b></div>





### The Consensus Algorithm

The begging question is "What consensus algorithm does Hermez 2.0 use?" That is, how do Hermez 2.0 nodes agree on which block to be added to the chain.

Like its earlier version, which uses Proof-of-Donation (PoD), Hermez 2.0 is designed to be decentralized. However, the old Proof-of-Donation gives way to a newer consensus algorithm called Proof of Efficiency.

A detailed description of Hermez 2.0's PoE is found[ here](https://ethresear.ch/t/proof-of-efficiency-a-new-consensus-mechanism-for-zk-rollups/11988).



#### Why the need to replace PoD?

The PoD model fell out of favour for several reasons.

Firstly, the PoD model with the complexity of its auction protocol, is vulnerable to attacks, especially at the bootstrapping phases. Also, since at any given point in time, the network is controlled by any permissionless participant, there is a risk for the network to suffer service level delays should such a third party turn malicious or experience operational issues.

Secondly, the auction protocol has proved to be not only complex for coordinators and validators but also costly. More so considering that not every competing validator gets rewarded but only the most effective.

Thirdly, the efficacy of selecting “the best” operator amounts to a winner-takes-all model, which turns out to be unfair to competitors with slightly less performance. Consequently, only a few select operators validate batches more often than others, defeating the very ideal of network decentralization.



#### Why the preference for PoE? 

The Proof of Efficiency (PoE) model is preferred mainly due to its simplicity. It solves many of the challenges experienced with the PoD model, such as attacks by design, as discussed above. 

A strategic implementation of PoE promises to ensure that the network, 

- maintains permissionless opportunity to produce L2 batches, 
- is efficient, which is key for overall network performance, 
- attains an acceptable degree of decentralization, 
- is secure from malicious attacks, especially by validators, and 
- keeps proportionate balance between the overall validation effort and the value in the network.Possibilities of coupling PoE with a PoS are being explored.



#### Data Availability

A typical zk-rollup schema requires that both the data (which is required by users to reconstruct the full state) and the validity proofs be published on-chain.

However, given the Ethereum setting, publishing data on-chain means incurring gas fees. This leads to a hard choice between full zk-rollup configuration and an affordable network.

Unless, among other things, the proving module can be highly accelerated to mitigate costs for the validators, a hybrid schema remains inevitable.

Although the team is yet to finalise the best consensus configuration, the obvious options are;

- Validium option: the proofs remain on-chain but the data is stored somewhere else,

- Volition option: full zk-rollup for some transactions, with both data and proofs on-chain, while for other transactions only the proofs go on-chain.



### The PoE Smart Contract 

Rollups entail two processes, batching of transactions and validation of the batched transactions. Hermez 2.0 will use Sequencers and Aggregators to respectively carry out the two processes, batching and validation of transactions.

That is, Sequencers will collect transaction-requests into batches and add them to the PoE smart contract, while Aggregators will check validity of transaction batches and provide validity proofs.

The PoE smart contract therefore makes two basic calls; A call to receive batches from Sequencers, and another call to Aggregators, requesting batches to be validated. See **Figure 2** below.



#### Proof of Efficiency Tokenomics

The PoE smart contract imposes a few requirements on Sequencers and Aggregators.

**Sequencers' Constraints**;

- Anyone running the zkNode, which is the software necessary for running a Hermez 2.0 node, can be a Sequencer. 
- Every Sequencer must pay a fee in $Matic in order to earn the right to create and propose batches. 
- A Sequencer who proposes valid batches, which consist of valid transactions, is incentivised with fees paid by transaction-requestors, the users of the network. 
- Specifically, a Sequencer collects L2 transactions from users, preprocesses them as a new L2 batch, then proposes the batch as a valid L2 transaction to the PoE smart contract.

**Aggregators' Constraints**;

- An Aggregator's task is to produce validity proofs for the L2 transactions proposed by Sequencers.
- In addition to running Hermez 2.0's zkNode software, Aggregators need to have specialised hardware for creating the zero-knowledge validity proofs. We herein call it the zkProver.
- The Aggregator who is the first to submit a validity proof for a given batch or batches, earns the Matic fees paid by the Sequencer(s) of the batch(es).
- The Aggregators need only indicate their intention to validate transactions and then run the race, to produce validity proofs, based on their own strategy.



<p align="center"><img src="fig2-simple-poe.png" width="650" /></p>
<div align="center"><b> Figure 2 : Simplified Proof of Efficiency </b></div>





### The zkNode

The network requires the release of the client that implements the synchronization and covers the roles of participants as Sequencers or Aggregators. zkNode is such a client. It is the software needed to run a Hermez 2.0 node.

Polygon Hermez 2.0 participants will choose how they participate; either as simply a node, to know the state of the network; or participate in the process of batch production in any of the two roles, as a **Sequencer** or an **Aggregator**. An Aggregator will be running the zkNode but also performs validation using the core part of the zkEVM, called the zkProver (this is labelled Prover in Figure 3 below.)

Other than the sequencing and the validating processes, the zkNode also enables synchronisation of batches and their validity proofs, which happens only after these have been added to L1. It uses a subcomponent called the Synchronizer.

The **Synchronizer** is therefore responsible for reading events from the Ethereum blockchain, including new batches, in order to keep the state fully synced. The information read from these events must be stored in the database. Synchronizer also handles possible reorgs, which will be detected by checking if last `ethBlockNum` and last `ethBlockHash` are synced.



<p align="center"><img src="fig3-zkNode-arch.png" width="600" /></p>
<div align="center"><b> Figure 3 : Hermez 2.0 zkNode Diagram </b></div>



The architecture of zkNode is modular and implements a set of functions as depicted in Figure 3 above.

The **RPC** (remote procedure calls) interface is a JSON RPC interface which is Ethereum compatible. It is implemented to enable integration of the zkEVM with existing tooling, such as Metamask, Etherscan and Infura. RPC adds transactions to the **Pool** and interacts with the **State** using read-only methods. 

The **State** subcomponent implements the Merkle Tree and connects to the DB backend. It checks integrity at block level (i.e., information related to; gas, block size, etc.) and some transaction-related information (e.g., signatures, sufficient balance, etc.). State also stores smart contract (SC) code into the Merkle tree and processes transactions using the EVM.



### The zkProver

Hermez 2.0 employs state-of-the-art zero-knowledge technology. It will use a zero-knowledge prover, dubbed zkProver, which is intended to run on any server and is being engineered to be compatible with most consumer hardware.Every Aggregator will use the zkProver to validate batches and provide validity proofs. zkProver has its own detailed architecture which is outlined below. It will consist mainly of the Main State Machine Executor, a collection of secondary State Machines each with its own executor, the STARK-proof builder, and the SNARK-proof builder. See **Figure 4** below for a simplified diagram of the Hermez 2.0 zkProver.

<p align="center"><img src="fig4-zkProv-arch.png" width="650" /></p>
<div align="center"><b> Figure 4 : A Simplified zkProver Diagram </b></div>



In a nutshell, the zkEVM expresses state changes in polynomial form. Therefore, the constraints that each proposed batch must satisfy are in fact polynomial constraints or polynomial identities. That is, all valid batches must satisfy certain polynomial constraints.



#### The Main State Machine Executor

The Main Executor handles the execution of the zkEVM. This is where EVM Bytecodes are interpreted using a new zero-knowledge Assembly language (zkASM), specially developed by the team. The executor also sets up the polynomial constraints that every valid batch of transactions must satisfy. Another new language, also a specially developed language by the team, called the Polynomial Identity Language (or PIL) is used to encode all the polynomial constraints.



#### A Collection of Secondary State Machines

Every computation required in proving correctness of transactions is represented in the zkEVM as a state machine. The zkProver, being the most complex part of the whole project, consists of several state machines; from those carrying out bitwise functionalities (e.g., XORing, padding, etc.) to those performing hashing (e.g., Keccak, Poseidon), even to verifying signatures (e.g., ECDSA).

The collection of secondary State Machines therefore refers to a collection of all state machines in the zkProver. It is not a subcomponent per se, but a collection of various executors for individual secondary state machines. These are; the Binary SM, the Memory SM, the Storage SM, the Poseidon SM, the Keccak SM and the Arithmetic SM. See Figure 5 below for dependencies among these SMs.

Depending on the specific operations each SM is responsible for, some use both zkASM and PIL, while others use only one.



<p align="center"><img src="fig5-col-sm-zkprov.png" width="800" /></p>
<div align="center"><b> Figure 5 : Hermez 2.0 State Machines </b></div>





### STARK-proof Builder

STARK, which is short for Scalable Transparent ARgument of Knowledge, is a proof system that enables provers to produce verifiable proofs, without the need for a trusted setup. 

STARK-proof Builder refers to the subcomponent used to produce zero-knowledge STARK-proofs, which are zk-proofs attesting to the fact that all the polynomial constraints are satisfied.

State machines generate polynomial constraints, and zk-STARKs are used to prove that batches satisfy these constraints. In particular, zkProver utilises Fast Reed-Solomon Interactive Oracle Proofs of Proximity (RS-IOPP), colloquially called [FRI](https://drops.dagstuhl.de/opus/volltexte/2018/9018/pdf/LIPIcs-ICALP-2018-14.pdf), to facilitate fast zk-STARK proving.



### SNARK-proof Builder

SNARK, which is similarly short for Succinct Non-interactive ARgument of Knowledge, is a proof system that produces verifiable proofs.

Since STARK-proofs are way larger than SNARK-proofs, Hermez 2.0 zkProver uses SNARK-proofs to prove the correctness of these STARK-proofs. Consequently, the SNARK-proofs, which are much cheaper to verify on L1, are published as the validity proofs.

The aim is to generate a [CIRCOM](https://www.techrxiv.org/articles/preprint/CIRCOM_A_Robust_and_Scalable_Language_for_Building_Complex_Zero-Knowledge_Circuits/19374986/1) circuit which can be used to generate or verify a SNARK proof. As to whether a PLONK or a GROTH16 SNARK proof will be used, is yet to be decided on.



### The LX-to-LY Bridge

A typical Bridge smart contract is a combination of two smart contracts, one deployed on the first chain and the other on the second.

The L2-to-L1 Bridge smart contract for Hermez 2.0 is also composed of two smart contracts; the Bridge L1 Contract and the Bridge L2 Contract. The two contracts are practically identical except for where each is deployed. **Bridge L1 Contract** is on the Ethereum mainnet in order to manage asset transfers between rollups, while **Bridge L2 Contract** is on a specific rollup and it is responsible for asset transfers between mainnet and the rollup (or rollups).



#### The Bridge L1 Contract

Firstly, **Bridge L1 Contract** carries out two operations, `bridge` and `claim`. The `bridge` operation transfers assets from one rollup to another, while the `claim` operation applies when the contract makes a claim from any rollup.

Bridge L1 Contract requires two Merkle trees in order to perform the above operations; `globalExitTree` and `mainnet exit tree`. The `globalExitTree` contains all the information of exit trees of all rollups, whereas the `mainnet exit tree` has information of transactions made by users who interact with the mainnet. 

A contract named the **global exit root manager L1** is responsible for managing exit roots across multiple networks. 

The exit tree structure is depicted in Figure 6, below.



<p align="center"><img src="fig6-exit-tr-strct.png" width="700" /></p>
<div align="center"><b> Figure 6 : The Exit Tree Structure </b></div>



#### The Bridge L2 Contract

Secondly, **Bridge L2 Contract** will be deployed on L2 with Ether on it. The Ether will be set on the genesis in order to enable mint/burn of native Ether.

Bridge L2 Contract also requires all the information of exit trees of all rollups contained in the `globalExitTree` Merkle tree. In this case, a smart contract named the **global exit root manager L2** is responsible for managing exit roots across multiple networks.

Note that when a batch is verified in the PoE smart contract in L1, the rollup exit root is updated in the global exit root manager L1. Bridge L2 Contract handles the rollup side of the `bridge` and the `claim` operations, as well as interacting with the `globalExitTree` and the `rollup exit tree`, mainly to update exit roots.



#### Concluding the LX-to-LY Bridge

Typically, a Bridge smart contract is an L2-to-L1 Bridge, but the Hermez 2.0 Bridge is more flexible and interoperable. It can function as a bridge between any two arbitrary Layer 2 chains, L2_A and L2_B , or between any Layer 2, L2_X and L1, the Ethereum blockchain. It consequently allows asset transfers among multiple rollups. Hence the term "LX-to-LY Bridge".





## Hermez 2.0 Design Ideals 



The decisions described above about engineering and implementation will help Hermez 2.0 attain its design ideals. That is, a network which is; permissionless, decentralized, secure, efficient and with verifiable block data.

Development efforts aim at **permissionless-ness**, that is, allowing anyone with the Hermez 2.0 software to participate in the network. For instance, the consensus algorithm will give everyone the opportunity to be a Sequencer or an Aggregator.

Data availability is most crucial for **decentralization**, where every user has sufficient data needed to rebuild the full state of a rollup. As discussed above, the team still has to decide on the best configuration of the data availability. The aim is to ensure that there is no censorship and no one party can control the network.

Hermez 2.0 was designed with **security** in mind. As a L2 solution, most of the security is inherited from Ethereum. Smart contracts will warrant that anyone who executes state changes must; firstly, do it correctly; secondly, create a proof that attests to the validity of a state change; and thirdly, avail validity proofs on-chain for verification.



### Efficiency and the Overall Strategy

Efficiency is key to network performance. Hermez 2.0 therefore applies several implementation strategies to guarantee efficiency.

The first strategy is to deploy PoE, which incentivizes the most efficient aggregators to participate in the proof generation process.

The second strategy is to carry out all computations off-chain while keeping only the necessary data and zk-proofs on-chain.

Various other strategies are implemented within specific components of the Hermez 2.0 system. For instance;

1. The way in which the Bridge smart contract is implemented, such as settling accounts in an UTXO manner, by only using the Exit Tree Roots.

2. Utilisation of specialised cryptographic primitives within the zkProver in order to speed up computations and minimise proof sizes, seen in;

   (a) Running a special zero-knowledge Assembly language (zkASM) for interpretation of byte codes,

   (b) Using zero-knowledge tools such as zk-STARKs for proving purposes, which are[ very fast, though yielding hefty proofs](https://docs.google.com/presentation/d/1gfB6WZMvM9mmDKofFibIgsyYShdf0RV_Y8TLz3k1Ls0/edit#slide=id.p). 

   So instead of publishing the sizable zk-STARK proofs as validity proofs, a zk-SNARK is used to attest to the correctness of the zk-STARK proofs. 

   These zk-SNARKs are in turn published as the validity proofs to state changes. The gas costs reduce from 5M to 350K.





## Conclusion 



Given the EVM OPCODE-Compatibility, Hermez 2.0 is designed to seamlessly process smart contracts and efficiently verify state changes. It is promising not only to be secure and efficient, but also to accomplish competitive decentralization.

In the effort to achieve high-speed proving and succinct proofs for quick verification, the team is focused on the optimization of the zkProver.

The team also leverages the synergies among the different Polygon teams that are also looking into zk-rollups solutions for achieving Ethereum scalability.

Although development is still underway, it was important for this document to be released in keeping with the transparency of open-source projects, as well as keeping the Polygon community of developers and users of Hermez 1.0 updated.

The next step is to prepare for a public testnet. Although it is difficult to set a definitive date, the plan is to launch the testnet in mid-2022.