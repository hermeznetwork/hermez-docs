
## zkProver

The task of creating the execution trace is performed by a component called the **executor**.
The executor takes as inputs the transactions of a batch, a ChainID, the root 
of a Merkle tree representing the previous state of the zkEVM in that chain and
the root of the new state after executing the transactions. 
Additionally, the executor gets values 
of the current state of the zkEVM to build the proof.

The executor is in fact an interpreter of an assembly language
called zkASM. 
The zkASM language is used to build a program called zkEVM-ROM that 
when executed by the Executor provides a suitable execution trace.
In the zkEVM-ROM program, each EVM opcode is implemented with a
set of zkASM instructions. 
Each instruction utilizes a row of the execution trace matrix, 
also known as a "step" of the zkEVM. 

The executor is part of the **zkProver**, which is the
core component of the Polygon zkEVM.

![](./figures/big-picture.png)

<div align="center"><b> Figure 1: Big picture of the Prover in the Polygon zkEVM. </b></div>

