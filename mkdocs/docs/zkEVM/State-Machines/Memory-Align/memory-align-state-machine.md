## Memory Align SM

The Memory SM checks memory reads and writes using a 32-byte word access, while the EVM can read and write 32-byte words with offsets at a byte level.

Table 6 shows an example of possible byte-addressed and 32-byte-addressed memory layouts for the same content (three words).

<div align="center"><b> Table 7: Sample memory layouts for byte and 32-byte access.
 </b>

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

The relationship between the $32$-byte word addressable layout and the byte addressable layout is called "memory alignment" and the Memory Align SM is the state machine that checks the correctness of this relationship.

In more detail, we have to check the following memory operations:

- $\texttt{MLOAD}$: It receives an offset and returns the 32 bytes in memory starting at that offset.
- $\texttt{MSTORE}$: It receives an offset and saves 32 bytes from the offset address of the memory.
- $\texttt{MSTOREE}$: It receives an offset and saves one byte on that address of the memory.

Notice that, in the general case, $\texttt{MLOAD}$ requires reading bytes of two different words.

Considering that the content of the memory is the one shown at Table 6, since the EVM is addressed at a byte level, if we want to check a read from the EVM of a word starting at the address $\texttt{0x22}$, the value that we should obtain is the following:

$$
\texttt{val} = \mathtt{0x1f \cdots b7236e21}.
$$

We denote the content of the words affected by an EVM memory read as $m_0$ and $m_1$.

In our example, these words are the following:

$$
\texttt{m}_0 = \mathtt{0x} \mathtt{88d11f} \cdots \mathtt{b723},

\quad \texttt{m}_1 = \mathtt{0x} \mathtt{6e21ff} \cdots \mathtt{54f9}.
$$

We define a read block as the string concatenating the content of the words affected by the read: $\texttt{m}_0 \mid \texttt{m}_1$.

Figure 7 shows the affected read words $m_0$ and $m_1$ that form the affected read block and the read value $\texttt{val}$ for a read from the EVM at address $\texttt{0x22}$ in our example memory of Table 6.

<p align="center"><img src="fig7-schm-mld-eg.png" width="700" /></p><div align="center"><b> Figure 7: Schema of MLOAD example </b></div>

Let us now introduce the flow at the time of validating a read. Suppose that we want to validate that if we perform an $\texttt{MLOAD}$ operation at the address $\mathtt{0x22}$, we get the previous value $\mathtt{0x1f\dotsb7236e21}$. At this point, the main state machine will perform several operations. First of all, it will have to query for the values $\texttt{m}_0$ and $\texttt{m}_1$. Henceforth, it must call the Memory SM in order to validate the previous queries.

Observe that it is easy to extract the memory positions to query from the address $\mathtt{0x22}$. In fact, if $a$ is the memory position of the $\texttt{MLOAD}$ operation, then \mF is always stored at the memory position $\lfloor \frac{a}{32} \rfloor$ and \mS is stored at the memory position $\lfloor \frac{a}{32} \rfloor + 1$. In our example, $a = \mathtt{0x22} = 34$. Hence, \mF is stored at the position $\lfloor \frac{32}{34} \rfloor = \mathtt{0x01}$ and \mS is stored at the position $\lfloor \frac{32}{34} \rfloor + 1= \mathtt{0x02}$.

Secondly, we should extract the correct $\texttt{offset}$. The $\texttt{offset}$ represents an index between $0$ and $31$ indicating the number of bytes we should offset from the starting of $\texttt{m}_0$ to correctly place $\texttt{val}$ in the block. In our case, the $\texttt{offset}$ is $2$. Similarly as before, it is easy to obtain the offset from $a$. In fact, the it is equal to $a$ $(\mod{32})$. Now, the Main SM will check via a Plookup to the Memory Align State Machine that \val is a correct read given the affected words $\texttt{m}_0$ and $\texttt{m}_1$ and the $\texttt{offset}$. That is, we should check that the value $\texttt{val}$ can be correctly split into $\texttt{m}_0$ and $\texttt{m}_1$ using the provided $\texttt{offset}$.

Similarly, $\texttt{MSTORE}$ instruction requires, in general, writing bytes in two words.

The idea is very similar, but we are provided with a value \val that we want to write into a specific location of the memory. We will denote by $\texttt{w}_0$ and $\texttt{w}_1$ the words that arise from $\texttt{m}_0$ and $\texttt{m}_1$ after the corresponding write.

Following our previous example, suppose that we want to write

$$
\texttt{val} = \mathtt{0xe201e6\dots662b}
$$

in the address $\texttt{0x22}$ of the byte-addressed Ethereum memory. We are using the same $\texttt{m}_0$ and $\texttt{m}_1$ (and since we are writting into the same address as before) and they will transition into (see Figure 8 ):

$$
\texttt{w}_0 = \mathtt{0x88d1}\color{-red!75}\mathtt{e201e6\dots}\color{black},\quad \texttt{w}_1 = \mathtt{0x}\color{-red!75} \mathtt{662b}\color{black} \mathtt{ff\dots54f9}.
$$

<p align="center"><img src="fig8-schm-mstr-eg.png" width="700" /></p><div align="center"><b> Figure 8: Schema of MSTORE example. </b></div>

Just as before, the main state machine will need to perform several operations. We will be given an address $\texttt{addr}$, an offset value $\texttt{offset}$ and a value to be wrote $\texttt{val}$. Identically as before, the Main SM will be in charge of reading the zkEVM memory to find $\texttt{m}_0$ and $\texttt{m}_1$ from the given address and offset. Of course, the validity of this query should be performed with a specific Plookup into the Memory SM, just as before.

Now, the Main SM can compute $\texttt{w}_0$ and $\texttt{w}_1$ from all the previous values in a uniquely way. The way of validating that we are providing the correct $\texttt{w}_0$ and $\texttt{w}_1$ is to perform a Plookup into the Memory Align SM. That is, we will check that the provided values $\texttt{w}_0$ and $\texttt{w}_1$ are correctly constructed from the provided $\texttt{val}$, $\texttt{m}_0$, $\texttt{m}_1$ and $\texttt{offset}$ values.

Finally, the last opcode $\texttt{MSTOREE}$ works similarly, $\textit{but it only affects one word}$ $\texttt{m}_0$. Moreover, we can only write one byte and hence, only the less significant byte of $\texttt{val}$ will be considered into the write. Observe that, in this opcode, $\texttt{m}_1$ and $\texttt{w}_1$ are unconstrained.