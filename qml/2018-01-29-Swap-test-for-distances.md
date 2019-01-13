--- 
id: 2
layout: chapter
chapter: 1 
comments: true 
tags: qml, tools
title: Swap test for distances and inner products
description: Here we see how to use the swap test to calculate distances and inner products betewen vectors in our dataset.
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
permalink: swapdistances
---

Intro to swap test
------------------

What is known as *swap test* is a simple but powerful circuit used to
measure the “proximity” of two quantum states (cosine distance in
machine learning). It consists in a controlled swap operation surrounded
by two Hadamard gates on the controlling qubit. Repeated measurements of
the ancilla qubit allows us to estimate the probability of reading $0$
or $1$, which in turn will allow us to estimate $\braket{\psi|\phi}$.
Let’s see the circuit:

![image](/assets/swap_distances/swap_test.png)

It is simple to check that the state at the end of the execution of the
circuit is the following:

$$
\Big[\ket{\psi} \ket{\phi} + \ket{\phi} \ket{\psi} \Big]\ket{0} +\Big[\ket{\psi} \ket{\phi} - \ket{\psi} \ket{\phi} \Big] \ket{1}
$$

Thus, the probability of reading a $0$ in the ancilla qubit is:
$$P (\ket{0}) = \left( \frac{1+|\braket{\psi|\phi}|^2}{2} \right)$$ And
the probability of reading a $1$ in the ancilla qubit is:
$$P (\ket{1}) = \left( \frac{1-|\braket{\psi|\phi}|^2}{2} \right)$$

This means that if the two states are completely orthogonal, we will
measure an equal number of zero and ones. On the other side, if
$\ket{\psi} = \ket{\phi}$, then the probability amplitude of reading
$\ket{1}$ in the ancilla qubit is $0$. Repeating this operation a
certain number of time, allows us to estimate the inner product between
$\ket{\psi},\ket{\phi}$. Unfortunately, at each measurement we
irrevocably destroy the states, and we need to recreate them in order to
perform again the swap test. This is not much of a problem, if we have
an efficient way of creating $\ket{\psi}$ and $\ket{\phi}$. We can
informally state what the swap test consist with the following theorem.

###### Theorem: Swap test for inner products
Suppose you have access to unitary
$U_\psi$ and $U_\phi$ that allows you to create $\ket{\psi}$ and
$\ket{\phi}$, each of them requiring time $T(U_\psi)$ and $T(U_\phi)$.
Then, there is a circuit that allows to estimate inner products between
two states $\ket{x},\ket{y}$ in $O(T(U_\psi)T(U_\phi)\varepsilon^{-2})$ time.

_Proof: The correctnes of the circuit was sown before. This is the analysis of the running time. We recognize in the measurement on the ancilla qubit a random variable $X$ with Bernulli distribution with $p=(1+|\braket{\psi|\phi}|^2)/2$, and variance $p(1-p)$. The number of repetitions that are necessary to estimate the expected value $\bar{p}$
of $X$ with relative error $\epsilon$ is bounded by the Chernoff bound._

Swap test for distances between vector and center of a cluster
--------------------------------------------------------------

Now we are going to see how to use the swap test to calculate the
distance between two vectors. This section is entirely based on the work
of Lloyd, Mohseni, and Rebentrost (2013). There, they explain how to use
this subroutine to do cluster assignment and many other interesting
things in quantum machine learning. This was one of the first paper I
read in quantum machine learning, and I really wanted to understand
everything, so I tried to do the calculation myself. I think I have
found some typos in the original paper, so here you will find what I
think is the correct version. At the bottom of this post you will find
the calculations. In the following section we will assume that we are
given access to two unitaries $U : \ket{i}\ket{0} \to \ket{i}\ket{v}$
and $V : \ket{i}\ket{0} \to \ket{i}\ket{|v_i|}  $.

Let’s recall the relation between inner product and distance of
$\vec{u}, \vec{v} \in \mathbb{R}^n$. The inner product between two
vector is $\braket{ v, u } = \sum_{i} v_i u_i $, and the norm of a
vector is $ |v|= \sqrt{\langle v, v \rangle} $. Therefore, the distance
can be rewritten as:



