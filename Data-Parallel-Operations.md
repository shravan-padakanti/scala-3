> This is also covered in the next course: https://github.com/rohitvg/scala-spark-4/wiki/Reduction-Operations-(fold,-foldLeft,-aggregate)

**In Scala, most collection operations can become data-parallel**. The `.par` call converts a sequential collection to a parallel collection.

```scala
(1 until 1000).par
    .filter(n => n % 3 == 0)
    .count(n => n.toString == n.toString.reverse)
```
However, some operations are not parallelizable.

## Non-Parallelizable Operations

### foldLeft operation

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
    xs.par.fold(Int.MinValue)(math.max) // use the min-element as the neutral element, and max func for folding.
    // We could have used: (x,y) => if (x>y) x else y instead of math.max
}
```

### Preconditions of the fold Operation
Given a list of "rock", "paper" and "scissors" strings, find out who won:
```scala
Array("paper", "rock", "paper", "scissors").par.fold("")(play)

def play(a: String, b: String): String = List(a, b).sorted match {
    case List("paper", "scissors") => "scissors"  // scissors beats papers
    case List("paper", "rock") => "paper"         // paper beats rock
    case List("rock", "scissors") => "rock"       // rock beats scissors
    case List(a, b) if a == b => a                // if users choose the same options
    case List("", b) => b                         // if one option is empty
}

// usage

play(play("paper", "rock"), play("paper", "scissors")) == "scissors" 
play("paper", play("rock", play("paper", "scissors"))) == "paper"    // same play but reorganized. Hence different answer.
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
Array('E', 'P', 'F', 'L').par.fold(0)((count, c) => if (isVowel(c)) count + 1 else count)
```

Question:

What does this snippet do?

* The program runs and returns the correct vowel count.
* The program is non-deterministic.
* The program returns incorrect vowel count.
* The program does not compile.

**Answer**:
The program does not compile. The signature of the fold operations says that the accumulator element must be the same type as the elements in the collection. Here elements are `char` but accumulator is an `int`. Also the fold operation can only produce values of the **same type** as the collection that it is called on, which is not the case here.


On the other hand, the `foldLeft` is more expressive than `fold`. Sanity check:
```scala
def fold(z: A)(op: (A, A) => A): A = foldLeft[A][z](op)
```

> Extra reading: http://stackoverflow.com/questions/16111440/scala-fold-vs-foldleft
> ```scala
> def fold[A1 >: A](z: A1)(op: (A1, A1) => A1): A1
> def foldLeft[B](z: B)(op: (B, A) => B): B
> ```
> This is the reason that `fold` can be implemented in parallel, while `foldLeft` cannot. This is not only because of the *Left part which implies that `foldLeft` goes from left to right sequentially, but also because the operator `op` cannot combine results computed in parallel -- it only defines how to combine the aggregation type `B` with the element type `A`, but not how to combine two aggregations of type `B`. The fold method, in turn, does define this, because the aggregation type `A1` has to be a supertype of the element type `A`, that is `A1 >: A`. This supertype relationship allows in the same time folding over the aggregation and elements, and combining aggregations -- both with a single operator.


## The aggregate Operation

Let’s examine the aggregate signature:
```scala
def aggregate[B](z: B)(f: (B, A) => B, g: (B, B) => B): B
// B is the folding type
// z is the accumulator
// f is the sequential folding operator
// g is the parallel folding operator
```
Thus, as it uses a combination of sequential and parallel operators, It is a combination of `foldLeft` and `fold`.

Revisiting our problem of counting the number of vowels in a character array using the `aggregate` method:

```scala
Array('E', 'P', 'F', 'L').par.aggregate(0)( (count, c) => if (isVowel(c)) count + 1 else count, _ + _ )
// the 0 (neutral element) and _ + _ (parallel reduction operator) form the monad.
```

## The Transformer Operations

So far, we saw the accessor combinators.

Transformer combinators, such as `map`, `filter`, `flatMap` and `groupBy`, do not return a single value, but instead return new collections as results.

