[ToC]





# The Arithmetic State Machine
As a secondary state machine, the Arithmetic State Machine has the executor part (the Arithmetic SM Executor) and an internal Arithmetic PIL (program) which is a set of verification rules, written in the PIL language. The Arithmetic SM Executor is written in two versions; Javascript and C/C++.



The Polygon Hermez Repo is here  [https://github.com/0xPolygonHermez](https://github.com/0xPolygonHermez)

**Arithmetic SM Executor**: [sm_arith folder](https://github.com/0xPolygonHermez/zkevm-proverjs/tree/main/src/sm/sm_arith)

**Arithmetic SM PIL**: [arith.pil](https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/pil/arith.pil) 




## Introduction

The Arithmetic State Machine (SM) is one of the six secondary state machines receiving instructions from the Main SM Executor. As a secondary state machine, the Arithmetic SM has the executor part (the Arithmetic SM Executor) and an internal Arithmetic PIL program written in the PIL language.

The main purpose of the Arithmetic SM is carry out elliptic curve arithmetic operations, such as Point Addition and Point Doubling.





### Standard Elliptic Curve Arithmetic



Consider an elliptic curve $E$ defined by $y^2 = x^3 + ax + b$ over the finite field $\mathbb{F} = \mathbb{Z}_p$, where $p$ is the prime,

\begin{aligned}
p = 2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 -2^6 - 2^4 - 1.
\end{aligned}

Set the coefficients $a = 0$ and $b = 7$, so that $E$ reduces to
\begin{aligned}
y^2 = x^3 + 7.
\end{aligned}



#### Field Arithmetic 

Consider points $( x_1, y_1)$ , $( x_2, y_2)$, and $( x_3, y_3)$ on $E$.

Here, $y_2$ and $y_3$ are the result of performing field arithmetic over $x_1,y_1$ and $x_2$. That is,
$$
x_1 \cdot y_1 + x_2 = y_2 \cdot 2^{256} + y_3.
$$
Note that,

1. If $y_1$ is set to $1$, the above equation represents field addition. 
2. Similarly, if $x_2$ is set to $0$, then the equation represents field multiplication.



#### Elliptic Curve Point Addition 

Given two points, $P = (x_1,y_1)$ and  $Q = (x_2,y_2)$, on the curve $E$ with $x_1 \neq x_2$, the point $P+Q = (x_3,y_3)$  is computed as follows,


\begin{aligned}
x_3 &= s^2 - x_1 - x_2,\\
y_3 &= s (x_1 - x_3) - y_1
\end{aligned}

where
\begin{aligned}
s = \dfrac{y_2 - y_1}{x_2 - x_1}.
\end{aligned}



#### Elliptic Curve Point Doubling 

Given a point $P = (x_1,y_1)$ on the curve $E$ such that $P \neq \mathcal{O}$, the point $P+P = 2P =
(x_3,y_3)$ is computed as follows,

\begin{aligned}
x_3 &= s^2 - 2x_1,\\
y_3 &= s (x_1 - x_3) - y_1,
\end{aligned}

where
\begin{aligned}
s = \dfrac{3x_1^2}{2y_1}.
\end{aligned}


***Remark***:
Since the above Elliptic Curve operations are implemented in the PIL language, it is more convenient to express them in terms of the constraints they must satisfy. These constraints are:

\begin{aligned}
\text{EQ}_0 \colon \quad &x_1 \cdot y_1 + x_2 - y_2 \cdot 2^{256} - y_3
= 0, \\
\text{EQ}_1 \colon \quad &s \cdot x_2 - s \cdot x_1 -y_2 + y_1 + q_0
\cdot p = 0, \\
\text{EQ}_2 \colon \quad & 2 \cdot s \cdot y_1 - 3 \cdot x_1 \cdot x_1 +
q_0 \cdot p = 0, \\
\text{EQ}_3 \colon \quad & s \cdot s - x_1 - x_2 - x_3 + q_1 \cdot p = 0, \\
\text{EQ}_4 \colon \quad & s \cdot x_1 - s \cdot x_3 - y_1 - y_3 + q_2
\cdot p = 0,
\end{aligned}

where $q_0,q_1,q_2 \in \mathbb{Z}$, implying that these equations hold true over the integers. 

This approach is taken in order avoid having to compute divisions by $p$.



Note also that only three possible computation scenarios arise:

1. $\text{EQ}_0$ is activated while the rest are deactivated,
2. $\text{EQ}_1$, $\text{EQ}_3$ and $\text{EQ}_4$ are activated but $\text{EQ}_0$ and $\text{EQ}_2$ are deactivated, 
3. $\text{EQ}_2$, $\text{EQ}_3$ and $\text{EQ}_4$ are activated and $\text{EQ}_0$ and $\text{EQ}_1$ are deactivated.

Since at most, one of $\text{EQ}_1$ and $\text{EQ}_2$ are activated in any scenario, we can afford "sharing'' the same $q_0$ for both.


Motivated by the implemented operations, the Arithmetic SM is composed of 6 registers 
\begin{aligned}
x_1,\ y_1,\ x_2,\ y_2,\ x_3,\ y_3.
\end{aligned}

Each of these registers is composed of $16$ sub-registers of $16$-bit ($2$ byte) capacity, adding up to
a total of $256$ bits per register. 

There is also a need to provide $s$ and $q_0,q_1,q_2$, which are also $256$-bit field elements. 



### How The Operations Are Performed



Compute the previous operations at $2$-byte level. 

This means that if, for instance, one is performing the multiplication of $x_1$ and $y_1$, at the first clock $x_1[0] \cdot y_1[0]$ is computed. 

Then, $(x_1[0] \cdot y_1[1]) + (x_1[1] \cdot y_1[0])$ is computed in the second clock, followed by $(x_1[0] \cdot y_1[2]) + (x_1[1] \cdot y_1[1]) + (x_1[2] \cdot y_1[0])$  in the third, and so on. 

As depicted in [Figure 1](\ref{eq:school}), this process is completely analogous to the schoolbook multiplication. However, it is performed at $2$-byte level, instead of at decimal level.


![School Multiplication Example](fig1-sch-mlt-eg.png)
<div align="center"><b> School Multiplication Example </b></div>

<!-- <p align="center"><img src="fig1-sch-mlt-eg.png" width="800" /></p>
<div align="center"><b> Figure 1: School Multiplication Example</b></div>
 -->


Use the following notation;
$$
\begin{aligned}
\mathbf{eq\ } &= x_1[0] \cdot y_1[0] \\
\mathbf{eq'} &= x_1[0] \cdot y_1[1] + x_1[1] \cdot y_1[0]
\end{aligned}
$$
But then, the carry generated by $\mathbf{eq}$ has to be taken into account by $\mathbf{eq'}$.



Going back to our equations; $\text{EQ}_0, \text{EQ}_1, \text{EQ}_2, \text{EQ}_4$; let's see how the operation is performed in $\text{EQ}_0$. 

First, we compute $\mathbf{eq}_0 = (x_1[0] \cdot y_1[0]) + x_2[0] -
y_3[0]$. 

Second, we compute $\mathbf{eq}_1 = (x_1[0] \cdot y_1[1]) + (x_1[1]
\cdot y_1[0]) + x_2[1] - y_3[1]$. 

Third, $\mathbf{eq}_2 = (x_1[0] \cdot y_1[2]) + (x_1[1] \cdot y_1[1]) + (x_1[2] \cdot y_1[0]) + x_2[2] - y_3[2]$.

This is continued until one reaches the computation, $\mathbf{eq}_{15} =
(x_1[0] \cdot y_1[15]) + (x_1[1] \cdot y_1[14]) + \dots + x_2[15] -
y_3[15]$. 

This is the time when $y_2$ come into place. 

Since we have filled the first $256$ bits of the result of the operation (and the result can be made of more than $256$ bits) we need a new register to place the result from this point. We change the addition of $x_2[i] -
y_3[i]$ by $-y_2[i]$. 

Therefore, we obtain that 

$\mathbf{eq}_{16} = (x_1[1]
\cdot y_1[15]) + (x_1[2] \cdot y_1[14]) + \dots - y_2[0]$, $\mathbf{eq}_{17} =
(x_1[2] \cdot y_1[15]) + (x_1[3] \cdot y_1[14]) + \dots - y_2[1]$ and so on. 

Continuing until the last two 

$\mathbf{eq}_{30} = (x_1[15] \cdot y_1[15]) - y_2[14]$,
$\mathbf{eq}_{31} = -y_2[15]$. 



The full list can be found in Appendix A. 


Now, notice that the $\mathbf{eq}_i$'s do not care about the carry they generate. That means, if $\mathbf{eq}_i = 10$, then what we really want the result to be is $0$ and save $1$ as a carry for the next operation. To express this fact as a constraint, we say that the following has to be satisfied:
$$
\mathbf{eq} + \text{carry} = \text{carry}' \cdot 2^{16},
$$
where $\text{carry}$ represents the carry taken into account in the actual clock, and $\text{carry}'$ represents the carry generated by the actual operation.



***Remark***:
A technicality is that $\text{carry}$ is subdivided into two other $\text{carry}_L$ and $\text{carry}_H$ such that:
$$
\text{carry} = \text{carry}_L + \text{carry}_H \cdot 2^{18}.
$$



