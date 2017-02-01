## Scala Collections Hierarchy

* `Traversable[T]` – Root type of all collections. It is a collection of elements with type `T`, with operations implemented using `foreach`. Each collection that implements this type, has to implement the `foreach` method.
* `Iterable[T]` – It is subtype of `Traverssable[T]`. It is a collection of elements with type `T`, with operations implemented using `iterator`. Has richer methods than `Traversable[T]`.
* `Seq[T]` – an ordered sequence of elements with type `T`. It is ordered: every element is assigned to an index.
* `Set[T]` – a `set` of elements with type `T` (no duplicates)
* `Map[K, V]` – a `map` of keys with type `K` associated with values of type `V` (no duplicate keys)

These traits are extended by the concrete collections.

## Parallel Collection Hierarchy

Traits `ParIterable[T]`, `ParSeq[T]`, `ParSet[T]` and `ParMap[K, V]` are the parallel counterparts of different sequential collection traits. Operations on objects that implement these traits generally execute in parallel.

For code that is agnostic about parallelism, there exists a separate hierarchy of generic collection traits `GenIterable[T]`, `GenSeq[T]`, `GenSet[T]` and `GenMap[K, V]`. Operations on objects that implement these traits **may or may not** execute in parallel.

## Writing Parallelism-Agnostic Code

Generic collection traits allow us to write code that is unaware of parallelism. Thus this code can be invoked on sequential or parallel collections.

Example – find the largest palindrome in the sequence (eg. 15251):
```scala
def largestPalindrome(xs: GenSeq[Int]): Int = {
    xs.aggregate(Int.MinValue)(
        (largest, n) => if (n > largest && n.toString == n.toString.reverse) n else largest, math.max
    )
}

val array = (0 until 1000000).toArray
largestPalindrome(array)      // invoke on sequential collection
largestPalindrome(array.par)  // invoke parallel collection. Converts to ParArray
```

## Non-Parallelizable Collections

A sequential collection can be converted into a parallel one by calling `par`.
```scala
val vector = Vector.fill(10000000)("")
val list = vector.toList

vector.par // creates a ParVector[String]
list.par   // also creates a ParVector[String]. Takes more time than vector for the parallel conversion.
```

## Parallelizable Collections

* `ParArray[T]` – parallel array of objects, counterpart of 'Array' and 'ArrayBuffer'
* `ParRange` – parallel range of integers, counterpart of 'Range'
* `ParVector[T]` – parallel vector, counterpart of 'Vector'
* `immutable.ParHashSet[T]` – counterpart of 'immutable.HashSet'
* `immutable.ParHashMap[K, V]` – counterpart of 'immutable.HashMap'
* `mutable.ParHashSet[T]` – counterpart of 'mutable.HashSet'
* `mutable.PasHashMap[K, V]` – counterpart of 'mutable.HashMap'
* `ParTrieMap[K, V]` – thread-safe parallel map with atomic snapshots, counterpart of 'TrieMap'
* for other collections, 'par' creates the closest parallel collection – e.g. a 'List' is converted to a 'ParVector'.

The conclusion is that unless the conversoin to a prallel collection takes a negligible amount of time compared to subsequent parallel operations, then pick the data structures carefully and make sure that they are parallelizable.

## Computing Set Intersection

```scala
// takes 2 generic sets, and returns a set
def intersection(a: GenSet[Int], b: GenSet[Int]): Set[Int] = {
    val result = mutable.Set[Int]()
    for (x <- a) if (b contains x) result += x
    result
}

intersection((0 until 1000).toSet, (0 until 1000 by 4).toSet)
intersection((0 until 1000).par.toSet, (0 until 1000 by 4).par.toSet)
```

Question: Is this program correct?

* Yes.
* No.

Answer: No. If the program is executed for sequential sets, then it runs fine. But when executed for parallel sets, then the `result` is accessed concurrently as the mutable.Set is not a thread-safe class.

## Side-Effecting Operations

**Rule**: Avoid mutations to the same memory locations without proper synchronization. Eg the variable `result` above.

## Synchronizing Side-Effects

Solution – use a concurrent collection, which can be mutated by multiple threads:

```scala
import java.util.concurrent._
def intersection(a: GenSet[Int], b: GenSet[Int]) = {
    val result = new ConcurrentSkipListSet[Int]() // <----------------instead of a mutable.Set
    for (x <- a) if (b contains x) result += x
    result
}

intersection((0 until 1000).toSet, (0 until 1000 by 4).toSet)
intersection((0 until 1000).par.toSet, (0 until 1000 by 4).par.toSet)
```
This runs fine for both sequential and parallel collections.


## Avoiding Side-Effects
Side-effects can be avoided by using the correct **combinators**. For example, we can use filter to compute the intersection:

```scala
def intersection(a: GenSet[Int], b: GenSet[Int]): GenSet[Int] = {
    if (a.size < b.size) a.filter(b(_)) // a.filter(b(_)) is the combinator
    else b.filter(a(_))
}

intersection((0 until 1000).toSet, (0 until 1000 by 4).toSet)
intersection((0 until 1000).par.toSet, (0 until 1000 by 4).par.toSet)
```

## Concurrent Modifications During Traversals

**Rule**: Never modify a parallel collection on which a data-parallel operation is in progress.

Example: here we create a cyclic graph where every node has 1 successor. 
```scala
val graph = mutable.Map[Int, Int]() ++= (0 until 100000).map(i => (i, i + 1))
graph(graph.size - 1) = 0
for ((k, v) <- graph.par) graph(k) = graph(v) // successor is replaced with successors successor.
val violation = graph.find({ case (i, v) => v != (i + 2) % graph.size }) // every node should have the successor, which is exactly by two larger than itself. The only exception are the nodes at the very end. For those, we have to do the division, modulo graph.size. 
println(s"violation: $violation") // no violations for a correct program
```
This program prints violations as there are 2 errors:
1. We modify the same collection that we are traversing in parallel
2. We read from a collection that is concurrently being modified by some other iteration of the loop

So:
* Never write to a collection that is concurrently traversed: `for ((k, v) <- graph.par) graph(k) = graph(v)`
* Never read from a collection that is concurrently modified. `graph(k) = graph(v)`

In either case, program non-deterministically prints different results, or crashes.

## The TrieMap Collection
`TrieMap` is an **exception **to these rules. This concurrent collection automatically creates atomic snapshots whenever a parallel operation starts. So concurrent updates are not observed during parallel traversals. 

The `snapshot` method (snapshots are taken in constant time i.e. `O(1)`) can be used to efficiently grab the current state:

```scala
val graph = concurrent.TrieMap[Int, Int]() ++= (0 until 100000).map(i => (i, i + 1))
graph(graph.size - 1) = 0
val previous = graph.snapshot()
for ((k, v) <- graph.par) graph(k) = previous(v)
val violation = graph.find({ case (i, v) => v != (i + 2) % graph.size })
println(s"violation: $violation")
```
This program runs correctly and does not print any violations.