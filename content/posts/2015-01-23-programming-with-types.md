---
title: programming with types
date: 2015-01-23
tags: [haskell, types]
---

One of the things that really grows on you after programming with Haskell for a
while is the idea that the types alone actually reveal quite a lot about what a
program does. The implementation is almost a secondary concern.

To a programmer who has worked in side-effecting languages their whole life,
it's very unnerving to learn that the most common way to search the Haskell
documentation is with a type signature. It's arguably even more disconcerting to
see something as terse as this in the official documentation:

```
const :: a -> b -> a
base Prelude, base Data.Function

Constant function.
```

Uh... cool. So what does `const` actually do?

## generics are very generic

To learn how to read type signatures, you first have to understand that generic
arguments are precisely that: they are generic. Haskell has no class hierarchy
because it's not an OOP language. In Java, since all types inherit from
`Object`, at a minimum you can always call `toString()` on a generic type, like
so:

```java
<A> void foo(A a) {
    System.out.println(a.toString());
}
```

In Haskell, absolutely _nothing_ is known about a generic type other than the
fact that it is of that type. Trying to compile the Haskell equivalent of the
Java above:

```haskell
foo :: a -> IO ()
foo = print
```

results in a compile error:

```
No instance for (Show a)
  arising from a use of `print'
In the expression: print
In an equation for `foo': foo = print
```

You can't even test for equality! This code snippet:

```haskell
bar :: a -> Bool
bar x = x == x
```

also results in a compile error:

```
No instance for (Eq a)
  arising from a use of `=='
In the expression: x == x
In an equation for `bar': bar x = x == x
```

`Show` and `Eq` are examples of what is called a typeclass. On a high level,
these are similar to interfaces in Java: for example, `Show` tells the Haskell
compiler that `show`, the equivalent of Java's `toString`, is available for that
type. The only way to make these functions compile is to explicitly say that you
expect the type `a` to have instances declared for the appropriate typeclass:

```haskell
foo :: (Show a) => a -> IO ()
foo = print

bar :: (Eq a) => a -> Bool
bar x = x == x
```

These compile. The closest Java equivalent would be something like this:

```java
<A extends Show> void foo(A a) {
    System.out.println(a.show());
}
```

The takeaway here is that Haskell is _that_ strict about its types. If you don't
explicitly say that a generic type can do something, then the only operation you
can perform on it is to return itself.

## arguments as the sole inputs

Next, you have to remember that Haskell is a pure language. That means that
given the same inputs for a function, the output will always be the same, a
property known as _referential transparency_ (it's actually slightly more
complicated than that, but it's close).

This, for example, means that global states are out. This Java function returns
a different value every time it is called:

```java
int bar = 0;
int foo(int a) {
    return a + bar++;
}
```

An equivalent can't be implemented in Haskell, at least not with the same type
signature. It is simply impossible.

## examples

That brings us directly to the most straightforward example, the `id` function:

```haskell
id :: a -> a
```

By now you should be able to see why there can only ever be one implementation
given this type signature. The implementation is simply:

```haskell
id :: a -> a
id x = x
```

This is because there is no way to find a value of type `a` within the context
of the function and because there are no operations that you can perform on a
value of the generic type `a`.

Let's move onto a slightly more complex example. Here is the type signature:

```haskell
apply :: (a -> b) -> a -> b
```

So `apply` is function that takes two arguments: the first is a function that
takes an `a` and returns a `b`, and the second is a value of type `a`. The
`apply` function itself must return a value of type `b`. How can it get its
hands on this value?

The only possible way is to call the function passed to it with the value that
was also passed to it, leading to this implementation:

```haskell
apply :: (a -> b) -> a -> b
apply f x = f x
```

Those of you who have used Haskell will recognise this as the `($)` operator.

By now the only implementation for the example I gave in the introduction should
be apparent:

```haskell
const :: a -> b -> a
const x _ = x
```

or, if you prefer:

```haskell
const :: a -> b -> a
const x = \_ -> x
```

Through the magic of currying, `const x` returns a function that ignores its
input and always produces the predefined constant result `x`.

## closing thoughts

And so on. The epiphany for me was the realisation that, given the constraints
of the Haskell programming language, type signatures are in fact very unique.

I could look a signature like this

```haskell
(a -> b) -> [a] -> [b]
```

and understand immediately that the only productive implementation for this
signature yields the `map` function. Or a signature like this:

```haskell
(a -> b -> b) -> b -> [a] -> b
```

and immediately see a `fold` function (`foldr` specifically in Haskell).

From experience I can say that it certainly takes some time to get used to
reading and understanding functions in this manner. It's a fundamental change in
how you interpret code. But once you've made the connection and internalised
this concept you don't ever think about code in quite the same way again.
