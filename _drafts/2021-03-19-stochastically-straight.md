---
title: Stochastically Straight
subtitle: A queer speedrun of my statistics degree
mathjax: true
tags: [probability, statistics, cs70]
---

# The World is Stochastic

Our minds love the bliss of determinism. Toss a coin (which is what
statisticians do day in day out), it either shows its "heads" side or its
"tails" side. Yet it is neither a head, nor a tail. It is a coin, which can
stochastically present itself either as a "head" or a "tail". Determinism
is simple, but this simplicity does not let us express the range of
possibilities that a coin can manifest.
Determinism is thus a terrible model of reality. Several phenomena in the
real world are inherently stochastic. I viscerally felt this when I started
coming to terms with my bisexuality. I don't want to be in a relationship
with a man and a woman at the same time, that's polyamory. At one instant
of time I am either straight or gay. Yet I am neither straight nor gay, I
am something else. I am stochastically straight and stochastically gay. The
label for that is "bisexual", making it inherently stochastic. Fortunately
I spent two and a half years studying stochastic things at UC Berkeley so
let's see what we can do with that.

# From Intuition to Formalization

In order to be able to answer statistical questions rigorously, we need to
formalize a few quantities. The fundamental quantity we will be exploring
here is the probability that I like someone given they are a girl. Let's
call this probability $p_g$. In other words,

$$
\mathrm{Pr}\left[\text{I like a person} \mid \text{they are a girl} \right]
= p_g.
$$

Note that this does **not** tell us anything about the probability that I
like a person given that they are a dude. That probability is **not**
$(1-p_g)$, which is merely the probability that I will **not** like a person
(romantically) given that they are a girl. So we need a second quantity
$p_d$, which is the probability that I like a person given that they are a
dude:

$$
\mathrm{Pr}\left[\text{I like a person} \mid \text{they are a dude} \right]
= p_d.
$$

Turns out I like dudes a little more than girls. What would that mean
mathematically? I am more likely to like a person given that they are a
dude than given that they are a girl, i.e., $p_d > p_g$. But how different
are these two quantities? They must have numerical values, right? How can
we find those values? We will explore such questions in the rest of the article.

Also, let's define

$$
q_g = 1 - p_g, \\
q_d = 1 - p_d.
$$

We will see soon that these quantities will be useful.

# Some Bayes-ic Calculations

(Please pardon the pun. As a principled statistician, I must include one of
these in every statistics work of mine.) I already told you that I like
dudes more than girls. Yet you see me on more dates with girls than with
dudes (you actually don't because I don't go on dates, but humor me for a
second, will ya?). How does that make any sense?

# Something Fishy

Let's say that I upload all my dates to Facebook, so Daddy Zucc is able to
keep track of all my dates. How many dates should he expect to have to
observe before concluding that I am bisexual?

My first date is either a dude or a girl. If it is the former, then the
first time Zucc sees me date a girl, he will know that I am bisexual. If my
first date is a girl, then the logic is similar.

# Estimating Probabilities

# Bayesian Estimation

# Testing Hypotheses

# Remarks and Clarifications

- When I talk about men and women in this article, I am referring to
  phenotypes and not genders per se. For example, for the purposes of this
  article, a male-presenting non-binary person would be a man. I chose not
  to consider non-binary genders explicitly here for a few reasons:
  - The math becomes substantially more annoying when we have to work with
    multinomial distributions instead of binomial distributions.
  - The non-binary spectrum is very large and I don't (yet) think I am pansexual
    so it wouldn't be a true inquiry into my sexuality anyway.
  - Oh and did I mention how much easier the math is with a dichotomy
    instead of a trichotomy? The latter adds a substantial amount of
    complexity for minimal pedagogical benefit.
