This last section wraps the document by introducing some advanced features that PIL supports, such as permutation checks over multiple (possibly distinct) domains.

## Public Inputs

Public inputs are values of a polynomial that are known prior to the execution of a state machine. In the following example, the public input $\texttt{publicInput}$ is set to be the first element of the polynomial $\texttt{a}$ and a colon "$:$" is used to indicate this to the compiler (see line 12 in the code excerpt below).

![Public Inputs PIL Example](figures/fig19-pil-eg-pub-inpts.png)

<div align="center"><b> Code Excerpt 19: Public Inputs PIL Example </b></div>

Note here, the use of the Lagrange polynomial $L_1$ to create a constraint,

$$
L_1 \cdot (\texttt{a} - :\texttt{publicInput}) = 0.
$$

Whenever relevant, the constraint enforces the value of $\texttt{a}$ to be equal to $\texttt{publicInput}$.

## Permutation Check

In this example we use the $\texttt{is}$ keyword to denote that the vectors $[\texttt{sm1.a},\texttt{sm1.b},\texttt{sm1.c}]$ and $[\texttt{sm2.a}, \texttt{sm2.b}, \texttt{sm2.c}]$ are a permutation of each other, seen as evaluations over the designated domain.

![Permutation Check PIL Example](figures/fig20-pil-eg-prm-chck.png)

<div align="center"><b> Code Excerpt 20: Permutation Check PIL Example </b></div>

This constraint becomes useful to connect distinct state machines, since it is forcing that polynomials belonging to different state machines are the same (up to permutation).

## Two Special Functionalities

Here are some vectors for which the $\texttt{in}$ and $\texttt{is}$ functionalities are designed for:

\begin{array}{ccc}
(3,2) & \text{in} & (1,2,3,4)\\
(1,5,5,5,8,1,1,2) & \text{in} & (1,2,4,5,8)\\
(3,2,3,1) & \text{is} & (1,2,3,3)\\
(5,5,6,9,0) & \text{is} & (6,5,9,0,5).
\end{array}

## The Connect Keyword

The $\texttt{connect}$ keyword is introduced to denote that the copy constraint argument is applied to $[\texttt{a},\texttt{b},\texttt{c}]$ using the permutation induced by $[\texttt{SA}, \texttt{SB}, \texttt{SC}]$.

![Connect Keywords PIL Example](figures/fig21-pil-eg-cnnct-kwrds.png)

<div align="center"><b> Code Excerpt 21: Connect Keywords PIL Example </b></div>

Naturally, the previous feature can be used to describe the correctness of an entire PlonK circuit in PIL:

![Plonk Circuit in PIL](figures/fig22-pil-eg-plnk-crct.png)

<div align="center"><b> Code Excerpt 22: Plonk Circuit in PIL </b></div>

## Permutation Check with Multiple Domain

Another important feature is the possibility to prove that polynomials of distinct state machines are the same (up to permutation) in a subset of its elements. This helps to improve efficiency when state machines are defined over subgroups of distinct size, since without this permutation argument one would need to equal the size of both polynomials.

PIL introduces this possibility by the introducing selectors that choose the subset of elements to be included in the permutation argument.

![Permutation Argument in PIL](figures/fig23-pil-eg-prm-argmnt.png)

<div align="center"><b> Code Excerpt 23: Permutation Argument in PIL </b></div>

Any combination of $\texttt{sel}$, $\texttt{not sel}$ and $\texttt{in}$, $\texttt{is}$ are available as permutation arguments. This leads to a total of $4$ possibilities.

Figure 18 depicts an example of the permutation multi-domain protocol, with one selector per table, that is, we prove that:

$$
\text{sel } \cdot [a,b,c] = \sigma\left(\text{sel' } \cdot [d,e,f]\right).
$$

![Permutation Multi-Domain Protocol](figures/fig18-prm-mlt-dom-prtcl.png)

<div align="center"><b> Figure 18: Permutation Multi-Domain Protocol </b></div>
