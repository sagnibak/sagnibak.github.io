---
title: "Python is the Haskell You Never Knew You Had: Currying"
# subtitle: A tale of a mathematical epiphany
mathjax: true
tags: [programming, functional, python, proof, induction]
---

# What is Currying?

It is the process of taking a function that takes multiple arguments and
turning it into a function that takes one argument and returns another
function which takes fewer arguments. For example, let's take a look at
the function below:

```python
def add(a, b):
    return a + b
```

A curried version of the function would look like this:
```python
def add(a):
    def add_a(b):
        return a + b
    return add_a
```
or like this:
```python
add = lambda a: lambda b: a + b
```

Now `add(3)` returns a function, but `add(3)(4)` returns the integer `7`. You
can store the intermediate function as `add3 = add(3)` and call it later, like
`add3(2)` and `add3(4)`.

# Why Currying?

This is one way that the functional programmer saves and passes around state.
This is also a great way to reduce expensive computation, especially when the
curried functions are pure. Sometimes it can also be a stylistic choice to
curry functions. I for one prefer the look of `add(2)(3)` much better than
`add(2, 3)`. (On a side note, I absolutely love `(add 2 3)` and absolutely
abhor `add 2 3`, iykyk.) A quick google search will give you several examples
of this, so I will not spend too much time on the "why". This article is really
about the "how". If you are looking for examples, you can check out
[this](https://lukajcb.github.io/blog/scala/2016/03/08/a-real-world-currying-example.html)
post that I found on the interwebs.

I should note here that in a language where functions are first-class objects,
and which supports closures (functions defined inside other functions which
capture, or have access to, the local variables of the enclosing function),
such as Python, Haskell, Scheme, etc., curried functions can be just as powerful
as their non-curried versions, since it is possible to curry any non-curried
function and vice versa. Following is a rather simple proof of this by induction.

If a function $f_n$ takes $n$ arguments,
then you can turn that into a function $c_n$ which takes one argument and
returns a function $c_{n-1}$ that takes $n-1$ arguments, and has access to
the argument that was passed to $c_n$ (hence $c_{n-1}$ is a closure). You
can keep doing this until you get to $c_1$, which takes one argument and
calls $f_n$ on the $n$ arguments received by $c_{n}, c_{n-1}, \ldots, c_1$,
thus returning the same result as $f_n$, the non-curried function. Below
is an example of that:

```python
def f_5(a, b, c, d, e):
    return a + b + c + d + e

def c_5(a):
    def c_4(b):
        def c_3(c):
            def c_2(d):
                def c_1(e):
                    return f_5(a, b, c, d, e)
                return c_1
            return c_2
        return c_3
    return c_4
```

And of course, if you have the curried function $c_n$, then you can get $f_n$
from that by simply calling $c_n$ $n$ times with the arguments supplied to
$f_n$, as shown below:

```python
def c_5(a):
    def c_4(b):
        def c_3(c):
            def c_2(d):
                def c_1(e):
                    return a + b + c + d + e
                return c_1
            return c_2
        return c_3
    return c_4

def f_5(a, b, c, d, e):
    return c_5(a)(b)(c)(d)(e)
```

This proves that there is a one-to-one correspondence between curried functions
and functions which take $n$ arguments for any positive natural number $n$.

What about functions that take no arguments? Pure functions which take no
arguments must necessarily be constant, hence they can be replaced by their
output in all of their occurrences. And impure functions ~~can create unicorns~~
are much harder to analyze so I am not talking about them here.

# Currying Everything in Python

Disclaimer: Do this at your own discretion, and only with pure functions.

While I absolutely love looking at `c_5(a)(b)(c)(d)(e)`, I absolutely abhor
its definition. One could simplify it to
```python
c_5 = lambda a: lambda b: lambda c: lambda d: lambda e: a + b + c + d + e
```
but that does not look particularly appealing either. Only Haskell as a
somewhat neat way of writing that:
```haskell
c_5 = \a -> \b -> \c -> \d -> \e -> a + b + c + d + e
```
But it still obfuscates the true nature of the function `c_5`, which is
that it takes five arguments and adds them all up, which is expressed
most concisely in the definition
```python
def c_5(a, b, c, d, e):
    a + b + c + d + e
```
But wait, now `c_5` is not curried. Python's `functools` module has a
function called `partial` which lets you partially apply functions, as
follows:
```python
from functools import partial

c_4 = partial(c_5, 1)
c_3 = partial(c_4, 9)
...
```
While this is a way to get the equivalent of currying out of a non-curried
function, it is a bit verbose in my opinion. What if we could add a decorator
to the definition of `c_5` that would curry it? I am thinking about something like
```python
@curry
def c_5(a, b, c, d, e):
    a + b + c + d + e
```

# The Currying Decorator

Now we need to make some design decisions. Python is a very flexible language.
The user can call a function with positional or keyword arguments. Some
arguments are optional, so they user might never supply them. Also, the biggest
reason why currying is useful is that it allows the programmer to save state
when needed, and sometimes you just _want_ to call a curried function like
`curried(1, 2, 3)` instead of `curried(1)(2)(3)`, since after all you defined
your curried function as `def curried(a, b, c)` with a decorator on top.
I want this decorator to handle all of that elegantly, and it is actually not
that hard.

I decided on defining a function `curry` which takes in an integer `num_args`
and returns a decorator that curries a function which takes at least `num_args`
arguments, and once that many arguments have been provided, the wrapped function
gets called with all the positional and keyword arguments. For example, `c_5`
would be defined as follows:
```python
@curry(num_args=5)
def c_5(a, b, c, d, e):
    a + b + c + d + e
```
To see more examples of this, please look at the [doctests](https://github.com/sagnibak/func_py/blob/master/curry.py#L18)
of `curry` in
[the github repo with all the code](https://github.com/sagnibak/func_py).
And please also read the entire [docstring](https://github.com/sagnibak/func_py/blob/master/curry.py#L7).

## Saving State, and Partial Applications

Let's try to implement `curry`. We want a function that returns a decorator
function, so it should look something like this:

```python
def curry(num_args):
    def decorator(fn):
        def inner(*args, **kwargs):
            ...
```

If `inner` is provided at least `num_args` arguments in total (positional plus
keyword arguments) then it should simply call `fn(*args, **kwargs)`, otherwise
it should return a partial application. What is a partial application? It is
something that stores the arguments given to it, and can be called with more
arguments. If it is given at least `num_args` arguments then it should call
`fn(*args, **kwargs)`, otherwise it should return another partial computation.
We can implement such behavior in a class `Partial`:

```python
class Partial:
    def __init__(self, num_args, fn, *args, **kwargs):
        self.num_args = num_args
        self.fn = fn
        self.args = args
        self.kwargs = kwargs

    def __call__(self, *more_args, **more_kwargs):
        all_args = self.args + more_args  # tuple addition
        all_kwargs = dict(**self.kwargs, **more_kwargs)  # non-mutative dictionary union
        num_args = len(all_args) + len(all_kwargs)
        if num_args >= self.num_args:
            return self.fn(*all_args, **all_kwargs)
        else:
            return Partial(self.num_args, self.fn, *all_args, **all_kwargs)
```

Note that the `__call__` magic method in Python makes an object callable,
meaning that now we can call `Partial` objects as if they were functions. Now
we can implement `curry` rather simply:

```python
def curry(num_args):
    def decorator(fn):
        def inner(*args, **kwargs):
            if len(args) + len(kwargs) >= num_args:
                return fn(*args, **kwargs)
            else:
                return Partial(num_args, fn, *args, **kwargs)
        
        return inner
    return decorator
```

We can actually simplify this code a little more, since a
`Partial` object with no `args` and no `kwargs` behaves exactly the same as any
other `Partial` object. We can get rid of the function `inner` entirely and
get a rather elegant implementation of `curry`:

```python
def curry(num_args):
    def decorator(fn):
        return Partial(num_args, fn)
    return decorator
```

This should give you an appreciation for what is really happening when we are
currying: we are storing state and passing it around, and that is why sometimes
it is exactly what you want.

## A Purely Functional Definition

If you look closely at the code above, you will notice that `Partial` objects
are never mutated, and also that only two methods are ever called on those
objects: `__init__` while initializing them, and `__call__` while calling them
as a function (in code that presumably uses the curried function). It is possible
to completely get rid of the `Partial` class and replicate this same behavior
with two functions `init` and `call` inside `decorator`:

```python
def curry(num_args):
    def decorator(fn):
        def init(*args, **kwargs):
            def call(*more_args, **more_kwargs):
                all_args = args + more_args
                all_kwargs = dict(**kwargs, **more_kwargs)
                if len(all_args) + len(all_kwargs) >= num_args:
                    return fn(*all_args, **all_kwargs)
                else:
                    return init(*all_args, **all_kwargs)

            return call
        return init()
    return decorator
```

Notice that `call` the same body as `Partial.__call__` and that the boilerplate
code in `__init__` is completely gone. This implementation of `curry`, called
[`curry_functional`](https://github.com/sagnibak/func_py/blob/master/curry.py#L110)
in the [github repo](https://github.com/sagnibak/func_py/blob/master/curry.py),
is purely functional. I still prefer the implementation using `Partial`
since one would have to draw out the function calls on paper to
understand what is going on in this implementation, whereas the first
types on the first implementation make a lot more sense and highlight the true
nature of what is going on, i.e., the passing around of partially-applied
functions. Indeed the types on the functional implementation are an atrocious
nesting of `Callable`'s and they don't inform the reader at all, so I left them
out in the github repo, but the types on the first implementation tell a very
nice story.

Despite the stylistic shortfalls of the functional implementation, I am including
it here to make a very important point: it is possible to pass around state
not just in objects but in functions (more precisely, closures) as well. Case
in point, we were able to replace the `Partial` object with the functions that
define it, since it is an immutable object (it is technically immutable, but 
it is possible to make it immutable in Python by deleting the `__setattr__`
method in `__init__`, and more importantly, it does not _need_ to be mutated).
This suggests that maybe we can re-implement classes using functions only! This
will be the topic of a future (the next?) blog post.

# Conclusion

While Python does not curry its functions by default (unlike Haskell), there is
a clean, elegant way to implement the same functionality. This also suggests
that functions can save and pass around state on their own, and that classes
are just syntax sugar for certain kinds of functions. Finally, it
reflects the fact that Lambda calculus, which is a framework where variable
declaration, and function definition and application are the only possible
actions, is enough to model all of computation.

All the code in this post can be found
[here](https://github.com/sagnibak/func_py/blob/master/curry.py).
