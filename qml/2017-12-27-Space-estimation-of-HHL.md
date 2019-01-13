--- 
 layout: chapter 
 comments: true 
 tags: qml, other  
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
title: Space estimation for HHL
---

Let’s imagine that we are given a quantum computer with 100 logical
qubits, and let’s also assume that we have high gate fidelity (i.e.
applying a gate won’t introduce major errors in our computation). This
means that we can run all the algorithm that we want. An idyllic
situation like this won’t probably happen in the near future (let’s say
5 years). Even if now we have the first prototypes of quantum computers
with the first dozen of quibts, those qubits are not stable, and
therefore the computation we can do is pretty limited: in fact, these
prototypes aren’t able to perform error-free computations (there’s no
error correction yet), and that the computation won’t be “long” as we
want: we will be able to apply a limited number of gate before the
system decohere.

The question is the following: can we compete in solving linear system
of equations with a classical computer? Can we use it to run HHL
algorithm? Let’s recall that for HHL we need 1 qubit register for the
ancilla, a register for the output of phase estimation in the
Hamiltonian simulation (that will store the superposition of the
eigenvalues) and the rest of the qubit can store the input register. We
assume to have logical qubits in our comparison.

We’ll see what happen when we change precision for floating point
operations: 32, 64bit. The sparsity of the matrix is supposed to be
small. Since we want to be as close as possible as real cases, let’s
take a famous example of matrix considered sparse. The product-user
matrix that websites like Amazon, or Netflix use to run Recommendation
Algorithms: rows represent the users of the service, while columns are
the products. The rows are empty except where an user purchased a
specific product or watched a particular movie. Let’s say that an
educated guess for the sparsity of the matrix is $100$.

![image](/assets/HHL_resource_estimation/space_resource_estimation.png)

The upper horizontal line is an estimate of the space in TB of the
hard-disks for the whole Google (Cirrusinsight, n.d.) (13 EB), while the
lower one is an estimate for the storage need to store the images of
Google Maps (Mesarina, n.d.) (43 PB).

Let’s do an example to show what the software is plotting for $100$
qubits. In HHL we need an ancilla qubit. So we have 99 qubits. To get
$64$ bit precision, we need to allocate 64 qubits: this is the phase
estimation of the Hamiltonian simulation step. So now we are left with
just $35$ qubits. With $35$ qubits we can span a Hilbert space of
dimension $2^{35}$: this allow us to encode a vector of data with the
same number of components. Suppose our vector of known values has $64$
bits floating points: classically, the cost for storing this amount of
data we need $2^{35}*64$ bits, which are $0.549.5$TB (Terabytes).
Summing up the cost for storing a $2^{35} \times 2^{35}$ matrix with
sparsity $100$, we get $27$ TB.

Remember that each component of the vector will be encoded as
probability amplitude in our quantum register. This imply that our
precision in modifying a qubit need to grow, along with the number fo
qubit and the fidelity of the gates. Here we just focused on the
computational capabilities of a small quantum computer with respect to
HHL algorithm. Don’t forget that for HHL we will need to store the
matrix to invert as in the classical case (in form of QRAM or other
oracle). \[Here\](<https://github.com/Scinawa/space_estimation_hhl>) the
code for generating the plot.

<div id="refs" class="references">

<div id="ref-exagoogle">

Cirrusinsight. n.d. “How Much Data Does Google Store?”

</div>

<div id="ref-gmaps">

Mesarina, Malena. n.d. “How Much Storage Space Do You Need for Google
Maps?”

</div>

</div>
