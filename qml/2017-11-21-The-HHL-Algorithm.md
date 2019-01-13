---
id: 1
layout: chapter
chapter: 7 
title: The HHL algorithm
comments: true 
status: done
tags: qml, algos
description: The (first) algorithm for inverting a matrix and solving linear system of equation. 
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---


A linear system of $N$ equations can be represented in matrix form as
$A\vec{x}=\vec{b}$. Its solution is defined as $\vec{x}=A^{-1}\vec{b}$.
This tells us that if we want to get the solution vector $\vec{x}$, we
should be able to invert the matrix $A$. Classically inverting a matrix
can be done in polynomial time, usually with algorithms that scale
between the square and the cube in the dimension of the system. HHL is a
quantum algorithm that allows to create a quantum state proportional to
the solution $\ket{A^{-1}\vec{b}}$ in time $polylog(N)$. This will give
us an exponential speedup with respect to classical algorithms, but it
will introduce a time dependency on other factors, such as the sparsity
of the matrix or the conditioning number.\
Let’s recall some notions from linear algebra. Given a Hermitian matrix,
we are also given a[^1] set of eigenvectors $\{ \vec{\varphi_i} \}$
which happen to be a base for the space. We can thus express the vector
$b$ as a linear combination of eigenvectors:
$\vec{b}=\sum_{j}\beta_j\vec{\varphi_j}$. The solution of the system is
therefore $\vec{x} = \sum_j \beta_j \lambda_j^{-1} \vec{\varphi}_j$. The
idea for the quantum algorithm is to relay on this observations and get
the state:
$$\ket{x} = \sum_{j} \beta_j \lambda_j^{-1}  \ket{ \varphi_j}  = \ket{A^{-1}\vec{b}}$$

If our matrix $A$ is not Hermitian, we can associate an Hermitian
matrix: $$A' =
  \begin{bmatrix}
    0 & A \\
    A^T & 0
  \end{bmatrix}$$ In this form, the multiplying a vector $x$ can be done
in this way: $(Ax, 0) = A' (0, x)$.\
A few initial remarks now: creating a quantum state
$\ket{x} \in \mathbb{R}^{2^n}$ (where $n$ is the number of qubits) does
not means that we are “solving” the linear system of equation, as we
don’t have classical access to the solution. Indeed, doing quantum
tomography for recovering $\ket{x}$ would cost us $O(N)$, forcing us to
lose the exponential speedup. But we are just getting as an output a
normalized version of $\vec{x}$, that is $\ket{x}$. Indeed, there are a
few assumption which is better to state explicitly: Aaronson (2015)

1.  On the input vector we have the following restrictions:

    1.  there must be a fast way of getting $|b\rangle$ from the
        classical input vector $b$. If $b$ is made of $N$ components,
        than we will need $log_2(N)$ qubits to express
        $|b\rangle = \sum_{i=0}^{N-1} b_i |i\rangle$. Practically, we
        assume the existence of QRAM: an operator
        $B: |0\rangle \to |b\rangle$. I hope to write more about
        this soon.

    2.  The initial vector $b$ should be relatively uniform, otherwise
        it will contradict the impossibility of an exponential speedup
        for black-box quantum search Aaronson (2015).

2.  The matrix $A$ should be $s$-sparse on rows (or other
    efficiently-simulable kind of Hamiltonians). This is needed because
    Hamiltonians with a sparse matrix can be efficiently simulated, and
    the amount of time needed to simulate them grows linearly with the
    sparsity $s$.

3.  The conditioning number
    $\kappa = \frac{\lambda_{max}} { \lambda_{min}} = $ should be
    low, i.e. the matrix should be robustly invertible, because the
    asymptotic complexity grows linearly with $\kappa$. Singular values
    of $A$ should lie between $1/\kappa$ and $1$.

