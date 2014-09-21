---
layout: post
title: "\"weak\" transpose in Haskell"
redirect_from:
  - /post/51229034879/
  - /post/51296868625/
---

Today at work I came across an interesting problem: given a list of lists,
construct a new list by shuffling or transposing them such that one item from
each list is taken across all lists until they are all empty. For example, given
a list

{% highlight ruby %}
[[1, 2, 3], [4, 5], [6, 7, 8]]
{% endhighlight %}

output

{% highlight ruby %}
[1, 4, 6, 2, 5, 7, 3, 8]
{% endhighlight %}

If the lists were all the same length, then this would be the
[transpose](http://en.wikipedia.org/wiki/Transpose) operation (edit: plus a
flatten). However, I wanted this to work for any list of lists, no matter the
length, by just ignoring any lists that become empty during the algorithm.

I work with Scala at work but only have a rudimentary understanding of
functional programming. So a few weeks ago I started reading [A Gentle
Introduction to Haskell](http://www.haskell.org/tutorial/) to gain a better
understanding of functional concepts (in particular about the idea of
[monads](http://www.haskell.org/tutorial/monads.html)).

Given this, I thought this problem would be rather nice to tackle with Haskell.
This is the algorithm I came up with:

{% highlight haskell %}
transpose :: [[a]] -> [a] -> [a]
transpose [] output = output
transpose input output = transpose
  (filter (\x -> not (null x)) (map (\x -> tail x) input))
  (output ++ (map (\x -> head x) input))

print (transpose [[1, 2, 3], [4, 5], [6, 7, 8]] [])
-- prints [1, 4, 6, 2, 5, 7, 3, 8]
{% endhighlight %}

Bouncing ideas off my colleagues, we realised that this algorithm could be
defined completely recursively with the simplest base case possible: if input is
empty, then return output. The magic all happens in the recursive call:

* The new input is going to be the list of the original lists in the input minus
the head (first) element, because we’re going to take those and append to our
output. This is done by mapping the tail function over each list. To satisfy the
“weakness” constraint, empty lists are then filtered out.

* The cumulative output can be constructed by simply appending the head of each
list in our input to it. Again, this can be done by mapping the head function
over each list.

And that’s it! Once the algorithm had been fleshed out, it turned out to be
surprisingly succinct and straightforward. I am constantly amazed by how
powerful recursive definitions of algorithms can be, and it really shows off how
useful functional programming is.

Finally, I rewrote this in Scala, wrapping the recursive function inside a nicer
external interface so that it wouldn’t be invoked incorrectly:

{% highlight scala %}
def transpose[T](input: List[List[T]]) = {
  def iter[T](input: List[List[T]], output: List[T]): List[T] =
    if (input.isEmpty) output
    else iter(
      input map { _.tail } filter { !_.isEmpty },
      output ::: (input map { _.head })
    )
  iter(input, Nil)
}

println(transpose(List(List(1, 2, 3), List(4, 5), List(6, 7, 8))))
// prints List(1, 4, 6, 2, 5, 7, 3, 8)
{% endhighlight %}

*edit:*

A colleague pointed out that strictly speaking the tail recursive function isn’t
required. Since the JVM doesn’t have [tail
recursion](http://en.wikipedia.org/wiki/Tail_call) optimisation, the Scala
compiler can only optimise a recursive function if the very last thing done in
the function is the call to itself, which means the accumulator has to be
included in the arguments for the function.

Without the optimisation, the algorithms can be simplified considerably:

{% highlight haskell %}
transpose :: [[a]] -> [a]
transpose [] = []
transpose input = (map (\x -> head x) input) ++
  transpose (filter (\x -> not (null x)) (map (\x -> tail x) input))
{% endhighlight %}

{% highlight scala %}
def transpose[T](list: List[List[T]]): List[T] =
  if (list.isEmpty) Nil
  else (list map { _.head }) :::
    transpose(list map { _.tail} filter { !_.isEmpty })
{% endhighlight %}

*edit2*:

And after a bit more thought, it can be made more concise even with the explicit
tail recursion:

{% highlight haskell %}
transpose :: [[a]] -> [a]
transpose [] = []
transpose input = map head input ++
  transpose (filter (not . null) (map tail input))
{% endhighlight %}
