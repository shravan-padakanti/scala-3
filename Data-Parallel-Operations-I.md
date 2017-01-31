**In Scala, most collection operations can become data-parallel**. The `.par` call converts a sequential collection to a parallel collection.

```scala
(1 until 1000).par
    .filter(n => n % 3 == 0)
    .count(n => n.toString == n.toString.reverse)
```
However, some operations are not parallelizable.

## Non-Parallelizable Operations

**Task**: implement the method sum using the foldLeft method.
```scala
def sum(xs: Array[Int]): Int = {
    xs.par.foldLeft(0)(_ + _)
}
```
**Question**: Does this implementation execute in parallel? Why not?

**Answer**: 