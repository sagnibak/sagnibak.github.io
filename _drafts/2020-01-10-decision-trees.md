---
title: Decision Trees
subtitle:
mathjax: true
tags: [decision trees, machine learning, cs 189]
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
temperature outside. If it is too hot or too cold, you will not go, and if it is
within, say, between 45°F and 90°F, you will go hiking, otherwise you will not.
This decision-making process can be modeled by a tree structure as shown below:

![insert image here]()

In place of this rather contrived example you can think of other decisions you
make everyday, like what to eat for breakfast tomorrow, which classes to take
next semester, what to do in the next hour, etc. As a fun exercise you can try
to draw a tree-diagram for them to visualize your decision-making process.
