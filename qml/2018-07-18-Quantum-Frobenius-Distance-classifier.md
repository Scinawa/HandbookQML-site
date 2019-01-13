---
id: 1
layout: chapter
chapter: 7 
title: "Quantum Frobenius Distance Classifier (QFDC)"
comments: true 
status: done
tags: qml, algos
description: A simple classifier inspired by Nearest Centroid but based on square distances. 
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'
permalink: qfdc
---


Yesterday night there was the TQC dinner in Sydney, and I had the change to speak with Maria, a very prolific author in QML. While speaking about her work on [distance based classification](https://arxiv.org/abs/1703.10793), which is [further analyzed here](https://arxiv.org/abs/1803.00853), she said that one of the purposes of the paper was to show that an Hadamard gate is enough to perform classification, and you don't need very complex circuit to exploit quantum mechanics in machine learning. These was exaclty our motivation behind our QFDC classifier as well, so here we are with a little descrption of QFDC! This text is taken straight outta [my paper](https://arxiv.org/abs/1805.08837).

As usual, I assume data is stored in a QRAM. We are in the settings of supervised learning, so we have some labeled samples $x(i)$ in $\mathbb{R}^d$ for K different labels. Let $X_k$ be defined as the matrix whose rows are those vectors, and therefore have $K$ of those matrices. 
$|T_k|$ is the number of elements in the cluster (so the number of rows in each matrix). 

For a test point $x(0)$, define the matrix $ X(0) \in \mathbb{R}^{|T_k| x d} $ 
which just repeats the row $x(0)$ for $|T_k|$ times. 
For $X(0)$, the number of rows is context dependent, but it hopefully be clear. Then, we define 

$$F_k( x(0)) = \frac{ ||X_k - X(0)||_F^2}{2 ( ||X_k||_F^2+ ||X(0)||_F^2) },$$ 

which corresponds to the average normalized squared distance between $x(0)$ and the cluster $k$. 
Let $h : \mathcal{X} \to [K]$ our classification function. We assign to $x(0)$ a label according to the following rule:

$$ h(x(0)) :=  argmin_{k \in [K]} F_k( x(0))$$

We will estimate $F_k( x(0))$ efficiently using the algorithm below. From our QRAM construction we know we can create a superposition of all vectors in the cluster as quantum states, have access to their norms and to the total number of points and norm of the clusters. We define a normalization factor as:

$$ N_k= ||X_k||_F^2 + ||X(0)||_F^2 = ||X_k||_F^2 +|T_k| ||x(0)||^2. $$



##### Require
- QRAM access to the matrix $X_k$ of cluster $k$ and to a test vector $x(0)$. Error parameter $\eta > 0$. 

##### Ensure
- An estimate $\overline{F_k (x(0))}$
 such that $| F_k(x(0)) - \overline{F_k( x(0))} | < \eta $.

##### Algorithm
- Start with three empty quantum register. The first is an ancilla qubit, the second is for the index, and the third one is for the data. 
$$ \ket{0}\ket{0}\ket{0} $$
- $s:=0$
- For $r=O(1/\eta^2)$
     - Create the state  
        $$ \frac{1}{\sqrt{N_k}}  \Big( \sqrt{|T_k|}||x(0)||\ket{0} +||X_k||_F \ket{1}\Big) \ket{0}\ket{0}$$  
     - Apply to the first two register the unitary that maps: 
       $$\ket{0}\ket{0} \mapsto \ket{0} \frac{1}{\sqrt{|T_k|}} \sum_{i \in T_k} \ket{i}\; \mbox{ and } \;  \ket{1}\ket{0} \mapsto \ket{1} \frac{1}{||X_k||_F} \sum_{i \in T_k} ||x(i)|| \ket{i}$$
     This will get you to:
     $$   \frac{1}{\sqrt{N_k}} \Big( \ket{0}  \sum_{i \in T_k}  ||x(0)|| \ket{i} + \ket{1} \sum_{i \in T_k}  ||x(i)|| \ket{i} \Big) \ket{0} $$
     - Now apply the unitary that maps
     $$ \ket{0} \ket{i} \ket{0} \mapsto   \ket{0} \ket{i} \ket{x(0)} \; \mbox{ and } \;   \ket{1} \ket{i} \ket{0} \mapsto   \ket{1} \ket{i} \ket{x(i)}$$
     
     to get the state
       $$   \frac{1}{\sqrt{N_k}} \Big( \ket{0}  \sum_{i \in T_k}  ||x(0)|| \ket{i} \ket{x(0)}+ \ket{1} \sum_{i \in T_k}  ||x(i)|| \ket{i}\ket{x(i)} \Big)  $$
     
    - Apply a Hadamard to the first register to get
     $$   \frac{1}{\sqrt{2N_k}}\ket{0} \sum_{i \in T_k} \Big(  ||x(0)|| \ket{i} \ket{x(0)} +  ||x(i)|| \ket{i}\ket{x(i)} \Big)  +
     \frac{1}{\sqrt{2N_k}}\ket{1} \sum_{i \in T_k} \Big(  ||x(0)|| \ket{i} \ket{x(0)} -  ||x(i)|| \ket{i}\ket{x(i)} \Big) $$
    - Measure the first register. If the outcome is $\ket{1}$ then $s:=s+1$
    
- Output $\frac{s}{r}$.

Eventually, if you want to get a quadratic speedup w.r.t. $\eta$, perform amplitude estimation (with $O(1/\eta)$ iterations) on register $\ket{1}$ with the unitary implementing steps 1 to 4 to get an estimate $D$ within error $\eta$. This would make the circuit more complex, therefore less suitable for NISQ devices, but if you have enough qubits/fault tolerance, you can add it. 


For the analysis, just note that the probability of measuring $\ket{1}$ is:

$$\frac{1}{2N_k} \left ( |T_k|||x(0)||^2 + \sum_{i \in T_k} ||x(i)||^2 - 2\sum_{i \in T_k} \braket{x(0), x(i)} \right) = F_k(x(0)).$$

By Hoeffding bounds, to estimate $F_k(x(0))$ with error $\eta$ we would need $O(\frac{1}{\eta^2})$ samples.
For the running time, we assume all unitaries are efficient (i.e. we are capable of doing them in polylogarithmic time) either because the quantum states can be prepared directly by some quantum procedure or given that the classical vectors are stored in the QRAM, hence the algorithm runs in time $\tilde{O}(\frac{1}{\eta^2})$. We can of course use amplitude estimation and save a factor of $\eta$. Depending on the application one may prefer to keep the quantum part of the classifier as simple as possible or optimize the running time by performing amplitude estimation.

Given this estimator we can now define the QFD classifier.

##### Require
- QRAM access to $K$ matrices $X_k$ of elements of different classes.
- A test vector $x(0)$. 
- Error parameter $\eta > 0$. 

##### Ensure
- A label for $x(0)$.   

##### Algorithm
- For $k \in [K]$ 
    - Use the QFD estimator to find $F_k(x(0))$ on $X_k$ and $x(0)$ with precision $\eta$.
- Output $h(x(0))=argmin_{k \in [K]} F_k( x(0))$.


The running time of the classifier can be made $\tilde{O}(\frac{K}{\eta})$ when using amplitude amplification. That was it. QFDC basically exploit the subroutine for finding the average sqared distance between a point and a cluster and assign the test point to the "closest" cluster. 

Drowbacks of this approach is that is very sentible to outliers. This is because we take the square of the distance of the points belonging to a cluster. This apparently can be mitigated by a proper dimensionality reduction algorithm, like [QSFA](QSFA).