$$ ||u-v|| = \sqrt{ \langle u-v, u-v \rangle } = \sqrt{\sum_{i} (u_i - v_i )^2 } = \sqrt{|u|^2 + |v|^2 -2 \langle u, v \rangle } $$

By setting $$ Z=|u|^2 + |v|^2 $$
it follows that:
$ ||u-v||^2 = Z \left( 1 - \frac{ 2 \langle u, v \rangle } {Z} \right) $.

As you may have guessed, to find the distance $||v-u||$ we will repeat the
necessary number of times the swap circuit in order to estimate the probability of a certain outcome. The task is now is to find the right states to create to measure a distance with this circuit. 

We first start by creating
$|\psi \rangle = \frac{1}{\sqrt{2}} \Big( \ket{0}\ket{u} + \ket{1}\ket{v} \Big)$
querying QRAM in $O(log(N))$ time, where N is the dimension of the
Hilbert space (the length of the vector of the data).\
Then we proceed by creating
$|\phi\rangle \frac{1}{\sqrt{Z}} \Big( |\vec{v}||0\rangle + |\vec{u}||1\rangle \Big) $
and and estimate $Z=|\vec{u}|^2 +  |\vec{v_j}|^2$. Remember that for two
vectors, $Z$ is easy to calculate, while in the case of a distance
between a vector and the center of a cluster then
$Z=|\vec{u}|+\sum_{i \in V} |\vec{v_i}|^2$. In this case, calculating
$Z$ scales linearly with the number of elements in the cluster, and we
don’t want that.

To create $\ket{\phi}$ and estimate $Z$, we have to start with another,
simpler-to-build $\ket{\phi^-}$ and make it evolve to $\ket{\phi}$. To
do so, we apply the following time dependent Hamiltonian for a certain
amount of time $t$ such that $t|\vec{v}|, t|\vec{u}| << 1$:

$$H = |\vec{u}|\ket{0}\bra{0}+|\vec{v}|\ket{1}\bra{1} \otimes \sigma_x$$
$$\ket{\phi^-} = \ket{-}\ket{0}$$

The evolution $e^{-iHt} \ket{\phi^-}$ for small $t$ will give us the
following state:
$$\Big( \frac{cos(|\vec{u}|t)}{\sqrt{2}}\ket{0} - \frac{cos(|\vec{v}|t)}{\sqrt{2}}\ket{1} \Big) \ket{0} - \Big( \frac{i sin(|\vec{u}|t)}{\sqrt{2}}\ket{0} - \frac{i sin(|\vec{v}|t)}{\sqrt{2}}\ket{1} \Big) \ket{1}$$

Reading the ancilla qubit in the second register, we should read $1$
with the following probability, given by small angle approximation of
the $sin$ function:

$$P(1) =   \lvert - \frac{i sin(|\vec{u}|t)}{\sqrt{2}} \rvert^2 + \lvert \frac{i sin(|\vec{v}|t)}{\sqrt{2}} \lvert^2   \approx |\frac{|\vec{u}|t}{\sqrt{2}}|^2 + | \frac{|\vec{v}|t}{\sqrt{2}} |^2 =  \frac{1}{2} \Big( |\vec{u}|^2t^2 + |\vec{v}|^2t^2 \Big) = Z^2t^2/2$$

Now we are almost ready to use the swap circuit. Note that our two
quantum register have a different dimension, so we cannot swap them.
What we can do instead is to swap the index register of $\ket{\phi}$
with the whole state $\ket{\psi}$. The probability of reading $1$ is:

$$\begin{split}
p(1) = \frac{2||\vec{u}||^2 + 2||\vec{v}||^2 - 4(u,v)}{8Z}
\end{split}$$

Conclusion
==========

We saw how to use a simple circuit to estimate things like inner product
and distance between two quantum vectors. We have assumed that we have
an efficient way of creating the states we are using, and we didn’t went
deep into explaining how. Given a $\epsilon > 0$, you can repeat the
previous circuit $O(\epsilon^{-2})$ times to have the desired precision.
Note the following thing: while calculating the value of $Z$ for two
vectors is easy, estimating it for calculating the distance between a
vector and the center of a cluster takes time linear in the number of
element in the superposition. Note that we can use amplitude estimation
in order to reduce the dependency on error to $O(\epsilon^{-1})$.

