---
title: birthday problem
date: 2013-03-22
---

Recently at work we came across a case where we needed to generate up to 10,000
random unique numbers. We had to fit it into 23 bits, giving us roughly 8
million different numbers to choose from.

We'd all done combinatorics before, so we knew that if we were to randomly
generate these numbers, the chance of there being a collision [isn't going to
be as low as what our intuition tells
us](http://en.wikipedia.org/wiki/Birthday_problem). But none of us were really
that fluent with our math, so when we plugged our formula into
[WolframAlpha](http://www.wolframalpha.com/) and it spit out 99.8% chance of a
collision, we were sure that the problem was with our formula and not with the
scenario.

I ended up testing the situation empirically in the Scala REPL and it turns out
the math was right after all. Here's the template for empirically testing the
classic birthday problem:

```scala
import scala.util.Random

def sample(size: Int, limit: Int): Seq[Int] =
  Stream.continually(Random.nextInt(limit)).take(size)

def isUnique(sample: Seq[Int]): Boolean =
  sample.distinct.size == sample.size

def collisionChance(size: Int, limit: Int, times: Int): Double =
  (Stream
    .continually(sample(size, limit))
    .take(times)
    .count(isUnique)
    .toDouble / times)

assert(collisionChance(23, 365, 10000) ~= 0.5)
```
