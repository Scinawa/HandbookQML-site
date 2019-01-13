---
id: 1
layout: chapter
chapter: 7 
title: Rewriting the swap test
comments: true 
status: done
tags: qml, other
description: A simple exercice that shows the equivalence between the swap test and a measurement in the Bell's basis.
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
permalink: rewriting
---


Some weeks ago I stumbled upon a nice paper by Garcia-Escartin and
Chamorro-Posada (2013). There, they show the equivalence between a
widely used circuit in quantum information, called the swap test, and a
phenomena that goes under the name of Hong-Ou-Mandel effect. In their
work, they rewrote the circuit of the swap test using less gates and
with no ancilla qubit. I find this fact pretty interesting, and I think
it’s worth sharing it with you. It gives us a new interpretation of the
swap test, that is possible to prove with very simple gate’s
manipulations. More specifically here we show the equivalence of the
swap test and the circuit we use to measure in the Bell’s bases (an
Hadamard on the first qubit and a CNOT) - the same used to create an EPR
pair. Here we will work with single qubit register, but the result can
be extended to register with multiple qubits. Assuming to work with one
qubit register, let’s recall the original circuit of the swap test:
![image](/assets/rewriting_swap_test/Fig1.PNG)

The probability of reading $1$ is $ \frac{1 - \braket{a \| b} }{2} $ and
the probability of reading $0$ is $ \frac{1 + \braket{a \| b} }{2} $.

The probability of reading $1$ in the ancilla qubit of the original swap
test is equal to the probability of reading $11$ in the modified version
of the swap test in Figure 7.

Here is the proof. We start by rewriting the swap test as a series of
controlled not operations. Note that $x \oplus x = 0 $ and that
$x \oplus 0 = x$. It’s very simple to show that the swap gate can be
replaced with a series of CNOTs:
$$\ket{x}\ket{y} \to \ket{x}\ket{x \oplus y} \to \ket{x \oplus (x \oplus y)}\ket{x \oplus y} \to \ket{y}\ket{x}$$

![image](/assets/rewriting_swap_test/Fig2.PNG)

We know that A NOT gate is just a $Z$ gate on another rotation axis. The
rotation axis can easily be changed by two surrounding Hadamard’s gate.

![image](/assets/rewriting_swap_test/Fig3.PNG)

The CCZ gate is pretty agnostic with respect to the target or control
qubit, so we can put the $Z$ rotation on any of the control qubit.

![image](/assets/rewriting_swap_test/Fig4.PNG)

In this circuit we note that some of the gates we are applying are
actually useless for the measurement on the ancilla qubit, and we can
remove them from the circuit.

![image](/assets/rewriting_swap_test/Fig5.PNG)

Again, we use the equivalence between the $X$ gate $HZH$ gate.

![image](/assets/rewriting_swap_test/Fig6.PNG)

We note that we could remove the ancilla qubit, and measure the other
two qubits instead. This is possible thanks to the principle of deffered
measurement. The probability of reading $1$ in the ancilla qubit is
equivalent to the probability of reading $1$ in both the qubit $\ket{a}$
and $\ket{b}$. We don’t mind measuring the two qubit since after
measuring the ancilla qubit we cannot use $\ket{a}$ and $\ket{b}$
nevertheless. So here we get the final circuit.

![image](/assets/rewriting_swap_test/Fig7.PNG)

This equivalence might be useful when we need to optimize a circuit and
we have to reduce both the number of gates and the number of ancilla
qubit. This result gives us a nice intuition on the behavior of the swap
test when the two qubits are entangled. But beware of remote hacking! Be
sure to run the swap test only on non entangled data, otherwise you
might get unexpected results! Take for example the Bell’s sates. $1$ of
the four Bell’s basis for 2 qubit gates will pass the test
($\frac{\ket{01} - \ket{10}}{\sqrt{2}}$), but it’s pretty
counterintuitive, since the first and the second qubit are always
different. Therefore, we should use the swap test only with
non-entangled data! Entanglement, along with its usefulness in quantum
protocols and computation brings much troubles. Indeed it’s because of
entanglement that bit commitment is not possible using quantum
resources, and it’s because of entanglement that we can attack position
based encryption schemes. And that’s it. Hope you enjoyed it as much as
I did. To extend this equivalence to multi-qubit register, look at the
paper!

<div id="refs" class="references">

<div id="ref-garcia2013swap">

Garcia-Escartin, Juan Carlos, and Pedro Chamorro-Posada. 2013. “Swap
Test and Hong-Ou-Mandel Effect Are Equivalent.” *Physical Review A* 87
(5). APS: 052330.

</div>

</div>