For the records:

##### Theorem: Chernoff Bounds
Let $X = \sum_{i=0}^n X_i $, where $X_i = 1$ with probability $p_i$ and $X_i=0$ with probability $1-p_i$. All $X_i$ are independent. Let $\mu = E[X] = \sum_{i=0}^n p_i$. Then:
-   $P(X \geq (1+\delta)\mu) \leq e^{-\frac{\delta^2}{2+\delta}\mu} $
    for all $\delta > 0$
-   $P(X \leq (1-\delta)\mu) \leq e^{\frac{\delta^2}{2}}$

##### Theorem: Chebyshev inequality
_Let $X$ a random variable with $E[X] = \mu$ and $Var[X]=\sigma_2$. For all $t > 0$:_

$$P(|X - \mu| > t\sigma) \leq 1/t^2$$

_If we substitute $k/\sigma$ on $t$, we get the equivalent version that we use to bound the error:_
$$P(|X - \mu|) \geq k) \leq \frac{\sigma^2}{k^2}$$

Calculations
------------

It’s time now to prove that our claim is true and to show some
calculation. After all the previous passages, this is the initial state:

$$\ket{0}\Big(  \frac{1}{\sqrt{Z}} \left( |\vec{u}|\ket{0} + |\vec{v}|\ket{1} \right)     \otimes \frac{1}{\sqrt{2}} (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)$$

We apply an Hadamard on the leftmost ancilla register:

$$\frac{1}{2\sqrt{Z}} \left[   \ket{0} \Big(   \left( |\vec{u}|\ket{0} + |\vec{v}|\ket{1} \right)     \otimes  (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)   +
                        \ket{1} \Big(  \left( |\vec{u}\ket{0} + \vec{v}\ket{1} \right)     \otimes  (\ket{0}\ket{u} + \ket{1}\ket{v} ) \Big)  \right] =$$

$$\begin{split}
= \frac{1}{2\sqrt{Z}} \Big[   \ket{0} \Big(  |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u}  + |v|\ket{11v} \Big) \\
\ket{1} \Big( |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u} + |v| \ket{11v} 
\Big)  \Big]
\end{split}$$

Controlled on the ancilla being $1$, we swap the second and the third
register:

$$\begin{split}
= \frac{1}{2\sqrt{Z}} \Big[   \ket{0} \Big(  |u|\ket{00u} + |u|\ket{01v} + |v|\ket{10u}  + |v|\ket{11v} \Big) \\
\ket{1} \Big( |u|\ket{00u} + |u|\ket{10v} + |v|\ket{01u} + |v| \ket{11v} \Big)  \Big]
\end{split}$$

Now we apply the Hadamard on the leftmost ancilla qubit again:

$$\begin{split}
= \frac{1}{2^{3/2}\sqrt{Z}} \Big[ 
|u|\ket{000u} + |u|\ket{001v} + |v|\ket{010u} + |v|\ket{011v} \\
+|u|\ket{100u} + |u|\ket{101v} + |v|\ket{110u} + |v|\ket{111v}  \\
+|u|\ket{000u} + |u|\ket{010v} + |v|\ket{001u} + |v|\ket{011v} \\
-|u|\ket{100u} - |u|\ket{110v} - |v|\ket{101u} - |v|\ket{111v}
\Big]
\end{split}$$

And now we check the probability of reading $\ket{1}$.

$$\begin{split}
p(1) = \frac{1}{2^{3}Z} (u\bra{v10} + v\bra{u01} - u\bra{v01} - v\bra{u10})\\
(u\ket{01v} + v\ket{10u} - u\ket{10v} - v\ket{01u})
\end{split}$$

$$\begin{split}
p(1) = \frac{2||u||^2 + 2||v||^2 - 4(u,v)}{8Z}
\end{split}$$

Thanks to IK and AG who checked :)

<div id="refs" class="references">

<div id="ref-Lloyd2013QuantumLearning">

1. Lloyd, Seth, Masoud Mohseni, and Patrick Rebentrost. 2013. “Quantum
algorithms for supervised and unsupervised machine learning.” *ArXiv*
1307.0411 (July): 1–11. <http://arxiv.org/abs/1307.0411>.

</div>

</div>
