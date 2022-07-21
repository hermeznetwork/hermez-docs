[ToC]

# The Hashing State Machine

## Sponge Construction

The **sponge construction** is a simple iterated construction for building a function 

$$
F: \mathbb{Z}^* \to \mathbb{Z}^l
$$

with variable-length input and arbitrary output length based on a fixed-length permutation
$$
f: \mathbb{Z}^b \to \mathbb{Z}^b
$$

operating on a fixed number $b$ of bits. Here $b$ is called the **width**. The array of $b$ bits that $f$ keeps transforming is called the **state**. The state array is split in two chunks of $r$ and $c$ bits respectively. We call $r$ the **bitrate** (or rate) and $c$ the **capacity**. We will understand later on the motivation for this splitting. 

Let us describe how the sponge construction works:

- (*Init Phase*) First of all, the input string is padded with a reversible **padding rule**, in order to achieve a length divisible by $r$. Subsequently, it is cut into blocks of $r$ bits. We also initialize the $b$ bits of the state to zero. 

- (*Absorbing Phase*) In this phase, the $r$-bit input blocks are XORed into the first $r$ bits of the state, interleaved with applications of the function $f$. We proceed until processing all blocks of $r$-bits. Observe that the last $c$ bits corresponding to the capacity value does not absorb any input from the outside. 

- (*Squeezing Phase*) In this phase, the first $r$ bits of the state are returned as output blocks, interleaved with applications of the function $f$. The number of output blocks is chosen at will by the user. Observe that the last $c$ bits corresponding to the capacity value are never output during this phase. Actually, if the output exceeds the specified length, we will just truncate it in order to fit. 

We depict an schema of the sponge construction in the Figure 1 below:

![School Multiplication Example](fig1-sponge-construction.png)
<div align="center"><b> Figure 1: Sponge Function Construction </b></div>

The elements that completely describe a single instance of a sponge construction are: the fixed-length permutation $f$, the padding rule **pad** and the rate value $r$.

## zkEVM Specific Constructions 

The zkEVM uses two different hash functions. The main reason to do so is because the Storage of the EVM uses KECCAK-256 as the hash function used for constructing the corresponding Merkle Trees. However, for zkEVM internal hashes, a Poseidon hash will be used, in order to reduce the proving complexity. In this section, the specific instances of both hash functions will be defined:

### KECCAK-256

The EVM makes use of KECCAK-256 hash function, which is constructed using KECCAK$[512]$ sponge construction. Let us, therefore, define the KECCAK$[c]$ sponge construction. This sponge operates with a width of $1600$ bits and a rate of $1600 - c$. In the case of KECCAK$[512]$, the rate chunk is composed of $1088$ bits (or equivalently, $136$ bytes) and the capacity chunk has $512$ bits (or equivalently, $64$ bytes). The permutation used in KECCAK$[c]$ is KECCAK-$p[1600, 24]$ (See [NIST SHA-3 Standard](https://csrc.nist.gov/publications/detail/fips/202/final)). The last ingredient we need to define in order to completely specify the hash function is the padding rule. In KECCAK$[c]$, the padding pad10*1 is used. If we define $j = (-m-2) \mod{r}$, where $m$ is the length of the input in bits, then the padding we have to append to the original input message is 

$$
P = 1 \mid\mid 0^j \mid\mid 1.
$$

Thus, given an input bit string $M$ and a output length $d$, KECCAK$[c](M, d)$ outputs a $d$ bit string following the previous sponge construction description. 

It should be noted that this construction does **not** follow the FIPS-202 based standard (a.k.a SHA-3). According to the NIST specification,the SHA3 padding changed to

$$
\text{SHA3-256}(M) = \text{KECCAK}[512](M \mid\mid 01, 256).
$$

The difference is the additional $01$ bits appended to the original message, which were not present in the orignal KECCAK specification. 

### Poseidon 

Poseidon (See [Poseidon Paper](https://eprint.iacr.org/2019/458.pdf)) is a hash function designed to minimize prover and verifier complexities when zero-knowledge proofs are generated and validated. The previously defined KECCAK-256 cryptographic hash require large circuits as they are not tailored to finite fields used in ZK proof systems (actually, KECCAK-256 works well in binary fields, and we will see later on that this fact introduces a lot of complexity in the constrain design). For this reason, zkEVM uses Poseidon hash as the main internal hash function. 

More concretely, we will now specify the specific instance of Poseidon that zkEVM uses. We will work over the field $\mathbb{F}_p$ where $p = 2^{64} - 2^{32} + 1$. The state width of the Poseidon permutation is of $8$ field elements (observe that we are changing the paradigm, working with whole field elements instead of working bit-wise) meanwhile we will work with a capacity of $4$-field elements. The Poseidon S-box layer that we will use is the $7$-power S-Box, i.e.

$$
SB(x) = x^7,
$$

The Poseidon instance also requires to specify the number of full and partial rounds of the permutation. In our case, we will use 

$$
R_F = 8 \text{ (number of full rounds) }, \quad R_P = 22 \text{ (number of partial rounds)}
$$

Only one squeezing iteration will be effectuated, with an output of the first $4$ field elements of the state (which consists of approximately $256$-bits, but no more than that). 

The Round Constants and the MDS matrix are completely specified using the previous parameters. 