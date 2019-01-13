---
id: 1
layout: chapter
chapter: 7 
title: The HHL algorithm
comments: true 
status: done
tags: qml, tools
title: Hamiltonian simulation for computer scientists
description: Some very shord notes on how to think Hamiltonians and Hamiltonian simulation if you are a computer scientist with little background in physics. Based on Lecture Notes by Childs. 
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

These are my notes are on Childs' [Lecture notes](https://www.cs.umd.edu/~amchilds/qa/) {% cite ChildsLectureNotes %}, Section 5

Introduction to Hamiltonians
============================

The only way possible to start a chapter on Hamiltonian simulation would
be to start from the work of Feynman, who had the first intuition on the
power of quantum mechanics for simulating physics with computers. We
know that the Hamiltonian dynamics of a closed quantum system, weather
its evolution changes with time or not, is give by the
Schr<span>ö</span>dinger equation:

$$ \frac{d}{dt}\ket{\psi(t)} = H(t)\ket{\psi(t)} $$

Given the initial conditions of the system (i.e. $\ket{\psi(0)} $ ) is
it possible to know the state of the system at time $t: \ket{\psi(t)} = e^{-i (H_1t/m)}\ket{\psi(0)}$.

As you can imagine, classical computers are suppose to struggle simulating the system to get $ \ket{\psi(t)}$, since this equation describes the dynamics of any quantum system, and we don’t think (hope
:D ) classical computer can simulate that efficiently. But we know that quantum computers can help “copying” the dynamic of another quantum
system. Why would you be bothered?

Imagine you are a quantum machine learning scientist, and you have just found a new mapping between an optimization problem and an Hamiltonian
dynamics, and you want to use quantum computer to perform the optimization {% cite otterbach2017unsupervised %}. You expect a quantum computers to
run the Hamiltonian simulation for you, and then sample useful information from the resulting quantum sate. This result might be fed
again into your classical algorithm to perform ML related task, in a virtuous cycle of hybrid quantum-classical computation.

Or imagine you that you are a chemist, and you have developed an
hypothesis for the Hamiltonian dynamics of a chemical compound. Now you
want to run some experiments to see if the formula behaves according to
the experiments. Or maybe you are testing properties of complex
compounds you don’t want to synthesize. We can formulate the problem of
HS in this way:

##### Hamiltonian simulation problem
_Given a state $\ket{\psi(0)}$ and an Hamiltonian $H$, obtain a state $\ket{\psi(t)}$ such that $\ket{\psi(t)}:=e^{-iHt}\ket{\psi(0)}$ *FILL* for some norm (usually trace norm)._

This leads us to the definition of efficiently simulable Hamiltonian:

##### Def: Efficient Hamiltonian simulation
_Given a state $\ket{\psi(0)}$ and an Hamiltonian $H$ acting on $n$ qubits, we say $H$ can be efficiently simulated if, $\forall t \geq 0, \forall \epsilon \geq 0$ there is a quantum circuit $U$ such that *FILL* < \epsilon$ using a number of gates that is polynomial in $n,t, 1/\epsilon$._ 

In the following, we suppose to have a quantum computer and quantum
access to the Hamiltonian $H$. Te importance of this problem might not
be immediately clear to a computer scientist. But if we think that every
quantum circuit is described by an Hamiltonian dynamic, being able to
simulate an Hamiltonian is like being able to run virtual machines in
our computer (This example actually came from a talk at IHP of Toby
Cubitt!). Remember that there’s a Theorem (No Fast-forward Theorem) that says that for an
Hamiltonian simulation problem, the number of gates is $\omega{t}$.
But concretely? What does it means to simulate an Hamiltonian of a
physical system? Let’s take the Hamiltonian of a particle in a
potential: $$H = \frac{p^2}{2m} + V(x)$$ We want to know the position of
the particle at time $t$ and therefore we have to compute
$e^{-iHt}\ket{\psi(0)}$

Some Hamiltonians we know to simulate efficiently
-------------------------------------------------

-   Hamiltonians that represent the dynamic of a quantum circuits (more
    formally, where you only admit local interactions between a constant
    number of qubits). This result is due to the famous Solovay-Kitaev Theorem (more about it before the References in this page).

-   Suppose you know how to efficiently change base of reference with a unitary $U$. Then, if the Hamiltonian can be efficiently applied that base, also
    $UHU$ can be efficiently applied. Proof:
    $e^{-iUHU^\dagger t} = Ue^{-iH t}U^\dagger $.

-   If $H$ is diagonal in the computational basis and we can compute
    efficiently $\braket{a||H|a}$ for a basis element $a$. By linearity:
    $$\ket{a,0} \to \ket{a, d(a)} \to e^{-itd(a)} \otimes I \ket{a,d(a)t} \to e^{-itd(a)}\ket{a,0} = e^{-itH}\ket{a,0}$$

    (In general: if we know how to calculate the eigenvalues, we can
    apply an Hamiltonian efficiently.)

-   The sum of two efficiently simulable Hamiltonians is efficiently
    simulable using Lie product formula
    $$e^{-i (H_1 + H_2) t} = lim_{m \to \infty} ( e^{-i (H_1t/m)} + e^{-i (H_2t/m) t}   )^m$$
    We chose $m$ such that
    $$|| e^{-i (H_1 + H_2) t} - ( e^{-i (H_1t/m)} + e^{-i (H_2t/m) t}   )^m || \leq$$
    and this gives $m=(vt^2/\varepsilon)$ and
    $v=\max{ ||H_1||, ||H_2||}$. Using higher order approximation is
    possible to reduce the dependency on $t$ to $O(t^1+\delta)$ for a
    chosen $\delta$. (wtf!)

-   This facts can be used to show that the sum of polynomially many
    efficiently simulable Hamiltonians is simulable efficiently.

-   The commutator $[H_1, H_2]$ of two efficiently simulable Hamiltonian
    can be computed efficiently because:
    $$e^{-i[H_1, H_2]t} = lim_{m\to \infty} (e^{-iH_1\sqrt[]{t/m}}e^{-iH_2\sqrt[]{t/m}}e^{H_1\sqrt[]{t/m}}e^{H_1\sqrt[]{t/m}})^m$$
    which we believe, without having idea on how to check it. :/

-   If the Hamiltonian is sparse, it can be efficiently simulated. The
    idea is to pre-compute a edge-coloring of the graph represented by
    the adjacency matrix of the sparse Hamiltonian. (For each $H$ you
    can consider a graph $G=(V, E)$ such that its adjacency matrix $A$
    is $a_{ij}=1$ if $H_{ij} \neq 0$ ).

Recalling the example of a particle in a potential energy: its momentum
$$\frac{p^2}{2m}$$ is diagonal in the fourier basis (and we know how to
do a QFT), and the potential $V(x)$ is diagonal in the computational
basis, thus this Hamiltonian is easy to simulate.

*Exercise/open problem*: do we know any algorithm that might benefit the
efficient simulation of $[H_1, H_2]$? Childs in Childs (n.d.) claims he
is not aware of any algorithm that uses that.


As handy reference, since we used it in this post, here is the Solovay-Kitaev Theorem from pag 7 of {% cite ChildsLectureNotes %}
##### Theorem: Solovay-Kitaev Theorem
Fix two universal gate sets that are closed under inverses.  Then any t-gate circuit using one gate set can be implemented to precision $\epsilon$ using a circuit of $t poly(\log t )$ gates from other
set (indeed, there is a classical algorithm for finding this circuit in time $t poly(\log \frac{t}{\epsilon})$).



### References 

{% bibliography --cited %}
