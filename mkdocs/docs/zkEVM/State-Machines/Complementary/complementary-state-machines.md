## Global

The Global State Machine is a state machine that computes various constant polynomials used by some of the state machines of the zkEVM. These polynomials are typically used for the distinct lookup arguments that PIL is able to perform.

At this moment, the Global SM used by the zkEVM is made of:

```
include "config.pil";

namespace Global(%N);
    pol constant L1;
    pol constant BYTE;
    pol constant BYTE2;
```


At this moment, the polynomials computed by the Global SM are showed in Table 1.

|   Row    |    L1    |   Row    |   BYTE   |   Row    |  BYTE2   |
| :------: | :------: | :------: | :------: | :------: | :------: |
|    1     |    1     |    1     |    0     |    1     |    0     |
|    2     |    0     |    2     |    1     |    2     |    1     |
|    3     |    0     |    3     |    2     |    3     |    2     |
| $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ |
|   256    |    0     |   256    |   255    |  65536   |  65535   |
|   257    |    0     |   257    |    0     |  65537   |    0     |
|   258    |    0     |   258    |    1     |  65538   |    1     |
| $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ | $\vdots$ |
|    N     |    0     |    N     |   255    |    N     |  65535   |
<div align="center"><b> Table 1: Description of the polynomials computed by the Global State Machine. </b></div>

## Byte4

The Byte4 State Machine takes as input two $16$-bit numbers and generates a $32$-bit number from them. This generation is obtained through the concatenation of the input numbers. A working example can be find in Table 2.

| **row**  | **SET**  |    **freeIn**     |        **out**        |       **out'**        |
| :------: | :------: | :---------------: | :-------------------: | :-------------------: |
|    1     |    0     | $\textsf{0xba04}$ | $\textsf{0x00000000}$ | $\textsf{0x0000ba04}$ |
|    2     |    1     | $\textsf{0x3ff2}$ | $\textsf{0x0000ba04}$ | $\textsf{0xba043ff2}$ |
|    3     |    0     | $\textsf{0x4443}$ | $\textsf{0xba043ff2}$ | $\textsf{0x00004443}$ |
|    4     |    1     | $\textsf{0xc1d1}$ | $\textsf{0x00004443}$ | $\textsf{0x4443c1d1}$ |
|    5     |    0     | $\textsf{0xd11e}$ | $\textsf{0x4443c1d1}$ | $\textsf{0x0000d11e}$ |
|    6     |    1     | $\textsf{0x6ab9}$ | $\textsf{0x0000d11e}$ | $\textsf{0xd11e6ab9}$ |
|    7     |    0     |     $\vdots$      | $\textsf{0xd11e6ab9}$ |       $\vdots$        |
| $\vdots$ | $\vdots$ |     $\vdots$      |       $\vdots$        |       $\vdots$        |
<div align="center"><b> Table 2: Example of they Byte4 SM. </b></div>

The Byte4 SM works as follows. In one clock, the first input $x$ is moved to the $\textsf{out}$ column. In the following clock, $x$ is concatenated to the second input $y$ and moved to the $\textsf{out}$ column. In order to make this "moving" possible, we introduce a constant polynomial, called $\textsf{SET}$, defined as follows:

\begin{aligned}
\textsf{SET} =
\begin{cases}
1, & \text{if } \textsf{row} \text{ is even}\\
0, & \text{if } \textsf{row} \text{ is odd}
\end{cases}
.
\end{aligned}

Once $\textsf{SET}$ is defined, the moving action is naturally enforced with the following constraint:

\[
\textsf{out}' = (1 - \textsf{SET}) \cdot \textsf{freeIn} + \textsf{SET} \cdot (2^{16} \cdot \textsf{out} + \textsf{freeIn}).
\]

Notice that when $\textsf{SET} = 0$, then $\textsf{out}' = \textsf{freeIn}$, i.e., $\textsf{out}$ is set to be the first input. In contrast, when $\textsf{SET} = 1$, then $\textsf{out}' = 2^{16} \cdot \textsf{out} + \textsf{freeIn}$, i.e., the previous input (stored in $\textsf{out}$) is set to be the upper part of $\textsf{out}$, while the second input is set to be the lower part of $\textsf{out}$. To achieve soundness, we must also check that both inputs are elements made at most of $2$ bytes.

The previous constraints are reflected in the PIL code for this SM:

```
include "config.pil";
include "global.pil";

namespace Byte4(%N);
  // Constant Polynomials
  pol constant SET; // 0, 1, 0, 1, 0, 1, ...

  // Input Polynomials
  pol committed freeIn;

  // State Variables
  pol committed out;

  // Constraints
  freeIn in Global.Byte2; // Check that input is in [0,1,...,65535]
  out' = (1 - SET)*freeIn + SET*(2**16*out + freeIn);
```
