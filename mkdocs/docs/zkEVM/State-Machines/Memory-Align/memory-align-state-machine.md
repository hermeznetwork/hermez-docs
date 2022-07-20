## Memory Align SM

The Memory SM checks memory reads and writes using a 32-byte word access, while the EVM can read and write 32-byte words with offsets at a byte level.

Table 6 shows an example of possible byte-addressed and 32-byte-addressed memory layouts for the same content (three words).

<!-- 
|                                         $\mathbf{ADDRESS}$                                         |                                           $\mathbf{BYTE}$                                            |
| :------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| $\mathtt{0x00}\\ \mathtt{0x01}\\ \mathtt{0x02}\\ \mathtt{\ldots}\\ \mathtt{0x1e}\\ \mathtt{0x1f}$  | $\mathtt{0xc4}\\ \mathtt{0x17}\\  \mathtt{0x4f}\\ \mathtt{\ldots}\\   \mathtt{0x81}\\ \mathtt{0xa7}$ |
| $\mathtt{0x20}\\ \mathtt{0x21}\\ \mathtt{0x22}\\ \mathtt{\ldots}\\ \mathtt{0x3e}\\ \mathtt{0x3f} $ | $ \mathtt{0x88}\\ \mathtt{0xd1}\\ \mathtt{0x1f}\\ \mathtt{\ldots}\\ \mathtt{0xb7}\\ \mathtt{0x23} $  |
| $\mathtt{0x40}\\ \mathtt{0x41}\\ \mathtt{0x42}\\ \mathtt{\ldots}\\ \mathtt{0x5e}\\ \mathtt{0x5f}$  |  $\mathtt{0x6e}\\ \mathtt{0x21}\\ \mathtt{0xff}\\ \mathtt{\ldots}\\ \mathtt{0x54}\\ \mathtt{0xf9} $  |

| $\mathbf{ADDRESS}$ |  $\mathbf{32-BYTE~~WORD}$  |
| :----------------: | :------------------------: |
|  $\mathtt{0x00}$   | $\mathtt{0xc4174f...81a7}$ |
|  $\mathtt{0x01}$   | $\mathtt{0x88d11f...b723}$ |
|  $\mathtt{0x02}$   | $\mathtt{0x6e21ff...54f9}$ | 
-->

<div align="center"><b> Table 7: Sample memory layouts for byte and 32-byte access.</b></div>

$$
\begin{array}{|c|c|}
\hline
\mathbf{ADDRESS} &\mathbf{BYTE} \\ \hline
\mathtt{0} &\mathtt{0xc4} \\
\mathtt{1} &\mathtt{0x17} \\
\mathtt{\vdots} &\mathtt{\vdots} \\
\mathtt{30} &\mathtt{0x81} \\
\mathtt{31} &\mathtt{0xa7} \\
\mathtt{32} &\mathtt{0x88} \\
\mathtt{33} &\mathtt{0xd1} \\
\mathtt{\vdots} &\mathtt{\vdots} \\
\mathtt{62} &\mathtt{0xb7} \\
\mathtt{63} &\mathtt{0x23} \\
\hline
\end{array}
$$

$$
\begin{array}{|c|c|}
\hline
\textbf{ADDRESS} & \textbf{32-BYTE WORD} \\ \hline
\mathtt{0} &\mathtt{0xc417...81a7} \\
\mathtt{1} &\mathtt{0x88d1...b723} \\
\hline
\end{array}
$$


The relationship between the 32-byte word addressable layout and the byte addressable layout is called "memory alignment" and the Memory Align SM is the state machine that checks the correctness of this relationship.

In more detail, we have to check the following memory operations:

- $\mathtt{MLOAD}$: It receives an offset and returns the 32 bytes in memory starting at that offset.
- $\mathtt{MSTORE}$: It receives an offset and saves 32 bytes from the offset address of the memory.
- $\mathtt{MSTORE8}$: It receives an offset and saves one byte on that address of the memory.

Notice that, in the general case, $\mathtt{MLOAD}$ requires reading bytes of two different words.

Considering that the content of the memory is the one shown at Table 6, since the EVM is addressed at a byte level, if we want to check a read from the EVM of a word starting at the address $\mathtt{0x22}$, the value that we should obtain is the following:

$$
\mathtt{val} = \mathtt{0x1f \cdots b7236e21}.
$$

We denote the content of the words affected by an EVM memory read as $\mathtt{m}_0$ and $\mathtt{m}_1$.

In our example, these words are the following:

$$
\mathtt{m}_0 = \mathtt{0x} \mathtt{88d11f} \cdots \mathtt{b723},
\quad \mathtt{m}_1 = \mathtt{0x} \mathtt{6e21ff} \cdots \mathtt{54f9}.
$$

We define a read block as the string concatenating the content of the words affected by the read: $\mathtt{m}_0 \mid \mathtt{m}_1$.