4.  We are not interested in the values of $x$ itself (i.e. the
    probability amplitudes of $\ket{x}$, but just in a measurement in a
    basis of choice: $ \braket{x|M|x} $.

There are many enhancements of HHL, where we can solve rectangular
matrices, over-determined and under-determined systems of equations,
dense matrices, and speedups obtained by applying amplitude
amplification techniques.

First step: Hamiltonian simulation and phase estimation
-------------------------------------------------------

The first step of HHL is the eigenvalue estimation of the unitary matrix
associated to the linear transformation $U_A=e^{iA}$. Remember that
given an hermitian matrix $A$, there is an isomorphism between it’s
(real) eigenvalues and the eigenvalues of the unitary matrix $U=e^{iA}$.
For each eigenvalue $\lambda_i$ of A there is an eigenvalue
$e^{i\lambda_i}$. Hamiltonian simulation is a procedure that, given a
time $t$ and an Hamiltonian $H$, allows us to perform a time unitary
evolution $U$ associated to $H$ for a given state $|\psi\rangle$ for a
given time $t$:

$$U|\psi{0\rangle} = e^{-iHt}|\psi(0)\rangle = |\psi(t)\rangle$$

Controlled operations of $U$ allows us to write the eigenvalues of $U$
in the phase of our quantum computer. As usual, we use an index register
$\sum_{i}^{O(\epsilon)} |i\rangle$ with a uniform superposition and use
this register the apply the controlled $U$. Let’s imagine
$K \propto O(\varepsilon)$ is the chosen precision for phase estimation.
That will allow us to build the following mapping:

$$\sum_{k \in [K]}  \sum_{j \in [N]} |k\rangle \beta_j|\varphi_j\rangle \to \sum_{k \in [K] } \sum_{j \in [N]} e^{2\pi i k \lambda_i/N}|k\rangle\beta_j|\varphi_j\rangle$$

Using QFT$^{-1}$ we perform a phase estimation as usual:
$$\sum_{k \in [K]}\sum_{j \in [N]} e^{2\pi i k \lambda_i/N}|k\rangle\beta_j|\varphi_j\rangle \to \sum_{j \in [N]}\beta_{j}|\varphi_j\rangle|\tilde{\lambda_j\rangle}$$

The idea is to use quantum phase estimation allows to calculate
eigenvalues and eigenvector of the Hermitian operator associated to a
matrix $A$. We need Hamiltonian simulation in order to encode
efficiently the eigenvalues as a phase. Phase estimation is applied
next, writing an approximation of the eigenvalues in a register:
$|\tilde{\lambda_j\rangle}$.

Second step: controlled rotations
---------------------------------

The second step is where the magic happen. Note that multiplying each
eigenvector by its the inverse of an eigenvalue is not unitary
transformation, so we have to find some trick. The problem is solved in
by introducing the right non linear operation: a measurement on an
ancilla qubit. We adjoin a single qubit register, and we perform an
operation controlled on the eigenvalues estimated in the previous step.
Here, $C=\lambda_{min}$:

$$|\lambda_j\rangle|0\rangle \to |\lambda_j\rangle\otimes \left( \sqrt{1-(\frac{C}{\lambda_j})^2}|1\rangle + \frac{C}{\lambda_j}|0\rangle \right)^A$$

Measuring the $A$ register, we make the rest of the state into the
subspace consistent with our observation. That’s a neat trick that allow
us to “move out” a value inside a ket in the “outer world” in a
meaningful way. To get rid of the register with the eigenvalues
$|\lambda_j\rangle$, we run eigenvalue estimation in reverse, in order
to be left with the state
$$\sum_{j} \beta_j C\lambda_j^{-1} |\psi_j\rangle |1\rangle +|G\rangle|0\rangle =  \sum_{j} C\beta_j U_A^{-1} |\psi_j\rangle \ket{1} + \ket{G}\ket{0} = C\ket{x}\ket{0} +\ket{G}\ket{1}.$$

Now we measure the ancilla qubit.

1.  If we observe $|1\rangle$, the new state of the system is
    $$\sum_{j}\beta_jC\lambda_j^{-1}|\varphi_j\rangle \propto |x\rangle$$
    The probability of observing $|1\rangle$ is:
    $ p(|1\rangle) =C^2 |||x\rangle||^2$. In this step we have
    introduced a dependency on the conditioning number:
    $$p(|1\rangle) = \sum_{j} |\frac{\beta_jC}{\lambda_y}|^2 \geq |\frac{C}{\lambda_{max}}|^2= \frac{1}{\kappa^2}$$

2.  If we observe $|0\rangle$ we start again from step 1.

It is possible to use amplitude amplification on $|1\rangle$ to increase
by a factor of a square root the dependency on $\kappa^{-2}$.

Complexity analysis
===================

The cost of eigenvalue estimation with error less than $\epsilon$, is
$O(log(N)s^{2} + log(1/\varepsilon))$. To read $|1\rangle$ in the second
step, we should repeat the algorithm $O(\kappa^2)$ times. I extend a
little bit **???**, which list the complexity of all the version of
quantum algorithm for solving linear system of equations.

-   The original version Aram W. Harrow, Hassidim, and Lloyd (2009), we
    have: $$O(\kappa^{2}log(N)s^{2} / \epsilon)$$
    $$\tilde{O}(\kappa T_B + log (N) s^2 \kappa^2 T_A / \varepsilon)$$

-   In Ambainis (2010), where he used a technique called variable time
    amplitude amplification in order to decrease the running time, by
    applying a particular flavor of Grover algorithm in order to reduce
    the dependency on $\kappa$ from $\kappa^3$ to $\kappa^2$.
    $$\tilde{O}(\kappa T_B + log(N)s^2 \kappa T_A / \varepsilon^3)$$
    This results is based upon a previous work of Childs, Kothari, and
    Somma (2015) in precision for simulation sparse Hamiltonians. I
    think results might have changed since Hao Low and Chuang (2016):
    the last result I am aware of on Hamiltonian simulation.

-   In Childs, Kothari, and Somma (2015) they improved the dependency on
    the error: going from a polynomial dependency to a polylog. The idea
    is to avoid the QFT, whose dependency on error cannot be improved.

-   In Wossnig, Zhao, and Prakash (2017), instead of using Hamiltonian
    simulation, they use singular value estimation: a procedure that use
    QRAM to create a register proportional to the superposition of the
    singular values of a matrix. Their complexity is
    $O(\kappa^2\sqrt{N}\times polylog(N)/\varepsilon)$ for dense
    matrices, and $O(\kappa^2||A||_F \times polylog(N)/\varepsilon)$.
    This idea is based on the result of Kerenidis and Prakash (2016).

Conclusions
===========

This was a pretty significant result in quantum algorithmic, that made
possible many of the first results in quantum machine learning. For
instance, this algorithm can be used to performing least-squares
estimation of a model Aram W Harrow (2014). You are given a matrix
$A \in R^{n \times p}$ with $n \geq p$, and $b \in \mathbb{R}^n$. You
are asked to find the best solution $x$ with the constrain that
$$\operatorname*{arg\,min}_{xc \in \mathbb{R}^p} ||Ax -b||.$$ To relate
linear system of equations and function minimization, consider the
following function $f(x) = x^TAx-x^Tb $. Then, the gradient of the
function is $\nabla f(x) = Ax-b $ therefore finding the minimum of the
function (with additional hypotesis of ML, such as convexity,
smoothness, etc..) reduces to solving a linear system of equations. This
is a well-known in Machine Learning, and some qML algorithms started to
use this subrutine soon after it has been crated. For instance in data
fitting: Wiebe, Braun, and Lloyd (2012), Rebentrost, Mohseni, and Lloyd
(2014) and other things like differential equations Berry (2014). For an
useful answer on stack-overflow, refer to Vega (n.d.).\
It’s worth adding that in the original paper, they use a particular
initial state for the control register, in order to minimize some error
studied in the paper’s appendix. Many useful information can be found
in: Melkebeek (2010), Aram W. Harrow, Hassidim, and Lloyd (2009), and
Lloyd (n.d.).

<div id="refs" class="references">

<div id="ref-Aaronson2015ReadPrint">

Aaronson, Scott. 2015. “Read the fine print.” *Nature Physics* 11 (4):
291–93. doi:[10.1038/nphys3272](https://doi.org/10.1038/nphys3272).

</div>

<div id="ref-Ambainis2010VariableEquations">

Ambainis, Andris. 2010. “Variable time amplitude amplification and a
faster quantum algorithm for solving systems of linear equations.”

</div>

<div id="ref-berry2014high">

Berry, Dominic W. 2014. “High-Order Quantum Algorithm for Solving Linear
Differential Equations.” *Journal of Physics A: Mathematical and
Theoretical* 47 (10). IOP Publishing: 105301.

</div>

<div id="ref-Childs2015QuantumPrecision">

Childs, Andrew M, Robin Kothari, and Rolando D Somma. 2015. “Quantum
linear systems algorithm with exponentially improved dependence on
precision.”

</div>

<div id="ref-HaoLow2016HamiltonianQubitization">

Hao Low, Guang, and Isaac L Chuang. 2016. “Hamiltonian Simulation by
Qubitization.”

</div>

<div id="ref-Harrow2014ReviewEquations">

Harrow, Aram W. 2014. “Review of Quantum Algorithms for Systems of
Linear Equations,” December, 2–4. <http://arxiv.org/abs/1501.00008>.

</div>

<div id="ref-Harrow2009QuantumEquations">

Harrow, Aram W., Avinatan Hassidim, and Seth Lloyd. 2009. “Quantum
Algorithm for Linear Systems of Equations.” *Physical Review Letters*
103 (15): 150502.
doi:[10.1103/PhysRevLett.103.150502](https://doi.org/10.1103/PhysRevLett.103.150502).

</div>

<div id="ref-Kerenidis2016QuantumSystems">

Kerenidis, Iordanis, and Anupam Prakash. 2016. “Quantum Recommendation
Systems.”

</div>

<div id="ref-lloydyoutube">

Lloyd, Seth. n.d. “Quantum Algorithm for Solving Linear Equations.”

</div>

<div id="ref-Melkebeek2010Lecture12Equations">

Melkebeek, Instructor Dieter Van. 2010. “Lecture12 : Order Finding
‘Solving’ Linear Equations,” 1–4.
[http://pages.cs.wisc.edu/{\\\~{}}dieter/Courses/2010f-CS880/Scribes/12/lecture12.pdf](http://pages.cs.wisc.edu/{\~{}}dieter/Courses/2010f-CS880/Scribes/12/lecture12.pdf).

</div>

<div id="ref-Rebentrost2014QuantumClassification">

Rebentrost, Patrick, Masoud Mohseni, and Seth Lloyd. 2014. “Quantum
support vector machine for big data classification.” *Physical Review
Letters*.
doi:[10.1103/PhysRevLett.113.130503](https://doi.org/10.1103/PhysRevLett.113.130503).

</div>

<div id="ref-otherHHLuse">

Vega, Juan Bermejo. n.d. “Applications of Hhl’s Algorithm for Solving
Linear Equations.”

</div>

<div id="ref-Wiebe2012QuantumFitting">

Wiebe, Nathan, Daniel Braun, and Seth Lloyd. 2012. “Quantum Algorithm
for Data Fitting.” *Physical Review Letters* 109 (5): 050505.
doi:[10.1103/PhysRevLett.109.050505](https://doi.org/10.1103/PhysRevLett.109.050505).

</div>

<div id="ref-wossnig2017quantum">

Wossnig, Leonard, Zhikuan Zhao, and Anupam Prakash. 2017. “A Quantum
Linear System Algorithm for Dense Matrices.” *ArXiv Preprint
ArXiv:1704.06174*.

</div>

</div>

[^1]: very convenient - since they are orthogonal. Generalized
    eigenvectors are not orthogonal
