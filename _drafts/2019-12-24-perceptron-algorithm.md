---
title: The Perceptron Algorithm
subtitle: The first step towards an SVM
mathjax: true
tags: [perceptron, machine learning, cs 189, svm]
---

## TL;DR

* The perceptron algorithm constructs a **linear** decision boundary by learning
a **weight vector** $w \in \mathbb{R}^d$.
  - The **decision function** is $w\top x$; the classification is
  $\\text{sign}(w\top x)$.
  - The **decision boundary** is $\\{ x \in \mathbb{R}^d \mid w\top x = 0 \\}$.

* The perceptron algorithm can be expressed as a linear program, and while one
could solve the linear program using an LP solver, we can also do (possibly)
online training using gradient descent.

* The perceptron algorithm is a very natural first step that one might
come up with when tasked with **binary classification**.


## The Setup: Binary Classification

We use the perceptron algorithm for binary classification. We are presented
with training samples $X_i$ which are vectors in $\mathbb{R}^d$ which have
labels $y_i$ that are either 1 or $-1$. We say that the input data has $d$
features, and based on those features, we want to decide whether the point
has the label 1 (it is in the class of choice) or $-1$ (it is not in the class
of choice).
Formally, we have $n$ data points $X_1, \ldots, X_n \in \mathbb{R}^d$, and
associated labels $y_1, \ldots, y_n \in \\{ -1, 1 \\}$.

Throughout this post we will be using the
[Iris Data Set](https://archive.ics.uci.edu/ml/datasets/iris). We will build
a classifier that decides if a flower is _Iris setosa_ or not.
In our example, a label of 1 corresponds to a flower being an _Iris setosa_,
and $-1$ corresponds to the flower being from some other species. In the
original data set, there are four features : sepal length, sepal width,
petal length, and petal width. However, for ease of visualization, we will use
only two of the four: sepal length and petal length.
Hence, we have $d = 2$. Each example $X_i$ has two elements. The first element
is the sepal length, and the second element is the petal length. Below is a plot
of the data that we will be working with:

![Iris data](/img/perceptron_iris0.png)

Something important that we can see is that the data is linearly separable,
which means that we can draw a line (more generally, a $(d-1)$-dimensional
hyperplane embedded in $\mathbb{R}^d$) that separates the two classes. Below
I have plotted three such lines:

![Iris data is linearly separable](/img/perceptron_iris_linear_sep.png)