Figure 7 shows the affected read words $\mathtt{m}_0$ and $\mathtt{m}_1$ that form the affected read block and the read value $\mathtt{val}$ for a read from the EVM at address $\mathtt{0x22}$ in our example memory of Table 6.

![Schema of MLOAD example](fig7-schm-mld-eg.png)
<div align="center"><b> Figure 1: Schema of MLOAD Example </b></div>

Let us now introduce the flow at the time of validating a read. Suppose that we want to validate that if we perform an $\mathtt{MLOAD}$ operation at the address $\mathtt{0x22}$, we get the previous value $\mathtt{0x1f\dotsb7236e21}$. At this point, the main state machine will perform several operations. First of all, it will have to query for the values $\mathtt{m}_0$ and $\mathtt{m}_1$. Henceforth, it must call the Memory SM in order to validate the previous queries.

Observe that it is easy to extract the memory positions to query from the address $\mathtt{0x22}$. In fact, if $a$ is the memory position of the $\mathtt{MLOAD}$ operation, then $\mathtt{m}_0$ is always stored at the memory position $\lfloor \frac{a}{32} \rfloor$ and $\mathtt{m}_1$ is stored at the memory position $\lfloor \frac{a}{32} \rfloor + 1$. In our example, $a = \mathtt{0x22} = 34$. Hence, $\mathtt{m}_0$ is stored at the position $\lfloor \frac{32}{34} \rfloor = \mathtt{0x01}$ and $\mathtt{m}_1$ is stored at the position $\lfloor \frac{32}{34} \rfloor + 1= \mathtt{0x02}$.

Secondly, we should extract the correct $\mathtt{offset}$. The $\mathtt{offset}$ represents an index between $0$ and $31$ indicating the number of bytes we should offset from the starting of $\mathtt{m}_0$ to correctly place $\mathtt{val}$ in the block. In our case, the $\mathtt{offset}$ is $2$. Similarly as before, it is easy to obtain the offset from $a$. In fact, the it is equal to $a$ $(\mathrm{mod} \ 32)$. Now, the Main SM will check via a Plookup to the Memory Align State Machine that \val is a correct read given the affected words $\mathtt{m}_0$ and $\mathtt{m}_1$ and the $\mathtt{offset}$. That is, we should check that the value $\mathtt{val}$ can be correctly split into $\mathtt{m}_0$ and $\mathtt{m}_1$ using the provided $\mathtt{offset}$.

Similarly, $\mathtt{MSTORE}$ instruction requires, in general, writing bytes in two words.

The idea is very similar, but we are provided with a value \val that we want to write into a specific location of the memory. We will denote by $\mathtt{w}_0$ and $\mathtt{w}_1$ the words that arise from $\mathtt{m}_0$ and $\mathtt{m}_1$ after the corresponding write.

Following our previous example, suppose that we want to write

$$
\mathtt{val} = \mathtt{0xe201e6\dots662b}
$$

in the address $\mathtt{0x22}$ of the byte-addressed Ethereum memory. We are using the same $\mathtt{m}_0$ and $\mathtt{m}_1$ (and since we are writting into the same address as before) and they will transition into (see Figure 8 ):

$$
\mathtt{w}_0 = \mathtt{0x88d1}\color{-red!75}\mathtt{e201e6\dots}\color{black},\quad \mathtt{w}_1 = \mathtt{0x}\color{-red!75} \mathtt{662b}\color{black} \mathtt{ff\dots54f9}.
$$

![Schema of MSTORE example](fig8-schm-mstr-eg.png)
<div align="center"><b> Figure 8: Schema of MSTORE example. </b></div>

Just as before, the main state machine will need to perform several operations. We will be given an address $\mathtt{addr}$, an offset value $\mathtt{offset}$ and a value to be wrote $\mathtt{val}$. Identically as before, the Main SM will be in charge of reading the zkEVM memory to find $\mathtt{m}_0$ and $\mathtt{m}_1$ from the given address and offset. Of course, the validity of this query should be performed with a specific Plookup into the Memory SM, just as before.

Now, the Main SM can compute $\mathtt{w}_0$ and $\mathtt{w}_1$ from all the previous values in a uniquely way. The way of validating that we are providing the correct $\mathtt{w}_0$ and $\mathtt{w}_1$ is to perform a Plookup into the Memory Align SM. That is, we will check that the provided values $\mathtt{w}_0$ and $\mathtt{w}_1$ are correctly constructed from the provided $\mathtt{val}$, $\mathtt{m}_0$, $\mathtt{m}_1$ and $\mathtt{offset}$ values.

Finally, the last opcode $\mathtt{MSTOREE}$ works similarly, *but it only affects one word* $\mathtt{m}_0$. Moreover, we can only write one byte and hence, only the less significant byte of $\mathtt{val}$ will be considered into the write. Observe that, in this opcode, $\mathtt{m}_1$ and $\mathtt{w}_1$ are unconstrained. 