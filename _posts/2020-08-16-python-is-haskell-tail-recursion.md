---
title: "Python is the Haskell You Never Knew You Had: Tail Call Optimization"
# subtitle: A tale of a mathematical epiphany
mathjax: true
tags: [recursion, programming, functional, python]
---

# Tail Calls

<!-- A tail call is when you make a function call in a tail context. A tail context
is a location in a program from where the called function can return to the
current function's caller, i.e., the current stack frame can be eliminated
before the tail call. -->

Consider the factorial function below:

<script src="https://gist.github.com/sagnibak/256dfb22b3ac6bfebd52f6a97c5f7030.js"></script>

<!-- {% highlight python linenos %}
def fac(n, acc=1):
    if n == 0 or n == 1:
        return acc
    else:
        return fac(n - 1, n * acc)
{% endhighlight %} -->

When we make the call `fac(3)`, two recursive calls are made: `fac(2, 3)` and
`fac(1, 6)`. The last call returns `6`, then `fac(2, 3)` returns `6`, and
finally the original call returns `6`. I would recommend looking at the execution
in [Python Tutor](https://www.pythontutor.com/visualize.html#code=def%20fac%28n,%20acc%3D1%29%3A%0A%20%20%20%20if%20n%20%3D%3D%200%20or%20n%20%3D%3D%201%3A%0A%20%20%20%20%20%20%20%20return%20acc%0A%20%20%20%20else%3A%0A%20%20%20%20%20%20%20%20return%20fac%28n%20-%201,%20n%20*%20acc%29%0A%20%20%20%20%20%20%20%20%0Afac%283%29&cumulative=false&curInstr=0&heapPrimitives=nevernest&mode=display&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false):

<iframe width="800" height="400" frameborder="0" src="https://pythontutor.com/iframe-embed.html#code=def%20fac%28n,%20acc%3D1%29%3A%0A%20%20%20%20if%20n%20%3D%3D%200%20or%20n%20%3D%3D%201%3A%0A%20%20%20%20%20%20%20%20return%20acc%0A%20%20%20%20else%3A%0A%20%20%20%20%20%20%20%20return%20fac%28n%20-%201,%20n%20*%20acc%29%0A%20%20%20%20%20%20%20%20%0Afac%283%29&codeDivHeight=400&codeDivWidth=350&cumulative=false&curInstr=0&heapPrimitives=nevernest&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false"> </iframe>

If you look carefully, you can see that first a huge call stack is created,
then a base case is reached, and then the return value is simply bubbled back
up to the `fac(3)` call, which simply hands that value back to the global
frame. This happens because after the recursive call is made by the caller,
no further computation needs to be done by the caller. This kind of function
call is called a tail call, and languages like Haskell, Scala, and Scheme can
avoid keeping around unnecessary stack frames in such calls. This is called
tail call optimization (TCO) or tail call elimitation.

This is useful because the computation of `fac(n)` without TCO requires
$\mathcal{O}(n)$ space to hold the $n$ stack frames and for large $n$, this
causes the stack to overflow, whereas with TCO this would take $\mathcal{O}(1)$
memory, since a constant number of stack frames is used regardless of $n$.

The optimized code should look much like the iterative version of factorial
below:

<script src="https://gist.github.com/sagnibak/934fc6ec181ea60907eecdc1aae0e2f7.js"></script>

<!-- {% highlight python linenos %}
def fac(n):
    acc = 1
    while n > 1:
        acc *= n
        n -= 1
    return acc
{% endhighlight %} -->

As you can see below, this only creates a constant number of (one) stack frame:

<iframe width="800" height="370" frameborder="0" src="https://pythontutor.com/iframe-embed.html#code=def%20fac%28n%29%3A%0A%20%20%20%20acc%20%3D%201%0A%20%20%20%20while%20n%20%3E%201%3A%0A%20%20%20%20%20%20%20%20acc%20*%3D%20n%0A%20%20%20%20%20%20%20%20n%20-%3D%201%0A%20%20%20%20return%20acc%0A%0Afac%283%29&codeDivHeight=400&codeDivWidth=350&cumulative=false&curInstr=0&heapPrimitives=nevernest&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false"> </iframe>

Of course, this code uses a loop and mutation, so as a diligent functional
programmer I will deride it and instead suggest that we restrict such behavior
to a single function and abstract it away behind a decorator, so that we can
make pristine tail calls in Python and also not blow away the stack.

# Tail Recursive Functions to Loops

Notice that the variables `n` and `acc` are the ones that change in every
iteration of the loop, and those are the parameters to each tail recursive
call. So maybe if we can keep track of the parameters and turn each recursive
call into an iteration in a loop, we will be able to avoid recursive calls.

The decorator should be a higher-order function which takes in a function `fn`
and returns an inner function which when called, calls `fn`, but with some
scaffolding. `fn` must follow a specific form: it must return something which
instructs the inner function (often called the trampoline function) whether it
wants to recurse or return. For this, we need two classes representing the two
cases:

<script src="https://gist.github.com/sagnibak/b3570a15b50ac9b629937cfa8a9df498.js"></script>

<!-- {% highlight python linenos %}
@dataclass
class TailCall:
    args: Tuple[Any, ...] = field(default_factory=tuple)
    kwargs: Dict[str, Any] = field(default_factory=dict)

@dataclass
class Return:
    return_val: Any
{% endhighlight %} -->

`fn` should return an instance of `TailCall` when it wants to make a tail
recursive call, and it should feed the arguments of the next call into the
instance. When it wants to simply return without making a recursive call,
it should return an instance of `Return`, which wraps the return value. The
decorator looks like this:

<script src="https://gist.github.com/sagnibak/0412b44e768703752aad9e0a65ae27cc.js"></script>

<!-- {% highlight python linenos %}
def tco(fn):
    def inner(*args, **kwargs):
        result = fn(*args, **kwargs)
        while isinstance(result, TailCall):
            result = fn(*result.args, **result.kwargs)
        else:
            return result.return_val
    return inner
{% endhighlight %} -->

Finally, `fac` looks like this:

<script src="https://gist.github.com/sagnibak/d5be5cb3319c1de2ad0590c8332a591f.js"></script>

<!-- {% highlight python linenos %}
@tco
def fac(n, acc=1):
    if n == 0:
        return Return(acc)
    else:
        return TailCall((n - 1, n * acc))
{% endhighlight %} -->

And thus we have achieved the functional ideal: restricting mutation and loops
to a single location, which in this case is the decorator `tco`, without any
(severe) overhead. (Note that a good compiler would look at the original `fac`
and replace the entire function body with a loop to guarantee zero overhead.)
Notice how there is only a single stack frame belonging to the function `fac`
at any point in time. This will let you compute `fac(1000)` and beyond without
a stack overflow error!

<iframe width="800" height="700" frameborder="0" src="https://pythontutor.com/iframe-embed.html#code=%23pythontutor_hide_type%3A%20class,%20function%0A%0Aclass%20TailCall%3A%0A%20%20%20%20def%20__init__%28self,%20args%3DNone,%20kwargs%3DNone%29%3A%0A%20%20%20%20%20%20%20%20self.args%20%3D%20args%20if%20args%20is%20not%20None%20else%20%28%29%0A%20%20%20%20%20%20%20%20self.kwargs%20%3D%20kwargs%20if%20kwargs%20is%20not%20None%20else%20%7B%7D%0A%0A%0Aclass%20Return%3A%0A%20%20%20%20def%20__init__%28self,%20return_val%29%3A%0A%20%20%20%20%20%20%20%20self.return_val%20%3D%20return_val%0A%0Adef%20tco%28fn%29%3A%0A%20%20%20%20def%20inner%28*args,%20**kwargs%29%3A%0A%20%20%20%20%20%20%20%20result%20%3D%20fn%28*args,%20**kwargs%29%0A%20%20%20%20%20%20%20%20while%20isinstance%28result,%20TailCall%29%3A%0A%20%20%20%20%20%20%20%20%20%20%20%20result%20%3D%20fn%28*result.args,%20**result.kwargs%29%0A%20%20%20%20%20%20%20%20return%20result.return_val%0A%20%20%20%20return%20inner%0A%0A%0A%40tco%0Adef%20fac%28n,%20acc%3D1%29%3A%0A%20%20%20%20if%20n%20%3D%3D%200%3A%0A%20%20%20%20%20%20%20%20return%20Return%28acc%29%0A%20%20%20%20else%3A%0A%20%20%20%20%20%20%20%20return%20TailCall%28%28n%20-%201,%20n%20*%20acc%29%29%0A%0Afac%283%29&codeDivHeight=400&codeDivWidth=350&cumulative=false&curInstr=0&heapPrimitives=nevernest&origin=opt-frontend.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false"> </iframe>

And this is how you implement tail call optimization in a language which does
not have native support for it. Below is a Github Gist with all the code, some
examples, and static types.

<script src="https://gist.github.com/sagnibak/cff833adc6833034fd76f27f168c2c82.js"></script>
