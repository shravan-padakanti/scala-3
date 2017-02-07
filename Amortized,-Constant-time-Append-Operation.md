## Constant Time Appends in Conc-Trees

Let’s use Conc-Trees to implement a `Combiner`.

How could we implement += method?

```scala
var xs: Conc[T] = Empty
def +=(elem: T) {
    xs = xs <> Single(elem)
}
```
This takes `O(log n)` time – can we do better than that?

## Constant Time Appends in Conc-Trees

To achieve `O(1)` appends with low constant factors, we need to extend the Conc-Tree data structure.

We will introduce a new Append node with different semantics:

```scala
case class Append[T](left: Conc[T], right: Conc[T]) extends Conc[T] {
    val level = 1 + math.max(left.level, right.level)
    val size = left.size + right.size
}
```

One possible `appendLeaf` implementation:

```scala
def appendLeaf[T](xs: Conc[T], y: T): Conc[T] = Append(xs, new Single(y))
```
Can we still do `O(log n)` concatenation? I.e. can we eliminate Append nodes in `O(log n)` time?

This implementation breaks the `O(log n)` bound on the concatenation.

![const_time_appends](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/const_time_appends.png)

The fundamental problem here is that we are essentially still building a link list with append nodes. So we need to link these notes more intelligently. In what follows, we will make sure that if the total number of elements in the tree is n, then there are never more than log n of append nodes in the data structure. 

### Counting in Binary Number System

![binary_counting](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/binary_counting.png)

* To count up to `n` in the binary number system, we need `O(n)` work.
* A number `n` requires `O(log n)` digits

Also,

* To add `n` leaves to an Append list, we need `O(n)` work.
* Storing `n` leaves requires `O(log n)` Append nodes

### Binary Number Representation

* **0** digit corresponds to a missing tree
* **1** digit corresponds to an existing tree

## Constant Time Appends in Conc-Trees

```scala

def appendLeaf[T](xs: Conc[T], ys: Single[T]): Conc[T] = xs match {
    case Empty => ys
    case xs: Single[T] => new <>(xs, ys)
    case _ <> _ => new Append(xs, ys)
    case xs: Append[T] => append(xs, ys)
}

@tailrec private def append[T](xs: Append[T], ys: Conc[T]): Conc[T] = {
    if (xs.right.level > ys.level) new Append(xs, ys)
    else {
        val zs = new <>(xs.right, ys)
        xs.left match {
            case ws @ Append(_, _) => append(ws, zs)
            case ws if ws.level <= zs.level => ws <> zs
            case ws => new Append(ws, zs)
        }
    }
}
```

We have implemented an _immutable_ data structure with:

* `O(1)` appends
* `O(log n)` concatenation

Next, we will see if we can implement a more efficient, _mutable_ data Conc-tree variant, which can implement a `Combiner`.