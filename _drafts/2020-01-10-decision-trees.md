---
title: Decision Trees
subtitle:
mathjax: true
tags: [decision trees, machine learning]
---

The decision tree is a very natural machine learning algorithm. The intuition
behind it is akin to the intuition we use when we make a decision ourselves, as
opposed to, say the perceptron algorithm which starts by thinking of points in
a space separated by a hyperplane, or Gaussian discriminant analysis which relies
heavily on the interaction of probability distributions. In this blog I highlight
the natural-ness of every aspect of decision trees. We will start by developing
the inference algorithm, and then we will discuss various heuristics we use to
build accurate trees that don't overfit.

## Making Decisions

Let's say you want to decide whether to go on a hike today. You might check the
weather first. If it is raining, you will not go. If it is not raining, you might
check your schedule to see if you actually have time to go hiking, and if you
don't have time then you won't go. If you do have time, you might check the
temperature outside. If it is too hot or too cold, you will not go, and if it is,
say, between 45°F and 90°F, you will go hiking.
This decision-making process can be modeled by a tree structure as shown below:

![stupid intro tree](/img/decision_tree/intro_tree.png)

In place of this rather contrived example you can think of other decisions you
make everyday, like what to eat for breakfast tomorrow, which classes to take
next semester, what to do in the next hour, etc. As a fun exercise you can try
to draw a tree diagram for them to visualize your decision-making process.

Decision trees used in machine learning are based on this intuition. When given
an observation, we test its features one after another until we reach a leaf node,
at which point we output the decision corresponding to the leaf node. We simply
traverse the tree until we reach a leaf node. At each internal node we decide
which child node to visit by performing some kind of test on the input vector.
Usually the test comprises checking whether a feature has a value in some some
range, although it can certainly be more complex (for example, we can have a
support vector machine or even a neural network in an internal node, though
I've never seen anyone do that). Making the decision is the easy part, though.

## Learning to Make Decisions (from Data)

The hard part is learning the decision-making process, which takes the form of
learning a decision tree from data. We are given a list of $n$ training examples
$x_1, \ldots, x_n$, where each of the points $x_i$ contains $d$ features. In the
stupid example above, the $d=3$ features would be
- whether it is raining,
- whether I have time, and
- the temperature.

In the [Iris data set](https://archive.ics.uci.edu/ml/datasets/Iris) which we
will use shortly, there are four features in each example, corresponding to
sepal length, sepal width, petal length, and petal width. And each example $x_i$
also has a corresponding label $y_i$. In the hiking example, each $y_i$ is a boolean
expressing whether or not we go hiking, and in the Iris data set each label $y_i$
is one of the three possible kinds of Iris flowers. If we are doing regression
instead, then the labels $y_i$ can be any real number.

Now the question is, given $x_1, \ldots, x_n \in \mathbb{R}^d$ and $y_1, \ldots,
y_n$, how one should go about constructing a decision tree that can classify unseen
data drawn from the same distribution as the training data as accurately as
possible.

## With Great Power Comes NP-Completeness

Decision trees are incredibly expressive. In fact with enough (exponentially many)
leaves, a decision tree can perfectly model any binary function (and hence, any
discrete-valued function in general). There are a few problems with this.

1. If our decision tree can model training data perfectly, this will almost certainly
lead to **overfitting**.
2. Even if we don't care about overfitting, searching through an exponentially
large space will take exponential time, hence we will not be able to train with
particularly large training sets. Hence, finding the optimal decision tree is
NP-complete.

One thing to note is that while a decision tree with unlimited depth can learn
any discrete-valued function, it may not be able to model training data perfectly,
if the data are themselves inconsistent. If we have $i \neq j$ such that $x_i = x_j$
but $y_i \neq y_j$, i.e., the training set has different labels for the *same*
example (due to overlapping distributions, for example), then we will not be able
to classify the training data perfectly. However that still does not mean our model
will not overfit. It certainly can. This was just a conceptual digression.

We do know two (generally) great ways to deal with NP-completeness: trying the
obvious thing (being greedy) and trying everything (something like branch-and-bound).
To learn decision trees, the most common approach is to use a **greedy heuristic**.
