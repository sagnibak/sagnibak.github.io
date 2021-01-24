---
title: Identity Matters in Deep Learning
subtitle: An executive summary
mathjax: true
tags: [deep learning, theory, resnet, neural networks]
---

<!-- # The Importance of Learning the Identity Transformation -->

# Notation and Terminology

Throughout this discussion we will be talking about fully-connected neural
networks (multilayer perceptron networks, which I abbreviate as MLP) which
have $H$ hidden layers, i.e., which are the composition of $H+1$ linear
transformations, where each of the hidden layers has the same number of units,
which is also the same as the number of input units. By an $l$-layer neural
network I mean a network consisting of $l$ linear transformations, i.e.,
$l = H+1$. Concretely, an $l$-layer MLP is the following function:

$$
\begin{align*}
    f(x) = W_l \cdots W_1 x,
\end{align*}
$$

where $W_{l-1},\ldots,W_1 \in \mathbb{R}^{d \times d}$ where $d$ is the number
of features in the input, and $W_l \in \mathbb{R}^{k \times d}$ where $k$ is
the number of output dimensions.

We will always be minimizing the expected squared error loss given some labels
$y$:

$$
\begin{align*}
    \min_{W_1,\ldots,W_l} \mathbb{E}\left[\|y - f(x)\|_2^2 \right].
\end{align*}
$$

Any variable with a single capital letter name,
including $W_i$ for any integer $i$, is a matrix.
I write the spectral norm (largest singular value) of a matrix
$W_i$ as $\\|W_i\\|$.

Under this setting there is no network that an $l$-layer neural network can
learn which an $(l+1)$-layer neural network can't. However, the authors of the
[ResNet paper](https://arxiv.org/abs/1512.03385) noted that in practice it
becomes harder and harder to train MLP networks as depth increases. Thus even
though deeper MLPs can _represent_ a superset (indeed a strict superset with
ReLU activations) of the functions that can be represented by shallower ones,
it becomes harder to _train_ the MLP networks as depth increases, leading to
poorer convergence and thus poorer performance. They designed residual neural
networks (ResNets) to fix this problem.

# The Big Problem and a Logical Solution

Before we learn about the solution, let's articulate the problem.
The reason an $(l+1)$-layer MLP can always represent any function that an
$l$-layer MLP can represent is that it is always possible for the "extra"
layer to learn the identity transformation. However, something being
_possible_ does not equate to it being _easy_. Indeed if we initialize
the MLP to a zero-centered distribution with small variance, which is
necessary for gradient descent to not blow up, then it is extremely hard
for _any_ layer to learn the identity transformation, since as the number
of layers increases, the gradient to any one layer gets attenuated even more,
slowing down learning.

This hints at a solution: making it easy for any layer to model the identity
transformation. Doing so (in hindsight) is no rocket science at all. We
simply need to replace $f: \mathbb{R}^d \to \mathbb{R}^k$ from above with
$g: \mathbb{R}^d \to \mathbb{R}^k$ defined as:

$$
\begin{align*}
    g(x) = (W_l + I) \cdots (W_1 + I) x.
\end{align*}
$$

Now when we initialize the $W_i$ matrices near zero, the network represents
a transformation close to the identity transformation instead of the zero
transformation. This lets gradients flow much better through the network, and
more importantly, this makes it easy for any layer to learn the identity
transformation. Empirically this has led practitioners to train previously
unthinkably deep neural networks, even one with 1200 layers  with "no
optimization difficulty" (see page 8 of 
[this](https://arxiv.org/abs/1512.03385) paper).
This solution carries over quite well to convolutional neural networks as well,
which Kaiming He et al. discuss at length in
[this](https://arxiv.org/abs/1512.03385) paper.

# Theoretically Awesome

[Moritz Hardt](https://mrtz.org/) and
[Tengyu Ma](https://ai.stanford.edu/~tengyuma/) prove two amazing theorems
about deep residual neural networks in their paper
["Identity Matters in Deep Learning"](https://arxiv.org/abs/1611.04231)
restricted to the regime where all activation functions are linear:

1. For a deep residual neural network with enough layers $l$ that being trained
   to minimize the squared error loss, there exists a global minimum such that
   the spectral norm of all of the matrices $W_i$ is bounded above by
   $\\mathcal{O}(1/l)$. This means none of the matrices will need to learn a
   value far away from its near-zero initialization.

2. In the ball $\\mathcal{B}\_{\\tau} := \\{ (W_1,\\ldots,W_l) : \\max_{1\\le i\\le l} \\|W_i\\| \\le \\tau \\}$  where the spectral norm of all the matrices $W_i$ is bounded
   by $\tau$, if $\tau < 1$, then any critical point of the loss function
   must be a _global minimum_.

The second point is stronger than any claim we can make about MLPs with linear
activations. While there are no sub-optimal local minima, there are loci of
saddle points with zero curvature whenever $l \ge 3$, as proved by Kenji
Kawaguchi in
["Deep Learning Without Poor Local Minima"](https://arxiv.org/abs/1605.07110).
This means gradient-based optimization algorithms like batch gradient descent
will not get stuck at any points other than the global minimizer when training
deep (linear) residual neural networks with initialization suitably close to
zero, while the same cannot be said about deep linear MLPs.

For proofs of the claims, please:
- read the original papers,
- take a look at my slides [here](https://drive.google.com/file/d/16uywAEvCSSCu2AIxRGSkjp34z9WrLqTe/view?usp=sharing),
- watch a video of my talk [here]().

# References

- Hardt, Moritz, and Tengyu Ma. “Identity Matters in Deep Learning.” ArXiv.org, 20 July 2018, [arxiv.org/abs/1611.04231](https://arxiv.org/abs/1611.04231).
- He, Kaiming, et al. “Deep Residual Learning for Image Recognition.” 2016 IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2016, [doi:10.1109/cvpr.2016.90](https://ieeexplore.ieee.org/document/7780459).
- Kawaguchi, Kenji. “Deep Learning without Poor Local Minima.” ArXiv.org, 27 Dec. 2016, [arxiv.org/abs/1605.07110](https://arxiv.org/abs/1605.07110).
