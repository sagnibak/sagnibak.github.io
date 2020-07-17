---
title: How to Write Combinatorial Proofs
subtitle: Why knowing how to count can save you a lot of algebra
mathjax: true
tags: [cs70, combinatorial proofs, proofs, counting]
---

It is a fact that ${n \choose k} = {n \choose n - k}$. Below is a proof:

$$
\begin{align*}
{n \choose k} &= \frac{n!}{k!(n-k)!} \\
              &= \frac{n!}{(n-n+k)!(n-k)!}\\
              &= \frac{n!}{(n-k)!(n - (n - k))!} \\
              &= {n \choose n - k}
\end{align*}
$$

We just used some algebraic manipulation to write the proof, but this is
bad for multiple reasons.
* This proof tells you nothing about counting, which combinatorics is all
about.
* This technique can get out of hand really fast when you want to prove something
very related like ${n \choose k} = {n - 1 \choose k - 1} + {n - 1 \choose k}$.

Here is another proof. The left hand side counts the number ways of selecting
$k$ objects out of $n$ total objects, by definition of $n \choose k$. If we can
show that the right hand side is another way to count the same thing, then we
will have a complete proof.

The right hand side counts the number of ways of choosing $n-k$ items out of $n$.
But secretly the RHS is counting the number of ways of choosing $k$ items out of
$n$: it is counting the $k$ items that do *not* get selected after we select $n-k$
items. This completes the proof. This is the same as if you have a bunch of candies
and you choose the ones you like, then you also chose which ones you did *not* like.

# The Three-Step Recipe

Every combinatorial proof contains three keys steps:
1. Identify the thing that both the LHS and RHS are counting.
2. Explain how the LHS is counting that.
3. Explain how the RHS is counting that.

Often one of 2. and 3. is very easy, and the other one is more involved.

# Examples
## Pascal's Equality

[Pascal's Equality](https://en.wikipedia.org/wiki/Pascal%27s_rule) states that

$${n \choose k} = {n - 1 \choose k - 1} + {n - 1 \choose k}.$$

Below is a combinatorial proof of this.

1. Both sides are counting the number of ways of choosing $k$ elements out of $n$.
2. By definition, $n \choose k$ is the number of ways of choosing $k$ elements out of $n$.
3. Let the $n$ items we are choosing from be $x_1, x_2, \ldots, x_n$. Any subset of these
elements will either contain $x_1$ or it won't contain $x_1$ (this may seem obvious but
in fact it is the key). If we choose $x_1$ then there are $n-1$ items remaining and we
need to choose $k-1$ items from them. There are ${n - 1 \choose k - 1}$ ways of doing that.
If we don't choose $x_1$ then also we are left with $n-1$ items but we need to choose $k$
items out of them, which we can do in ${n - 1 \choose k}$ ways. Hence the total number of
ways of choosing $k$ elements out of $n$ is also expressed by
${n - 1 \choose k - 1} + {n - 1 \choose k}$.

This completes the proof.

<!--
## The Hockey-Stick Identity

The [hockey-stick identity](https://en.wikipedia.org/wiki/Hockey-stick_identity) states

$$
\sum_{i=k}^n {i \choose k} = {n + 1 \choose k + 1}.
$$

For example, if we let $n=4$ and $k=2$, we have the identity

$$
{2 \choose 2} + {3 \choose 2} + {4 \choose 2} = {5 \choose 3}.
$$

Let's attempt a combinatorial proof.

1. Both sides are counting the number of ways to select $k+1$ items out of $n+1$ total items.
2. The right hand side counts this by definition.
3. The left hand side though.
-->

## Ternary Strings

A ternary string of length $n$ is a string of the digits 0, 1, and 2 which contains $n$ digits.
For example, 122002 is a ternary string of length 6 and 00022 is a ternary string of length 5.
Now, here is a seemingly unrelated identity:

$$
\sum_{k=0}^n 2^k {n \choose k} = 3^n.
$$

Try to prove this on your own first. The title of this section is a big hint.

Another hint: how many ternary strings of length $n$ contain $n$ 2's? How many of them contain
$(n-1)$ 2's? How many contain no 2's? How many contain $k$ 2's?

Below is a combinatorial of this identity.

1. Both sides count the number of ternary strings of length $n$.
2. In a ternary string of length $n$ there are $n$ slots, and for each slot, you have three
choices: 0, 1, 2. Hence you are making 3 choices for each one of $n$ slots. For every choice
for the first slot, you can make 3 choices for the second slot, and so on. Hence there are
$3^n$ ways of constructing ternary strings of length $n$. For those of you who like balls and
bins, this is the same as throwing $n$ distinct balls into 3 distinct bins, with each ball
corresponding to a slot and each bin corresponding to a digit. There are $3^n$ ways of throwing
the balls into the bins without any restrictions.
3. If we stipulate that there are $n-k$ 2's in a ternary string of length $n$, then we first
need to choose $k$ locations to place numbers which are *not* 2 (i.e., numbers which are 0 or 1).
There are ${n \choose k}$ ways of doing that. Then, for each of those $k$ locations we must choose
whether to put a 0 or a 1 in that location. There are $2^k$ decisions to be made here. Hence,
there are $2^k {n \choose k}$ ternary strings of length $n$ which contain exactly $n-k$ 2's.
But a ternary string of length $n$ can contain anywhere from 0 to $n$ 2's, hence we need to sum
over the possible number of 2's in order to get the total number of ternary strings:
$\sum_{k=0}^n 2^k {n \choose k}$.

This is completes the proof.

# Conclusion

The art of writing combinatorial proofs lies in being able to identify exactly what both
sides are trying to count, which can take some practice to master. In my experience, trying to
frame the problem in terms of balls and bins, forming a team, and constructing strings helps
in most cases. A really common trick is breaking the counting problem up into multiple
steps, like we did in step 3 of the last two examples above, and appropriately adding or
multiplying the results of the different steps. The thing that I really like about combinatorial
proofs is that coming up with them forces you to think of variables not as abstract symbols that
we manipulate according to a fixed set of rules, but as real things that you can see, reason
about, and count.
