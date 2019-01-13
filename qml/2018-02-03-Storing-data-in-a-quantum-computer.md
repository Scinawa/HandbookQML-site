--- 
id: 1
layout: chapter
chapter: 7 
title: Storing data in a quantum computer
comments: true 
status: todo
tags: qml, intro 
description: Here we see how we can encode classical data in a quantum computer, and why it is important in quantum machine learning.
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

We are going to see what does it mean to store/represent data on a
quantum computer. Is very important to know how, since knowing what are
the most common ways of encoding data in a quantum computer might pave
the way for the intuition in solving new problems. Let me quote an
article of 2015: Schuld, Sinayskiy, and Petruccione (2015): *In order to
use the strengths of quantum mechanics without being confined by
classical ideas of data encoding, finding “genuinely quantum” ways of
representing and extracting information could become vital for the
future of quantum machine learning*. 

Usually we store information in a classical data structure, and the assume to have quantum access to it.
In general, this quantum access consist of a query: an operation
$U\ket{i}\ket{0}\to \ket{i}\ket{\psi_i}$, where the first is called the
index register, and the second is a target register that holds the
information that you requested. To get an intuition of what the previous
sentence means, I borrow an intuitive example that I stole from a
youtube video of Seth Lloyd. Imagine that you have a source of photons -
which represent your query register - and you send one towards a CD. Due
to the duality wave-particle, you are actually hitting your CD with a
“thing” that is not anymore located deterministically as a single
particle in the space, but behaves as a wave. When the wave hits the
surface of the CD, it gets all the information stored in the little
holes of $0$s and $1$s, and gets reflected carrying on this information.
This wave represent the output of your query. (Sure, we assume the
interaction between the wave and the CD does not make the wave-function
collapse).\
Let’s start. As good computer scientist, let’s organize what we know how
to do by data types.

Scalars
=======

Integer: $\mathbb{Z}$
---------------------

Let’s start with the most simple “type” of date: the integers. Let
$m \in \mathbb{N}$. We take the binary expansion of $m$, and set the
qubits of our computer as the binary digits of the number. As example,
if your number’s binary expansion is $0100\cdots0111$ we can create the
state:
$\ket{x} =  \ket{0}\otimes \ket{1} \ket{0} \ket{0} \cdots \ket{0} \ket{1} \ket{1} \ket{1}$.
Formally, given $m$:

$$\ket{m} = \bigotimes_{i=0}^{n} m_i$$

Using superposition of states like these we might create things like
$\frac{1}{\sqrt{2}} (\ket{5}+\ket{9})$ or more involved convex
combination of states.\
The time needed to create this state is linear in the number of
bits/qubits. It might be used to get speedup in the number of query to
an oracle, like in (<span class="citeproc-not-found"
data-reference-id="Wiebe0QuantumModels">**???**</span>), or in general
where you aim at getting a speedup in oracle complexity using amplitude
amplification and similar. For negative integers, we might just use a
qubit more for the sign. (Don’t be tempted into saying that
$\ket{3}+\ket{3}=\ket{6}$. It’s not!)\

Rational: $\mathbb{Q}$
----------------------

As far as I know, in quantum computation / quantum machine learning,
there are some register with rational numbers, usually as $n$-bit
approximation of a reals between $0$ and $1$. In that case, just take
the binary expansion and use the previous encoding.

Reals: $\mathbb{R}$
-------------------

As before, if the number is between $0$ and $1$, use the previous
encoding. It’s pretty rare to store just a single number in
$\mathbb{R}$, and usually real numbers are encoded into amplitudes and
used when dealing with vectors in $\mathbb{R}^n$.

Vectors
=======

Binary vectors: $\\{0,1\\}^n$
---------------------------

Let $\vec{b} \in \{0,1\}^n$. As for the encoding used for the integers:

$$\ket{b} = \bigotimes_{i=0}^{n} b_i$$

As an example, suppose you want to encode the vector
$[1,0,1,0,1,0] \in \{0,1\}^6$, which is $42$ in decimal. This will
correspond to the $42$-th base of the Hilbert space where our qubits
will evolve. In some sense, we are not fully using the $C^{2^{n}}$
Hilbert space: we are only mapping a binary vector in a (canonical)
base. As a consequence, distances between points in the new space are
different.\
We can imagine some other encodings. For instance we can map a $0$ into
$1$ and $1$ as $-1$ (even if I don’t know how it might be used nor how
to build it).
$$\ket{v} = \frac{1}{\sqrt{2^n}} \sum_{i \in \{0,1\}^n} (-1)^{b_i} \ket{i}$$\

Real vectors: $\mathbb{R}^n$
----------------------------

Maybe you are used to see Greek letters inside a ket to represent
quantum states, and use latin letters to represent quantum states that
use binary expansion to hold classical data. The following is a very
common encoding in quantum machine learning. For a vector
$\vec{x} \in \mathbb{R}^{2^n}$, we can build:

$$\ket{x} = \frac{1}{{\left \lVert x \right \rVert}}\sum_{i=0}^{N}\vec{x}_i\ket{i} = |\vec{x}|^{-1}\vec{x}$$

