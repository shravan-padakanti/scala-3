Having seen parallel versions of [map](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Parallel-map()#parallel-map-of-an-array-producing-an-array) and [fold](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Parallel-fold()#parallel-reduce-of-a-tree) , we now examine parallel `scanLeft`.

`scanLeft` produces a collection containing cumulative results of applying the operator going left to right.

```scala
List(1,3,8).scanLeft(100)((acc,elem) => acc + elem) == List(100, 101, 104, 112)
```
Thus, just like `foldLeft`, `scanLeft` scans from left and passes the accumulator to each element in the list. Note that in `foldLeft` we fold the list into a final value. Here the output is a list.

### scanLeft - meaning and properties
```scala
List(a1, a2, a3).scanLeft(f)(a0) = List(b0, b1, b2, b3)
```
where
```
* b0 = a0
* b1 = f(b0, a1)
* b2 = f(b1, a2)
* b3 = f(b2, a3)
```

We **assume that f is assocative**, throughout this segment. 

`scanRight` is different from `scanLeft`, even if f is associative:
```scala
List(1,3,8).scanLeft(100)((acc,elem) => acc + elem) == List(100, 101, 104, 112)
List(1,3,8).scanRight(100)((acc,elem) => acc + elem) == List(112, 111, 108, 100)
```
We consider only scanLeft, but scanRight is dual.

## Sequential scan
```scala
List(a1, a2, ..., aN).scanLeft(f)(a0) = List(b0, b1, b2, ..., bN)
```
where 
```
* b0 = a0 
* bi = f(bi−1, ai) for 1 <= i <= N.
```
We give a sequential definition of `scanLeft`:

* take an array `inp`, an element `a0`, and binary operation `f`
* write the output to array `out`, assuming `out.length >= inp.length + 1`

```scala
def scanLeft[A](inp: Array[A], a0: A, f: (A,A) => A, out: Array[A]): Unit = {
    out(0)= a0
    var a= a0
    var i= 0
    while (i < inp.length) {
        a = f(a,inp(i))
        i = i + 1
        out(i)= a
    }
}
```

## Parallel scan

Can scanLeft be made parallel? Assume that f is associative.

**Goal**: an algorithm that runs in O(log n) given infinite parallelism

At first, the task seems impossible; it seems that:
* the value of the last element in sequence depends on all previous ones
* need to wait on all previous partial results to be computed first
* such approach gives O(n) even with infinite parallelism

**Idea**: give up on reusing all intermediate results

* do more work (more f applications)
* improve parallelism, more than compensate for recomputation

### High-level approach: express scan using map and reduce

Can you define result of scanLeft using `map` and `reduce`? Assume input is given in array `inp` and that you have `reduceSeg1` and `mapSeg` functions on array segments:

```scala
def reduceSeg1[A](inp: Array[A], left: Int, right: Int,
a0: Int, f: (A,A) => A): A
def mapSeg[A,B](inp: Array[A], left: Int, right: Int,
fi : (Int,A) => B,
out: Array[B]): Unit

