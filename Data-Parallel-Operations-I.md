**In Scala, most collection operations can become data-parallel**. The `.par` call converts a sequential collection to a parallel collection.

```scala
(1 until 1000).par
    .filter(n => n % 3 == 0)
    .count(n => n.toString == n.toString.reverse)
```
However, some operations are not parallelizable.

## Non-Parallelizable Operations

### fold operation

**Task**: implement the method sum using the `foldLeft` method.
```scala
def sum(xs: Array[Int]): Int = {
    xs.par.foldLeft(0)(_ + _)
}
```
**Question**: Does this implementation execute in parallel? Why not?

**Answer**: No. To see why not, we examine `foldLeft` more closely:

```scala
def foldLeft[B](z: B)(f: (B, A) => B): B
```
The accumulator is passed sequentially to each element. i.e. previous elements need to be updated before updating next elements. Hence this cannot be data-parallelized.

## Parallelizable Operations

### fold operation

Next, let’s examine the `fold` method signature:
```scala
def fold(z: A)(f: (A, A) => A): A
```

Unlike `foldLeft`, the `fold` operaion can process the elements in a reduction tree and so so it can execute in parallel.

We can re-approach our task above to implement the `sum` method on a List using `fold` instead of `foldLeft`: 

```scala
def sum(xs: Array[Int]): Int = {
    xs.par.fold(0)(_ + _)
}
```

Now we implement a `max` method also using `fold`:
```scala
def max(xs: Array[Int]): Int = {
    xs.par.fold(Int.MinValue)(math.max)
}
```

### Preconditions of the fold Operation
Given a list of ”paper”, ”rock” and ”scissors” strings, find out who won:
```scala
Array("paper", "rock", "paper", "scissors").par.fold("")(play)

def play(a: String, b: String): String = List(a, b).sorted match {
    case List("paper", "scissors") => "scissors"
    case List("paper", "rock") => "paper"
    case List("rock", "scissors") => "rock"
    case List(a, b) if a == b => a
    case List("", b) => b
}

play(play("paper", "rock"), play("paper", "scissors")) == "scissors"
play("paper", play("rock", play("paper", "scissors"))) == "paper"
```
Why does this happen? This is because the `play` operator is **commutative**, but not **associative**.

In order for the fold operation to work correctly, the following relations must hold:

```
f(a, f(b, c)) == f(f(a, b), c)
f(z, a) == f(a, z) == a
```
We say that the neutral element `z` and the binary operator `f` must form a **monoid**.

Commutativity does not matter for fold – the following relation is not necessary:
```
f(a, b) == f(b, a)
```

### Limitations of the `fold` Operation

Given an array of characters, use fold to return the vowel count:
```scala
Array(‘E‘, ‘P‘, ‘F‘, ‘L‘).par.fold(0)((count, c) => if (isVowel(c)) count + 1 else count)
```

Question:

What does this snippet do?

* The program runs and returns the correct vowel count.
* The program is non-deterministic.
* The program returns incorrect vowel count.
* The program does not compile.

**Answer**:


## The aggregate Operation

Let’s examine the aggregate signature:
```scala
def aggregate[B](z: B)(f: (B, A) => B, g: (B, B) => B): B
```
A combination of foldLeft and fold.

Count the number of vowels in a character array using the `aggregate` method:

```scala
Array(‘E‘, ‘P‘, ‘F‘, ‘L‘).par.aggregate(0)( (count, c) => if (isVowel(c)) count + 1 else count, _ + _ )
```

## The Transformer Operations

So far, we saw the accessor combinators.

Transformer combinators, such as `map`, `filter`, `flatMap` and `groupBy`, do not return a single value, but instead return new collections as results.

