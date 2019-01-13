---
id: 1
layout: chapter
chapter: 7 
title: Selected articles in Quantum Machine Learning"
comments: true 
status: todo
tags: qml, other
description: List of relevant papers in quantum machine learning"
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---

This is a collection of paper I have found useful in the last years. It is far from complete and you are welcome to suggest new entries here that you think I have missed.
I don't claim for completeness though. 

#### 2018
- [Quantum Statistical Inference](https://arxiv.org/pdf/1812.04877.pdf) `#phdthesis, #algo`  
The PhD thesis of Zhao! With focus on Gaussian Processes, Quantum Bayesian Deep Learning (and other resources about causality and correlations..).

- [Troubling Trends in Machine Learning Scholarship](https://arxiv.org/pdf/1807.03341.pdf) `#opinion-paper`  
Is a self-autocritic of the ML community on the way they are doing science now. I think this might be relevant as well for the QML practicioner.

- [Quantum machine learning for data scientits](https://arxiv.org/pdf/1804.10068.pdf) `#review`  `#tutorial`
This is a very nice review of some of the most known qml algorithms. I wish I had this when I started studying QML.  

- [Image classification of MNIST dataset using quantum slow feature analysis]() `#algo`  
This is my first work in quantum machine learning. Here we show 2 new algorithms 
The idea is to give evidence that QRAM based algorithms can obtain a speedup w.r.t classical algorithm in QML *on real data*.

- [Quantum algorithm implementations for beginners](https://arxiv.org/pdf/1804.03719.pdf) `#review` `#tutorial`  

#### 2017

- [Implementing a distance based classifier with a quantum interference circuit]()  `#algo`  


- [Quantum machine learning for quantum anomaly detection]() `#algo`  
Here the authors used previous technique to perform anomaly detection. Basically they project the data on the 1-dimensional subspace of the covariance matrix of the data. In this way anomalies are supposed to lie furhter away from the rest of the dataset. 

- [ Quantum machine learning: a classical perspective](https://arxiv.org/pdf/1707.08561.pdf): `#review` `#quantum learning theory` 


#### 2016
- [Quantum Discriminant Analysis for Dimensionality Reduction and Classification]()  `#algo`  
Here the authors wrote two different algorithm, one for dimensionality reduction and the second for classification, with the same capabilities 


- [Quantum Recommendation Systems]() `#algo`  
It is where you can learn about QRAM and quantum singular value estimation. 

#### 2015

- [Advances in quantum machine learning]( https://arxiv.org/pdf/1512.02900.pdf ) `#implementations`,  `#review`   
It cover things up to 2015, so here you can find descriptions of Neural Networks, Bayesian Networks, HHL, PCA, Quantum Nearest Centroid, Quantum k-Nearest Neighbour, and others.

- [Quantum algorithms for topological and geometric analysis of data]() `#algo`  

##### 2014

- [Quantum Algorithms for Nearest-Neighbor Methods for Supervised and Unsupervised Learning]() `#tools`, `#algorithms`  
This paper offer two approaches for calculating distances between vectors. 
The idea for k-NN is to calculate distances between the test point and the training set in superposition and then use amplitude amplification tecniques to find the minimum, thus getting a quadratic speedup.
 
- [Quantum support vector machine for big data classification Patrick]() `#algo`  
This was one of the first example on how to use HHL-like algorithms in order to get something useful out of them.  


- [Quantum self-testing]()  `#algo`  
The authors discovered how partial application of the swap test are sufficient to transform a quantum state $\sigma$ into $U\sigma U^\dagger$ where $U=e^{-i\rho}$ given the ability to create multiples copies of $\rho$. 
This work uses a particular access model of the data (sample complexity), which can be obtained from a QRAM 

##### 2013
- [Quantum algorithms for supervised and unsupervised machine learning](https://arxiv.org/pdf/1307.0411.pdf) `#algo`  
This explain how to use swap test in order to calculate distances. Then it shows how this swap-test-for-distances can be used to do NearestCentroid and k-Means with adiabatic quantum computation 



##### 2009
- [Quantum algorithms for linear systems of equations]() `#algo`  
This is the paper that started everything. :) Tecniques for sparse Hamiltonian simulation and phase estimation were applied in order to estimate the singular values of a matrix. Then a controleld rotation on ancilla qubit + postselection creates a state proportional to the solution of a system of equation. You can learn more about it [here](HHL). 


### Code

- [Grove](http://grove-docs.readthedocs.io/en/latest/)
- [Qiskit-acqua]() 
- [Projective Simulation](https://projectivesimulation.org)

