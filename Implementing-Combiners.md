Here we will see how the combiners are implemented for various collections, and learn about data structures that are more amenable for parallel computations.

So while the emphasis in the previous week was on data parallel APIs and parallel programming obstructions, this week's lecture will be more algorithmic in nature. It will give you an insight into how to pick the right data structure, and organize the data in a parallel program.

[Recall](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Data-Parallel-Operations#the-transformer-operations) that a transformer operation is a collection operation that creates another collection instead of just a single value. Methods such as `filter`, `map`, `flatMap` and `groupBy` are examples of transformer operations. By contrast, methods, such as `fold`, `sum`, and `aggregate` are not transformer operations. 

We have seen Builders and Combiners in the [last lecture](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Splitters-and-Combiners): 

## Builders

Builders are used in sequential collection methods:
```scala
trait Builder[T, Repr] {
    def +=(elem: T): this.type
    def result: Repr
}
```

## Combiners

Combiners can be used in parallel collection methods:
```scala
trait Combiner[T, Repr] extends Builder[T, Repr] {
    def combine(that: Combiner[T, Repr]): Combiner[T, Repr]
}
```

How can we implement the `combine` method efficiently?

* when Repr is a set or a map, combine represents union (Eg. `Set(1,2,3) U Set(2,3,4) = Set(1,2,3,4)`)
* when Repr is a sequence, combine represents concatenation (Eg. `Seq(1,2,3) U Seq(2,3,4) = Seq(1,2,3,2,3,4)`)

In both cases, the combine operation must be efficient, i.e. execute in `O(log n + log m)` time, where n and m are the sizes of two input combiners. Else, parallel processing is slower than sequential or does not provide any speedup.

**Question**: Is the method combine efficient?

```scala
// concatenate 2 arrays to produce a third array
def combine(xs: Array[Int], ys: Array[Int]): Array[Int] = {
    val r = new Array[Int](xs.length + ys.length)
    Array.copy(xs, 0, r, 0, xs.length)
    Array.copy(ys, 0, r, xs.length, ys.length)
    r
}
```
* Yes.
* No.

**Answer:**
The above function takes O(n+m) time. Since it is not logarithmic as we want, it is not efficient.


## Array Concatenation

Arrays **cannot** be efficiently concatenated (as seen in the above example).

## Sets

Typically, set data structures have efficient lookup, insertion and deletion.

* hash tables – `expected O(1)`
* balanced trees – `O(log n)`
* linked lists – `O(n)`

Most set implementations do not have efficient union operation.

## Sequences

Operation complexity for sequences can vary.

* mutable linked lists – constant time `O(1)` prepend and append, linear time `O(n)` insertion
* functional (cons) lists – constant time `O(1)` prepend operations, everything else linear time `O(n)`
* array lists – amortized constant time `O(1)` append, `O(1)` random accesss, otherwise linear time `O(n)`

Mutable linked list can have `O(1)` concatenation, but for most sequences, concatenation is `O(n)`.

