---
title: (One (Deep) Reason) Why Java Arrays are Broken
mathjax: true
tags: [programming, oop, functional, java, type-systems]
---

In the article [(One Reason) Why Java Arrays are Broken](2022-08-01-java-array-variance.md)
I explained that the typing of Java arrays is broken because they are
always covariant in the type of object stored, but not all uses of
arrays happen in covariant contexts, leading to bugs, which can be
avoided using invariant generics, and fixed properly using generics
with the correct kind of variance: covariance for producer containers
(containers that provide input values for computations)
and contravariance for consumer containers (containers that get
assigned output values of computations). In this article we will see
how this relates to covariant and contravariant functors by modeling
mutation functorially and monadically.

# What Even is Mutation, What Even is Assignment?

It helps to think of mutation in a monadic way.
- Every time a mutation happens, it can be modeled as a State monad
- Every monad is a functor
- Functors are containers
- Functors are either contravariant or covariant
- What is the variance of the State monad
- I don't see the State monad in Java. Point it out to me please! :)))