Note that to span a space of dimension $N=2^n$, you just need $log_2(n)$
qubits: we encode each component of the classical vector in the
amplitudes of a state vector. Ideally, we know from Grover and Rudolph
(2002) how to create quantum states that corresponds to vector of data
(i.e. “efficiently integrable probability distribution”). We miss an
important ingredient. This encoding might not be enough if you have to
manipulate “many” vectors, as in some sense what you are creating is
vector with unitary norm. What if we want to build a superposition of
two vectors? Well, might expect to be able to create a state
$\frac{1}{\sqrt{N}} \sum_{i} \ket{x_i}$, but there’s a problem. Imagine
to do it with just two vectors: $x_1 = [-1, -1, -1]$ and
$x_2 = [1,1,1]$. Well, their (uniform) linear combination is the vector
$[0,0,0]$. What does this means? that to make a unitary vector out of
it, we need a exceptionally small normalizing factor. Usually this kind
of superpositions are obtained as a result of a measurement on an
ancilla qubit. The measurement has a probability that is proportional to
the norm of the vectors. Therefore, to be able to build this state we’re
gonna need an intolerable number of trial and error in building this
state. This problem can be amended by adjoining an ancilla register, as
we see now.

Matrices
========

Imagine to store your vectors in the rows of a matrix. Let
$X \in \mathbb{R}^{n \times d}$, a matrix of $n$ vectors of $d$
components. We will encode them using $log(d)+log(d)$ qubits as the
states:

$$\frac{1}{\sqrt{\sum_{i=0}^n {\left \lVert x(i) \right \rVert}^2 }} \sum_{i=0}^n {\left \lVert x(i) \right \rVert}\ket{i}\ket{x(i)}$$

Or, put it another way:

$$\frac{1}{\sqrt{\sum_{i=0}^n} {\left \lVert x(i) \right \rVert}^2} \sum_{i,j} X_{ij}\ket{i}\ket{j}$$

The problem is how to build it this state. We are going to need a very
specific oracle (which we call QRAM, even if there is ambiguity in
literature on that). A QRAM gives us access to two things: the norm of
the rows of a matrix and the rows itself. Calling the two oracles
combined, we can do the following mapping:

$$\sum_{i=0}^{n} \ket{i} \ket{0} \to  \sum_{i=0}^n {\left \lVert  x(i)  \right \rVert}\ket{i}\ket{x(i)}$$

Basically, we use the superposition in the first register to select the
rows of the matrix that we want, and after the query we have them in the
second register. A QRAM is a tree-like classical data structure that
offer quantum access in an oracular way to a data structure like this.
You can think of a QRAM as a circuit that encodes your matrix. Note that
using this nice encoding, the ratios between the distances between
vectors is the same as in the Hilbert space. Also note that once the
vector is created, the only way to recover $x$ from $\ket{x}$ is to do
quantum tomography (i.e. destroying the state with a measurement). The
cost (in term of time and space) of creating this data structure is a
little bit more than linear: $O(nd log (nd))$ but it pays by giving a
access time for a query that is $O(log(nd))$. (An example of QRAM can be
found in Kerenidis and Prakash (2017), and will obviously covered in
this blog in the next posts. Yes, I know. It might be difficult with the
physical implementation of QRAM, but I have faith the experimental
physicists. :)

Graphs
======

For specific problems we can even change the computational model (i.e.
no more gates on wires used to describe computation). For instance,
given a graph $G=(V,E)$ we can encode it as a state $\ket{G}$ such that:
$$K_G^V\ket{G} = \ket{G} \forall v \in V$$ where
$K_G^v = X_y\prod_{u \in N}(v)Z_u $, and $X_u$ and $Z_u$ are the Pauli
operators on $u$. The way of picture this encoding is this. Take as many
qubits in state $\ket{+}$ as nodes in the graph, and apply controlled
$Z$ rotation between qubits representing adjacent nodes. There are some
algorithms that use this state as input, for instance in Zhao,
Pérez-Delgado, and Fitzsimons (2016), where they even extended this
definition.\

Conclusions
===========

Te precision that we can use for specifying the amplitude of a quantum
state might be limited in practice by the precision of our quantum
computer in manipulating quantum states (i.e. development in techniques
in quantum metrology and sensing). Techniques that use a certain
precision in the amplitude of a state might suffer of initial technical
limitations of the hardware. As a parallel, think of what’s happening
with CPUs where we had 16, 32 and now 64 bits of precision.\

<div id="refs" class="references">

<div id="ref-Grover2002">

Grover, Lov, and Terry Rudolph. 2002. “Creating superpositions that
correspond to efficiently integrable probability distributions.”

</div>

<div id="ref-kerenidis2017quantum">

Kerenidis, Iordanis, and Anupam Prakash. 2017. “Quantum Gradient Descent
for Linear Systems and Least Squares.” *ArXiv Preprint
ArXiv:1704.04992*.

</div>

<div id="ref-schuld2015introduction">

Schuld, Maria, Ilya Sinayskiy, and Francesco Petruccione. 2015. “An
Introduction to Quantum Machine Learning.” *Contemporary Physics* 56
(2). Taylor & Francis: 172–85.

</div>

<div id="ref-zhao2016fast">

Zhao, Liming, Carlos A Pérez-Delgado, and Joseph F Fitzsimons. 2016.
“Fast Graph Operations in Quantum Computation.” *Physical Review A* 93
(3). APS: 032314.

</div>

</div>
