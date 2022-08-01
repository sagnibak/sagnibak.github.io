---
title: (One Reason) Why Java Arrays are Broken
mathjax: true
tags: [programming, oop, functional, java, type-systems]
---

# TL;DR: Type Variance.

Arrays in Java are covariant containers, but they can
be used in contravariant contexts, leading to mayhem.

# Two Textbook Examples

The following code (adapted from an example in Joshua Bloch's
[Effective Java](https://www.oreilly.com/library/view/effective-java/9780134686097/),
Item 28) is valid Java, insofar that the compiler doesn't complain:

```java
String[] strings = new String[2];
Object[] obs = strings;
obs[0] = 3;
```

The last line will fail at runtime with an `ArrayStoreException`, since `obs`
knows that it is an array of `String` and not `Integer`, and this type
constraint is enforced at runtime.

But why were we able to store a `String[]` in a variable of type
`Object[]`? Well, javac says, a `String` is an `Object`, so if you can
do something with an `Object[]` then you should also be able to do the
same thing with a `String[]`. For example, consider the following
(contrived) function that sums up the hashes of all objects in `obs`:

```java
long sumHash(Object[] obs) {
    long result = 0;
    for (Object ob : obs) {
        result += ob.hashCode();
    }
    return result;
}
```

This function can be called on `String[]`, `Integer[]`, etc., indeed
on an array of any type that inherits from `Object`, precisely because
it only uses features provided by the supertype `Object` instead of
using any features of a more specialized type like `String`.

Since an `Object[]` only allows you to use features of the `Object`
class on its elements, javac concludes, being able to cast an array of
any subtype of `Object` to `Object[]` makes perfect sense. This is an
example of **covariance**: whenever we have a type `A` and another type
`B` that extends `A`, every `B[]` is also an `A[]` since every `B` is
also an `A`.

# What's the Difference?

Why does `sumHash` work as expected but the first example fail? Some
would jump to blame mutability, but that's not the root cause of our
troubles. It helps to extract the problematic part of the first
example into a function:

```java
void assignInt(Object[] obs) {
    obs[0] = 69;
}
```

The function `assignInt` takes an `Object[]` of length at least 1 and
assigns an (autoboxed) `Integer` to its first index. A call to this
code will fail with an `ArrayStoreException` if the type of `obs` is,
for instance, `String[]`. (One might argue that the signature of
this function should be `void assignInt(Integer[])`, but that is a
question of programming style, which is out of the scope of this
post.)

But there are _some_ types for which this function will behave as
expected. The following calls _will_ succeed:
- `assignInt(new Integer[1])`
- `assignInt(new Number[1])`
- `assignInt(new Object[1])`

Note, however, that the following calls _will_ fail at runtime
(although not at compile time):
- `assignInt(new String[1])`
- `assignInt(new Double[1])`

Notice a pattern here? An array of any <b>super</b>type of `Integer`
is a valid input to `assignInt`. When used as an input to `assignInt`,
`Object[]` is a <b>sub</b>type of `Integer[]`, since an `Object[]` can
be used in place of an `Integer[]`. This is an example of
**contravariance** of types.

This is in stark contrast to
`sumHash`, where an `Object[]` is a <b>super</b>type of `Integer[]`,
since an `Integer[]` can be used in place of an `Object[]`, which we
postulated earlier as an example of covariance.

And therein lies the problem: Java expects arrays to be covariant in
all contexts, but allows them to be used in contravariant contexts,
leading to errors that cannot be caught at compile time. If it were
possible to specify that `assignInt` expects an array of any supertype
of `Integer[]` whereas `sumHash` expects an array of any subtype of
`Object` then that would solve our problems.

# And we can Actually do that in Java

using generics! Java generics are <b>in</b>variant by default, meaning
that even if we have two types `A` and `B` where `B` is a subtype of
`A`, a `List<B>` is neither a supertype nor a subtype of `List<A>`.
At first blush this seems less useful, since, for instance, the
function `long sumHash(List<Object>)` cannot be called on an instance
of `List<String>`, nor can the function `void
assignInt(List<Integer>)` be called on a `List<Number>`, even though
both would clearly work as expected. In order to communicate type
variance to the compiler, we need to write generic functions instead.
The following functions work as expected, and take variance into
account explicitly using bounded wildcards:

- `void sumHash(List<? extends Object>)`: This will take a list of any
  type that extends `Object`. Note that the bound `? extends Object`
  can be replaced with `?` since every type is a subtype of `Object`.
- `void assignInt(List<? super Integer>)`: This will take a list of
  any type that is a supertype of `Integer`.

There is a general principle about type variance at play here, which
Bloch summarizes as "producer-`extends`, consumer-`super` (PECS)" in Item 31 of
[Effective Java](https://www.oreilly.com/library/view/effective-java/9780134686097/)
which I will delve much deeper into in a following post, where I
discuss a fair amount of Category Theory and the general principle of
variance and the specific ways in which it applies to programming with
generic types.

In summary, we can obtain compile-time type safety and appropriate type variance
using generic containers in place of arrays. For this reason it is
generally considered good practice to use generic containers instead
of arrays unless the (usually minimal) performance penalty is
unbearable.

# References
- [Effective Java](https://www.oreilly.com/library/view/effective-java/9780134686097/)
  by Joshua Bloch
- [The Java Language Specification](https://docs.oracle.com/javase/specs/jls/se11/html/index.html)
