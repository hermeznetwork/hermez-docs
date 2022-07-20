## Fibonacci State Machine

Let's consider that we want to validate that a certain 
number is a number of a [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) given certain 
initial conditions.
To do so, we can build a state machine with two registries, $A$ and $B$ as shown in the following picture:

![Fibonacci Sequence](figures/fibonacci-sequence.pdf.png)

Notice that the initial conditions for the state machine are $A_1=0$ and $B_1=1$ and that we have the following relations between the states of these registries:

\begin{aligned}
A_{i+1} &= B_i, \\
B_{i+1} &= A_i + B_i.
\end{aligned}

Let's represent the states of these registries as polynomials in $\mathbb{Z}_p[x]$ evaluated on the subgroup $H = \{\omega, \omega^2, \omega^3, \omega^4, \omega^5, \omega^6, \omega^7, \omega^8 = 1\}$ of $8$-roots of unity in $\mathbb{Z}_p^*$. Then, we have the following relations:

\begin{aligned}
A(\omega^i) &= A_i \quad \Longrightarrow \quad A = [0, 1, 1, 2, 3, 5, 8, 13] \\
B(\omega^i) &= B_i \quad \Longrightarrow \quad B = [1, 1, 2, 3, 5, 8, 13, 21]
\end{aligned}

The relations between the states of registries can be translated into identities in the polynomial setting as follows:

\begin{aligned}
A(x\omega) &= \bigg\lvert_H  B(x), \\
B(x\omega) &= \bigg\lvert_H  A(x) + B(x).
\end{aligned}

However, the previous identities do not correctly and uniquely describe our sequence because:

  1.  The registries are not cyclic: When we evaluate the identities at $\omega^8$:

    \begin{aligned}
    A(\omega^9) &= A(\omega) = 0 \neq  21 = B(\omega^8), \\
    B(\omega^9) &= B(\omega) = 1 \neq  34 = A(\omega^8) + B(\omega^8).
    \end{aligned}

  2.  We can use other initial conditions, for example $(2,4)$, that also fulfill the identities: $(2,4)\to(4,6)\to(6,10)\to(10,16)\to(16,26)\to(26,42)\to(42,68)\to(68,110).$

We have to modify a little our solution in order correctly and uniquely describe the Fibonacci sequence with cyclic polynomial identities. To do that, let's add an auxiliary registry $C$:

![Fibonacci Sequence Aux](figures/fibonacci-sequence-aux.pdf.png)

The corresponding polynomial $C$ is:

\begin{aligned}
C(\omega^i) &= C_i \quad \Longrightarrow \quad C = [1, 0, 0, 0, 0, 0, 0, 0].
\end{aligned}

With this auxiliary registry, we can now fix the polynomial identities as follows:

\begin{aligned}
A(x\omega) &= \bigg\lvert_H  B(x)(1 - C(x\omega)), \\
B(x\omega) &= \bigg\lvert_H (A(x) + B(x))(1 - C(x\omega)) + C(x\omega).
\end{aligned}

Note that now at $x = \omega^8$ the identities are satisfied:

\begin{aligned}
A(\omega^9) &= A(\omega) = 0 = B(\omega^8)(1 - C(\omega)), \\
B(\omega^9) &= B(\omega) = 1 = (A(\omega^8) + B(\omega^8))(1 - C(\omega)) + C(\omega).
\end{aligned}

Observe that we can also use other initial conditions $(A_1, B_1)$ slightly modifying our polynomial identities:

\begin{aligned}
A(x\omega) &= \bigg\lvert_H  B(x)(1 - C(x\omega))+ A_1C(x\omega), \\
B(x\omega) &= \bigg\lvert_H  (A(x) + B(x))(1 - C(x\omega)) + B_1 C(x\omega).
\end{aligned}

In our previous example $(A_1, B_1) = (0, 1)$.

## Proving our State Machine (High Level)

![Polynomial Commitment](figures/polynomial-commitment.pdf.png)

The previous polynomial relations can be efficiently proven through **polynomial commitments** such as [Kate](https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf) and [FRI-based](https://drops.dagstuhl.de/opus/volltexte/2018/9018/pdf/LIPIcs-ICALP-2018-14.pdf).

Commitment schemes are binding and hiding:

  1. **Binding**: The prover can not change the polynomial she committed to.
  1. **Hiding**: The verifier can not deduce which is the committed polynomial by only looking at the commitment.
