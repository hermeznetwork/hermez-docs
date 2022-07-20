<!-- TODO: I assume that a knowledge about registers and the basic stuff has been explained in another document has been done

Suppose we are giving a state machine with a set of registers
\[
\{\text{A}, \text{B}, \dots \},
\]
a set of defined ROM instructions between them
\[
\{\text{INS1}, \text{INS2}, \text{INS3}, \dots, \mathbf{FREE} \},
\]
and a set of methods implemented in the executor, to be load into the registers as free inputs
\[
\{\text{ExecutorMethod()}, \dots \}.
\]
Recall that registers are, in fact, composed of $4$ columns. Hence, for instance, $A$ can be decomposed as four columns $A_0, A_1, A_2, A_3$, where $A_0$ represents the less significative bits of $A$ and similarly, represents $A_3$ the most significative bits of $A$.


  Hence, it consists on two rows of the resulting table.

  \begin{figure}[H]
      \centering
      \begin{tabular}{| c | c | c | c | c | c | c | c |}
          \hline
          \textbf{FREE0} & \textbf{FREE1} & \textbf{FREE2} & \textbf{FREE3} & $A_0$ & $A_1$ & $A_2$ & $A_3$ \\
          \hline
          0000           & 0000           & 0101           & 0111           & 0000  & 0000  & 0000  & 0000  \\
          0000           & 0000           & 0101           & 0111           & 0000  & 0000  & 0101  & 0111  \\
          \hline
      \end{tabular}
  \end{figure} -->

Ethereum is a state machine that transition from an old state to a new state by reading a series of transactions. It is a natural choice, in order to interpret the set of EVM opcodes, to design another state machine as for the interpreter. One should think of it as building a state machine inside another state machine, or more concretely, building an Ethereum inside the Ethereum itself. The distinction here is that the former contains a virtual machine, the zkEVM, that is zero-knowledge friendly.

## The zkEVM as a Microprocessor

Following the previous discussion, it is good to see the outer state machine as a microprocessor. What we have done is creating a microprocessor, composed by a series of assembly instructions and its associate program (i.e., the ROM) running on top of it, that interprets the EVM opcodes.

![](./figures/CPU.png)

<div align="center"><b> Figure 1: Block diagram of a basic uniprocessor-CPU computer. Black lines indicate data flow, whereas red lines indicate control flow; arrows indicate flow directions. </b></div>

As in input, the microprocessor will take the transactions that we want to process and the old state. After fetching the input, the ROM is used to interpret the transactions and generate a new state (the output) from them.

![](./figures/machine-cycle.png)

<div align="center"><b> Figure 2: Block diagram of a basic machine cycle. </b></div>

## The Role of the zkASM

The zero-knowledge Assembly (zkASM) is the language used to describe, in a more abstract way, the ROM of our processor. Specifically, this ROM will tell the Executor how to interpret the distinct types of transactions that it could possibly receive as an input. From this point, the Executor will be capable of generating a set of polynomials will describe the state transition and will be later on used by the STARK generator to generate a prove of correctness of this transition.

![](./figures/big-picture.png)

<div align="center"><b> Figure 3: Big picture of the Prover in the zkEVM project, with focus in the zkASM part. </b></div>
