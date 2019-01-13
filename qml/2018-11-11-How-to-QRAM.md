---
id: 1
layout: chapter
chapter: 8 
title: How to QRAM
comments: true 
status: done
tags: qml, tools
description: "The fastest way to load classical data in a quantum computer is the QRAM: an efficient state preparation tecnique that requires an almost linear time algorithm as preprocessing."
author:
- Alessandro Scinawa Luongo
permalink: QRAM
---
In this post we’re going to see how to build circuit to efficiently load data in a quantum computer. 

This post is organized as such: first we well'see the theory behind the QRAM, stating the theorem that you can use in your algorithms to load classical data in a quantum computer. Then we will see *in practice* how to build some circuits which give you deeper insight on how a QRAM is built. Then, we will address some critique to the QRAM and draw some conclusions.

You can think of QRAM in two ways. First as a unitary gate you execute in your quantum computer, and also as a particular data format that you can use to store your data. Think of it as a data structure based on binary trees. If your data is a matrix (i.e. you have a list of vectors that represents data points to load into a quantum computer: $n$ rows and $d$ columns. As a circuit, QRAM is cna be run efficiently, meaning that the depth of the circuit is polylogarithmic in the input size. 


Loading classical data in the quantum computer means the following: for a given vector $x(i) \in \mathbb{R}^d$ and $i \in [n]$, we can use QRAM to build the following state:

$$ U_{QRAM}: \frac{1}{\sqrt{n}} \sum_{i\in [n]} \ket{i}\ket{0} \mapsto \frac{1}{\sqrt{\sum_{i \in [n]} \norm{x_i}^2}} \sum_{i=0}^{n} \ket{i}\ket{x(i)} $$

We we'll see how QRAM is needed to do much more than just that.

Let's take a second to admire this state and what it means. Mathematically, this state is just a very long vector of $nd$ (eventually padded to make $n$ and $d$ powers of two). You build it by piling all the rows of $X$ one after the other and then scaling everything to get unitary norm. Put it nother way, each of the $\ket{x_i}$ in the second register is a unitary vector ($\ket{x(i)} = \norm{x(i)}^{-1}x(i)$), and each of them is weighted by their norm ($\norm{x_i}$). You might wonder if you need the first register in the output of the QRAM, or if you can get rid of it. Well, it turns out it is actually very useful, as you will see in the next algorithms. 
In some sense, this state is not difficult to build per-se. The difficult thing is that we want to built it with a circuit whose depth is polylogarithmic in $n$ and $d$.

Let me indulge in another hopefully useful and not misleading image. To give you an intuition, think of our QRAM as a CD-ROM. The data is encoded in binary using little holes (or their absence) to represent a $0$ or a $1$. Imagine the first register as a photon directed towards the CD, whose wave function is a uniform probability distribution over the surface of the disk. When the photon reach the CD, (imagining the impact keeps the state coherent) it gets reflected, but now it has “read” the information on the CD. This example, which I like a lot, was said by Seth Lloyd in [this video](https://www.youtube.com/watch?v=Lbndu5EIWvI). 

Mathematically, the idea behind QRAM is rooted in an observation described in  {% cite Grover2002creating %}, which in just two pages of paper said gave an efficient algorithm for state preparation (i.e. the problem of finding a circuit that builds a given quantum state) for a specific set of state (that, for the record are square integrable probability didstribution) _having precomputed the amplitudes_. Luckily, a data vector in $\mathbb{R}^d$ (i.e. like most of the classical data that we use in ML) can be though is an efficiently square integrable distribution, and thus we can use the technique described there.

Before telling you how the circuit looks like, let me shed some light about an ambiguity in some names that we have currently in quantum information. When we (I ? ) speak about QRAM we are referring to an oracle that performs a *amplitude* encoding of the values in a quantum state. Generically, oracles are used to encode *numbers* in quantum state in what is commonly known as *digital encoding* (see [here](loading) for more clarification on possible encodings for data). Usually, an oracle (like the Grover oracle, or an oracle for amplitude amplification) perform this mapping. 

$$ U:\ket{i}\ket{0}\mapsto\ket{i}\ket{x_i}$$

(note the change in notation: here $x_i \in [\mathbb{Z}]$ so it's something like $\ket{010110}$)

Let's go technical here. Imagine we have received a set of classical data in form of a matrix of $n$ rows (samples) and $d$ columns (features). The QRAM for a matrix $X$ is a collection of binary trees $B_i^M$ of depth $\log d$, where each tree has stored the information needed to create a state corresponding to each row of the matrix. Your quantum circuit will perform "parallel" controlled rotation on $\log d$ qubits efficiently. Now I start with the theorem showing you that the circuit I'm building acually works: meaning that it creates a quantum state proportional to a given matrix.  

##### Theorem: The QRAM

_Let_ $X \in \mathbb{R}^{N \times d}$. _There exists a data structure to store the rows of $X$ such that:_
- _The size of the data structure is_ $O(nd\\: log(nd))$
- _The time to store a row $x(i)$ is $O(d\\: log^2(nd))$, and the time to store the whole matrix $X$ is thus $O(nd\\: log^2(nd))$_
- _A quantum algorithm that has quantum access to the data structure can perform the mapping 
$U\_X: \ket{i}\ket{0} \to \ket{i}\ket{x(i)} $ and the mapping $V\_X : \ket{j}\ket{0} \to \ket{j}\ket{\widehat{X}}$ for $j \in [d]$ where $\widehat{X} \in \mathbb{R}^n$ has entries $\widehat{X}_i = \norm{x(i)}$ in time $polylog(nd)$_

The proof consist of builing the following classical data structure. Let's start (but note that I'll be mostly re-writing what's written in {% cite kerenidis2016recommendation %}). For each row $x(i) \in X$ we build a  binary tree $B_i$ of depth $\lceil \log d\rceil$ with $d$ leaves. Each internal node $v$ of the tree stores the sum of the entries of all the leaves of the subtree rooted at $v$, that is, the sum of the 2 childrens. The leaf of each tree stores the value $x_{ij}^2$, along with the sign of $x_{ij}$. Organized as such, you can easily verify that the root of the tree stores the sqared norm of $x(i)$. 

Each layer of the tree is stored using an ordered list. **clarify better: what is an ordered list and what the time to retrive the ADDRESS of the node is $\log nd$. Note that probably, the time is $\log nd$ becuase you need to manipulate $\log nd$ bits even to do a query to a dictionary, which is O(1).**

To finish this classical pre-processing on our data we need to measure the time and the space needed to build and store this data structure. 
- the time needed to buidl a binary tree for a vector of length $d$ is $O(d \log^2 nd)$ because for each of the $d$ component of the vector, there are $\lceil log d \rceil$ updates to the data structure (given by the creation of a new leave and the updates on the internal nodes of the
tree), and each update requires $O(\log nd)$ operations to retrieve the address of the node to update. 

For what concern the space, given that we have to store $n$ vectors, the time needed to store the whole matrix is $O(nd\\: log^2(nd))$. The space used to store the matrix $X$ is therefore $O(nd\\: log^2 nd)$
since there are $N$ vectors and each vector will create $O(d\:log(d))$ nodes, and each node use $O(log Nd)$ bits.

**also question: why we just consider n and d and never the bits of precision we use to store the data?**



Here, summing  That looks like “integrating” the probability distribution corresponding to a data vector $x(i)$, which Lov Grover wrote about {% cite Grover2002creating %}, albeit referring to easily sqare integrable probability distribution. 


Note that for a tree $B_i$, at depth $t$ the data stored in a node
$k \in \{0,1\}^t$ is:

$$B_{i,k} := \sum_{j \in [n], j_{1:t}=k} x^2_{ij}$$

In this formula with $j_{1:t}=k$ we denote the first $t$ bits in the binary expansion of $j \in \left[ d \right]$. This value represent the probability of observing outcome $k$ by reading only the first $t$  qubits of $\ket{x(i)}$ in the standard basis. This observation will be used to perform a series of controlled rotations on the data registe (i.e. the set of qubits storing the vector $x(i)$. Let's see how. 


Now we will show how $U_X$ can be used to build a state $\ket{x(i)}$ using the tree $B_i$ in polylog($Nd$). In the following we assume to
have quantum access to all the data structure allowing us to query $U_X$ in polylog($Nd$) time.s Given an initially empty register $\ket{0}$ of
$\lceil log d \rceil$ qubits, we perform a series of rotation such that the amplitudes of the state of the register match the probability stored
in the QRAM trees. Specifically, we apply the following map:

$$\ket{i}\ket{k}\ket{0} \to \ket{i}\ket{k}\frac{1}{\sqrt{B_{i,j}}} \Big(\sqrt{B_{i,k0}}\ket{0} + \sqrt{B_{i,k1}}\ket{1}  \Big)$$

For the last qubit, the controlled rotations take into consideration
also the eventual sign:

$$\ket{i}\ket{k}\ket{0} \to \ket{i}\ket{k}\frac{1}{\sqrt{B_{i,j}}} \Big(sgn(a_{i,k0})\sqrt{B_{i,k0}}\ket{0} + sgn(a_{i,k0})\sqrt{B_{i,k1}}\ket{1}  \Big)$$

The number of controlled rotation to apply is $\lceil log d \rceil$. The operation for the qubit $t+1$, is controlled on the index register $\ket{i}$ and on the previous $t$ bits of the register being on state $k$. For each controlled operation there are 2 query to the binary tree: once for each children.

Finally, we show how to build $V\_X$. 
This procedure is analogue to the creation of a binary trees $B\_i$, except that instead of saving the square of the amplitude we just need to save the square of the norm of the vectors. The $i$-th component of the vector $\widehat{X}$ is 
$||x(i)||$, and it’s the value stored at the root node of the $i$-th binary tree. 
Thus, when creating $B\_i$ we also update the tree for the oracle $V_X$. Querying the oracle $V_X$ can be done in polylog($Nd$)  time as well.



Observe also that the root of each tree stores the (squared) norm of the vector. Therefore, we can build *another* tree on top of the other trees that we have built, in case we want to load into the quantum computer also the norms of the vectors:

$$  \sum_{i=0}^n\ket{i} \to \sum_{i=0}^n ||x_i||\ket{i} $$ 

####  Some circuits. 

Let's see some circuits that we can use to create a QRAM. Let's start from the simplest. Do you remember the Grover oracle? In the Grover algorithm you have an oracle that is capable of distinguishing 

you can think of this as a list $[n]$ elements representing all the $f(x)$ for which just one is $1$ and the others are $0$. How would you build the oracle $U_f \ket{x}\ket{0} \to \ket{x}{f(x)}$? This oracle is just a controlled operation! Controlled on the value of the index register (the leftmost) being $x$ we perform a X gate on the data register. 

Note that the depth of this oracle is $O(1)$ since we have just a single gate to execute. (it's actually a little bit more than a single gate since we have to decompose the multi-controlled-not in the gate set supported by the QPU (Quantum processing unit) but this is another story..

### Multiplexer oracle for numbers
We can push this idea further. What if, instead of writing just a $1$ in the data register, we perform other controlled-NOT gate?! This would allow us to load bitstrings into the computer. Here is an example:

Here is an image of the circuit.

**image**

Note that, since in qiskit you dont have the ability to control operation on qubits being $0$, we perform a $X$ gate before using a "normal" control (and we undo it after). 
In this case the depth of the circuit is $O(n)$ for $n$ different m-bits values. This is the oracle used in the [perceptron](perceptron) paper, and actually many many other. 

### Multiplexer QRAM 

We can apply the same idea of the multiplexer for loading nubmers and use it to perform the controlled rotations we need to encode information in the amplitudes. 
In this way, the depth of the circuit will still be linear in the number of rows (i.e. we don't get any exponential speedup) but at least we are able to built the state we need.


Here is an image of the circuit.

**image**

### Bucket brigade oracle and bucket brigade enabled QRAM

This is where fun starts. We want to have a log-depth cirucuit to 
This is thorougly described in {% cite giovannetti2008architectures %} but going into depth would require a post in its own. 

We want to make a QRAM for loading vectors out of this, and this is the real deal. Controlled on the index register $\ket{i}$, we want to use a bucket-brigade architecture to load each of the $\log d$ layer of the $B_i$ trees in an ancillary register. Then, we can use other controlled rotation (controlled on the ancilla register we used to store the partial amplitudes loaded so far) to perform the rotation we have done manually in the previous example. 

In this way we can have achieve the goal we want: a log depth circuit to load matrix. 




#### Conclusions, criticisms, ideas...
In the original work, instead of building the QRAM tree after tree assuming the are given all the vectors at once, the authors assumed to receive the compoent $x_{ij}$ of the matrix $X$ one after the other. We build the QRAM for the dataset sequencially, one vector at the time, so the time needed for the QRAM is at least $O(N)$ (albeit this can be parallelized). 

I hope now is clear why I consider the QRAM as a particular data format. From the trees $B_i$ it is straightforward to recover the original matrix $X$!

There are some criticism that QRAM will be difficult to build, due to the complexity of the circuit (many CNOT and controlled orataions are involved). Well, I argue that it will be as difficult as executing any other quantum cirucit. Moreover, we know from {% cite arunachalam2015robustness %} that circuit doing a polynomial number of queries to a QRAM might not need error correcting codes (they will just fail with a certain probability, and we can take majority of voting and bound exponentially the failure probability of the algorithm). 

Note that this model of "receiving and preprocessing" data suits perfectly the machine learning problem. For instance, things like normalizing, removing the mean, and scaling data to unitary variance (all things which are often done when the training set is received, can also be done in $O(n)$ time during the preprocessing needed to build the QRAM.

*Idea 1*. Note that you can imagine to store in the QRAM also a dataset after a pass of "Gaussian noise",  such that we anonymize the value of each of the vectors of the matrix but we might preserve the overall value of the thing that we can learn. Unfortunately, I do not know much about privacy preserving machine learning, but this is definitely worth studying more. We, as quantum information community cannot pretend to ignore all the progress made in privacy preserving machine learning since sooner or later, if we expect to apply QML in the real world we also need to be compliant to the laws we have done so far and we (I ?) expect to be developed in the following years. 

*Idea 2*. Also note that if you give me the circuit for a QRAM, I can recover very simply the original matrix X. Indeed, as we said at the beginning you can think of the QRAM as a particular data format. Essentially, the value of the controlled rotation is the value of the angle in the binary tree, and from these values we can recover each vector in the matrix $X$. What if we add some scrambling in such a way that the only way to recover the dataset is doing tomography on a massively huge quantum state!? \o/ That would be augment furhter the privacy of storing our data..


I just arrived with Giorgio in Grenoble to celebrate the New Year's Eve. The 3rd of January I'll be visiting the quantum group at Grenoble's university and I'll be talking about [q-means](https://arxiv.org/abs/1812.03584) and I am super excited :) \o/ 

Daje, bella li. 

Totally unrelated:
<iframe width="420" height="315" src="https://www.youtube.com/watch?v=bGV1xYJFAEI" frameborder="0" allowfullscreen></iframe>


### References

{% bibliography --cited %}
