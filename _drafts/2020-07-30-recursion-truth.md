---
title: Recursion is a Search for Truth
subtitle: A tale of a mathematical epiphany
mathjax: true
tags: [recursion, programming, proofs]
---

# A Benign Linked List

Here's a small implementation of a linked list in Python, complete with
an `__eq__` method to determine the equality of two linked lists. 

<!-- ```python -->
{% highlight python linenos %}
class LinkedList:
    def __eq__(self, other):
        if self == Nil and other == Nil:
            return True
        elif isinstance(self, Cons) and isinstance(other, Cons):
            return (self.val == other.val) and (self.next == other.next)
        else:
            return False

class NilClass(LinkedList):
    pass

Nil = NilClass()  # singleton end marker

class Cons(LinkedList):
    def __init__(self, val, next=Nil):
        self.val = val
        self.next = next
{% endhighlight %}
<!-- ``` -->

For example, `Cons(1, Cons(2, Nil)) == Cons(1, Cons(2, Nil))` evaluates to
`True` as do `Cons(2, Nil) == Cons(2, Nil)` and `Cons(1, Nil) != Cons(3, Nil)`.

# RecursionError

That's what you actually get when you run the first two of the three snippets.
Why? Think about what happens when you try to determine if `Nil == Nil`. Python
converts that into the method call `Nil.__eq__(Nil)`. Then, in line 3, `self`
is bound to `Nil`, so `Nil.__eq__(Nil)` is called again. This leads to infinite
recursion.

We can fix this by replacing line 3 with either of:

{% highlight python linenos %}
        if self is Nil and other is Nil:
{% endhighlight %}

or 

{% highlight python linenos %}
        if isinstance(self, NilClass) and isinstance(other, NilClass):
{% endhighlight %}

The distinction between line 3 in the first code block and the two replacements
is the essence of why recursion works (and doesn't work at other times).