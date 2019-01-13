---
id: 1
layout: chapter
chapter: 7 
title: How to evaluate a classifier
comments: true 
status: done
tags: qml, intro
description: Once you have created your quantum machine learning classifier, you need to understand how good it is in solving your problem. Here is a short recap of the most used metrics in data science.
author:
- 'Alessandro “Scinawa” Luongo'
bibliography:
- 'sample.bib'
- 'Mendeley.bib'

---


Practitioners in quantum machine learning should not only build their
skills in quantum algorithms, and having some basic notions of
statistics and data science won’t hurt. In the following the see some
ways to evaluate a classifier. What does it means in practice? Imagine
you have a medical test that is able to tell if a patient is sick or
not. You might want to consider the behavior of your classier with
respect to the following parameters: the cost of identifying a sick
patient as healthy is high, and the cost of identifying a healthy
patient as sick. For example, if the patient is a zombie and it
contaminates all the rest of the humanity you want to minimize the
occurrences of the first case, while if the cure for “zombiness” is
lethal for a human patient, you want to minimize the occurrences of the
second case. With P and N we count the number of patients tested
Positively or Negatively. This is formalized in the following
definitions, which consists in statistics to be calculated on the test
set of a data analysis.

-   **TP True positives (statistical power)** : are those labeled as
    sick that are actually sick.

-   **FP False positives (type I error)**: are those labeled as sick but
    that actually are healthy

-   **FN False negatives (type II error)** : are those labeled as
    healthy but that are actually sick.

-   **TN True negative**: are those labeled as healthy that are healthy.

Given this simple intuition, we can take a binary classifier and imagine
to do an experiment over a data set. Then we can measure:

-   **True Positive Rate (TPR) = Recall = Sensitivity**: is the ratio of
    correctly identified elements among all the elements identified as
    sick. It answer the question: “how are we good at detecting sick
    people?”.
    $$\frac{  TP }{ TP +  FN}  + \frac{TP }{P} \simeq  P(test=1|sick=1)$$
    This is an estimator of the probability of a positive test given a
    sick individual.

-   **True Negative Rate (TNR) = Specificity** is a measure that tells
    you how many are labeled as healthy but that are actually sick.
    $$\frac{ TN }{ TN +  FP}  = p(test = 0 |  sick =0)$$ How many
    healthy patients will test negatively to the test? How are we good
    at avoiding false alarms?

-   **False Positive Rate = Fallout**
    $$FPR = \frac{  FP }{ FP +  TN }  = 1 - TNR$$

-   **False Negative Rate = Miss Rate**
    $$FNR = \frac{  FN }{ FN +  TP } = 1 - TPR$$

-   **Precision, Positive Predictive Value (PPV)**:
    $$\frac{ TP }{ TP + FP} \simeq p(sick=1 | positive=1)$$ How many
    positive to the test are actually sick?

-   **$F_1$ score** is a more compressed index of performance which is a
    possible measure of performance of a binary classifier. Is simply
    the harmonic mean of Precision and Sensitivity:
    $$F_1 = 2\frac{Precision \times Sensitivity }{Precision + Sensitivity }$$

-   **Receiver Operating Characteristic (ROC)** Evaluate the TRP and FPR
    at all the scores returned by a classifier by changing a parameter.
    It is a plot of the true positive rate against the false positive
    rate for the different possible value (cutpoints) of a test or
    experiment.

-   The **confusion matrix** generalize these 4 combination of (TP TN FP
    FN) to multiple classes: is a $l \times l$ where at row $i$ and
    column $j$ you have the number of elements from the class$i$ that
    have been classified as elements of class $j$.

Bref. This post because I always forgot about these terms and I wasn’t
able to find them described in a concise way with the same formalism
without googling more time than that I spent writing this post. Other
links:
[here](https://uberpython.wordpress.com/2012/01/01/precision-recall-sensitivity-and-specificity/)
