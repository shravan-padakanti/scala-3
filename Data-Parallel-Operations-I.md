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