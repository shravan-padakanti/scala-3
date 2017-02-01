## Data-Parallel Abstractions

We will study the following abstractions:

1. [Iterators](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Splitters-and-Combiners#1-iterators)
2. [Splitters](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Splitters-and-Combiners#2-splitter)
3. [Builders](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Splitters-and-Combiners#3-builder)
4. [Combiners](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Splitters-and-Combiners#4-combiner)

### 1. Iterators

The simplified `Iterator` trait is as follows:

```scala
trait Iterator[A] {
    def next(): A
    def hasNext: Boolean
}

def iterator: Iterator[A] // on every collection
```

The _iterator contract_:
* `next` can be called only if `hasNext` returns true
* after `hasNext` returns `false`, it will always return `false`

#### Using iterators

How would you implement `foldLeft` on an iterator?

```scala
def foldLeft[B](z: B)(f: (B, A) => B): B = {
    var s = z
    while (hasNext) s = f(s, next())
    s
}
```

### 2. Splitter

The simplified `Splitter` trait is as follows:

```scala
trait Splitter[A] extends Iterator[A] {
    def split: Seq[Splitter[A]]
    def remaining: Int
}

def splitter: Splitter[A] // on every parallel collection
```

The _splitter contract_:
* after calling 'split', the original splitter is left in an undefined state
* the resulting splitters traverse disjoint subsets of the original splitter
* 'remaining' is an estimate on the number of remaining elements
* 'split' is an efficient method – 'O(log n)' or better

#### Using splitters

How would you implement `fold` on a splitter?

```scala
def fold(z: A)(f: (A, A) => A): A = {
    if (remaining < threshold) foldLeft(z)(f)
    else {
        val children = for (child <- split) yield task { child.fold(z)(f) }
		children.map(_.join()).foldLeft(z)(f)
    }
}
```

### 3. Builder
The simplified 'Builder' trait is as follows:

```scala
trait Builder[A, Repr] {
    def +=(elem: A): Builder[A, Repr]
    def result: Repr
}

def newBuilder: Builder[A, Repr] // on every collection
```

The _builder contract_:

* calling 'result' returns a collection of type 'Repr', containing the elements that were previously added with +=
* calling 'result' leaves the Builder in an undefined state

#### Using splitters

How would you implement the 'filter' method using 'newBuilder'?

```scala
def filter(p: T => Boolean): Repr = {
    val b = newBuilder
    for (x <- this) if (p(x)) b += x
    b.result
}
```

### 4. Combiner

The simplified 'Combiner' trait is as follows:

```scala
trait Combiner[A, Repr] extends Builder[A, Repr] {
    def combine(that: Combiner[A, Repr]): Combiner[A, Repr]
}

def newCombiner: Combiner[T, Repr] // on every parallel collection
```

The _combiner contract_:

* calling 'combine' returns a new combiner that contains elements of input combiners
* calling 'combine' leaves both original Combiners in an undefined state
* 'combine' is an efficient method – 'O(log n)' or better

#### Using Combiners

How would you implement a parallel 'filter' method using 'splitter' and 'newCombiner'?