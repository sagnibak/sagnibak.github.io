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

In fact, the number of nodes to model some functions like the majority function
(determining if there are more 1's than 0's in the input vector) and XOR
(determining if there is an odd number of 1's in the input vector) is actually
exponential in the dimension $d$ of the training data.

### NP-Hard does not mean perfect tho

One thing to note is that while a decision tree with unlimited depth can learn
any discrete-valued function, it may not be able to model training data perfectly,
if the data are themselves inconsistent, since a function cannot map the same
input to multiple outputs.

Think about it: if your data say that you go hiking when it is raining, you have
time, and the temperature is greater than 90°F, and you also *do not* go hiking
when it is raining, you have time, and the temperature is greater than 90°F, what would you
conclude by looking at the data alone, after forgoing all your knowledge about
temperature and precipitation? You cannot both go hiking and also not go hiking
at the same time. This is analogous to the fact that then there is no
**function** that can correctly classify all your input data. Thus you cannot
always perfectly classify your training data even if you are using a very complex
hypothesis function (decision tree in this case).

Formally, if we have $i \neq j$ such that $x_i = x_j$
but $y_i \neq y_j$, i.e., the training set has different labels for the *same*
example, then we will not be able
to classify the training data perfectly since no function $f$ can satisfy $f(x_i)
\neq f(x_j) : x_i = x_j$. However that still does not mean our model
will not overfit. It certainly can.

This actually happens quite frequently in real-life data, since we often do not
want to model mathematical functions but probability distributions, which often
overlap, leading to the same or similar points having completely different
classifications. Fortunately, dealing with this is rather easy.

### Coping with NP-Completeness

We do know two (generally) great ways to deal with NP-completeness: trying the
obvious thing (being greedy) and trying everything (something like branch-and-bound).
To learn decision trees, the most common approach is to use a **greedy heuristic**.

In order to be able to construct decision trees in reasonable amounts of time,
the most common approach is to recursively grow the tree, by choosing the best
split at each internal node. It turns out that this is linear in the number of
training samples, thus even for relatively large data sets, your tree is ready
before your tea is (I am quite proud of this).

## Greedily Growing Your Tree

In order to grow a decision tree greedily we first need to define a concrete
data structure. In this article my trees are binary trees. Each internal node
stores the feature to test (the splitting feature) and the splitting value for
that feature, along with two subtrees. Each leaf node stores the classification
(an integer representing the class for classification trees, and a real number
for regression trees).

```scala
DecisionTree := LeafNode | InternalNode
LeafNode := classification: Int | Real
InternalNode := (split_feature: Int,
                 split_value: Real,
                 left_subtree: DecisionTree,
                 right_subtree: DecisionTree)
```

From the definitions above we can see that if we want to construct a decision
tree recursively, we need to find a splitting feature and a splitting value
before we can make two recursive calls to the tree-growing function. We also
need to determine how to simplify the problem for the recursive calls, and decide
when to stop.

Note that this is not the only way to define or build
decision trees. Decision trees are a heavily researched machine learning algorithm
and if you think you have a new idea about how to make decision trees, there
are at least five papers on it if there aren't five textbooks describing it.

If you want to follow along with a well-documented implementation of a decision
tree that I made, please take a look at this implementation in my repository
[visuaml](https://github.com/sagnibak/visuaml/blob/master/decision_trees/dtree.py).
Note that in this implementation, I store the indices of the examples in the leaf
node instead of storing a class, so that I can potentially calculate posterior
probabilities of classes instead of just predicting the most frequent class.

In the following discussion, let $S$ be the set of indices to
consider at an internal node, i.e., if $S = \\{ 3, 5, 6 \\}$, then the internal
node only considers examples $x_3, x_5, x_6$ which have labels $y_3, y_5, y_6$.

### When to stop

If all the examples being considered have the **same label**, then there is no need
to split, and we can terminate tree growth by returning a leaf node.

Formally, if $y_i = y_j \\ \forall i, j \in S$, then we return a leaf node containing
the label $y_i$ where $i$ is any entry of $S$.

Note that if we have **inconsistent** training data then it is possible that all the
labels are not identical but we cannot make any split since all the points being
considered are the same, i.e., $x_i = x_j \\ \forall i, j \in S$ but $\exists i,
j \in S : y_i \neq y_j$. We need to detect this case and also terminate training
by returning a leaf node. In this case one sensible thing to do is to pick the
label that has occurs most often in the data being considered, i.e., $y_i : i \in S$.

Here is a link to the
[relevant part of my code](https://github.com/sagnibak/visuaml/blob/master/decision_trees/dtree.py#L221).

This can be justified loosely with the following intuition: if in the training
data we see that on rainy days when you have time and the temperature is greater
than 90°F you go hiking on 2 occasions but you don't go on 7 occasions, then it
is probably more likely that you will not go hiking in such a scenario. Hence,
choosing the label with plurality should get you better accuracy in the long run,
especially if your sample size is large enough (i.e., if your leaves contain
enough elements), since that reduces the variance of the estimate of the probability
of you going hiking given the scenario.

We will also add more stopping criteria when we consider reducing overfitting.

### How to split

If we decide that we have not reached a stopping criterion, then we need to
construct an internal node. Hence, we first need to choose a feature to split on
and pick the best value to split at. The most optimal way to do this (for fitting
the training data, at least) would be to try growing a full tree using every split
at every level all the way to the leaves, but that is NP-complete, so we will
use heuristics instead. For training classification trees, people tend to use
[Gini impurity](https://en.wikipedia.org/wiki/Decision_tree_learning#Gini_impurity) or
[entropy](https://en.wikipedia.org/wiki/Decision_tree_learning#Information_gain),
while for training regression trees (not covered really covered here) people tend to use
[mean squared error](https://en.wikipedia.org/wiki/Decision_tree_learning#Variance_reduction).
