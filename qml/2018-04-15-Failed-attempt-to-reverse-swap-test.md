---
 layout: chapters
 comments: true
 tags: qml, other
author:
- Alessandro Scinawa Luongo
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
title: Trying to reverse the swap test
permalink: failed
---

This post has born from an attempt of finding a reversible circuit for
computing the swap test: a circuit used to compute the inner product of
two quantum states. This circuit was originally proposed for solving the
state distinguishably problem, but as you can imagine is very used in
quantum machine learning too. Before starting, let’s note one thing. A
reversible circuit for the swap test implies that we are able to
recreate the two input states. Conceptually, this should be impossible,
because of the no cloning theorem. With a very neat observation we can
realize that we are not even able to preserve one of the states.

There is no unitary operator $U\ket{x}\ket{y}$ that allows you to
estimate the scalar product between two states $x,y$ as $\braket{x|y}$
using only one copy of $\ket{x}$.

By absurd. Assume this unitary exists. Than it would be possible to
estimate the scalar product between $\ket{x}$ and all the base states
$\ket{i}$. (basically doing tomography for the state). This is a way of
recover classically the state of $\ket{x}$. By knowing $\ket{x}$, we
could recreate as many copies as we want of $\ket{x}$. Therefore, we
could use this procedure to clone a state. This is prevented by the
no-cloning theorem.

Let’s see what happens if we try to reverse it.

![image](/assets/reverse_swap.png)

It is good to know that the circuit in Figure \[conservative\] is
inspired by the proof $BPP \subseteq BQP$. The idea is the following: if
after a swap test, and before doing any measurement on the ancilla
qubit, we do a CNOT on a second ancillary qubit, and then execute the
inverse of the swap test. Being the swap test self-inverse operator, it
simply means that we apply the swap test twice. Let’s start the
calculations from the CNOT on the second ancilla qubit.

$$\frac{1}{2} \Big[ \left( \ket{ab} + \ket{ba} \right)\ket{00} +  \left( \ket{ab} - \ket{ba} \right)\ket{11}  \Big]  \xrightarrow{\text{H}}$$

$$\frac{1}{2} \Big[ \left( \ket{ab} + \ket{ba} \right)\ket{+0} +  \left( \ket{ab} - \ket{ba} \right)\ket{-1}  \Big]  \xrightarrow{\text{SWAP}}$$

$$\frac{1}{2} \left[
\frac{1}{\sqrt{2}} \Big[ \Big( \ket{ab} + \ket{ba} \Big) \ket{0} + \Big( \ket{ab} + \ket{ba} \Big) \ket{1} \Big] \ket{0} + \frac{1}{\sqrt{2}} \Big[ \Big( \ket{ab} - \ket{ba} \Big) \ket{0} - \Big( \ket{ba} - \ket{ab} \Big) \ket{1} \Big] \ket{1}
\right] =$$

$$\frac{1}{2} \left[
\left[ 2\left( \ket{ab} + \ket{ba} \right)\ket{+} \right] \ket{0} +
\left[ 2\left( \ket{ab} - \ket{ba} \right)\ket{+} \right] \ket{1}
\right] \xrightarrow{\text{H}}$$

$$\frac{1}{2} \left[
\frac{1}{\sqrt{2}} \left[ 2\left( \ket{ab} + \ket{ba} \right)\ket{0} \right] \ket{0} +
\frac{1}{\sqrt{2}} \left[ 2\left( \ket{ab} - \ket{ba} \right)\ket{0} \right] \ket{1}
\right].$$

$$p(\ket{0}) = \frac{1}{4}\Big( 2 + 2 |\braket{ab|ba}|\Big) =  \frac{ 1+ \braket{ab|ba}}{2} = \frac{ 1+ |\braket{a|b}|^2}{2}$$
And therefore $p(\ket{1})$ is $\frac{ 1- |\braket{a|b}|^2}{2}$ as in the
original swap test. So, the result is the same, but as in the original
swap test, the register are pretty entangled, therefore we haven’t
reversed our swap.

Here I have applied the rules:

-   $ (A\otimes B)^{\dagger} = A^{\dagger} \otimes B^{\dagger} $

-   $ \left( \bra{\phi} \otimes \bra{\psi} \right) \left( \ket{\phi} \otimes \ket{\psi} \right) = \braket{\psi, \psi} \otimes \braket{\phi, \phi}$

You may have noted that this circuit is very similar to circuit that you
obtain if you perform amplitude amplification Brassard et al. (2000) on
the swap test. The swap circuit is the algorithm $A$ that produces
states with a certain probability distribution, and the CNOT is the
unitary $U_f$ that is able to recognize the “good” states from bad
states. By setting the second ancilla qubit to $\ket{+}$ we would be
able to write on the phase of our state some useful information to
recover with a QFT later on. That’s very cool, since amplitude
amplification allows us to decrease quadratically the computational
complexity of the algorithm with respect to the error in the estimation
of the amplitude of the ancilla qubit.

<div id="refs" class="references">

<div id="ref-brassard2002quantum">

Brassard, Gilles, Peter Høyer, Michele Mosca, and Alain Tapp. 2000.
“Quantum Amplitude Amplification and Estimation.” *ArXiv Preprint
Quant-Ph/0005055*.

</div>

</div>
