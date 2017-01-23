## Basic parallel construct
Given expressions e1 and e2, how do we compute them in parallel and return the pair of results.

```
parallel(e1, e2)

         e1
     / ------ \      
o---            --->
     \ ------ /
         e2
```

To understand this, lets take a look at p-norm.

### Example: Computing p-norm
Given a vector as an array (of integers), compute its p-norm

A p-norm is a generalization of the notion of length from geometry 2-norm of a two-dimensional vector (a1, a2) is `(a1^2 + a2^2)^(1/2)`.

So we have 2 operations here. The sum of squares of the coordinates is operation1 and square root of the result is operation2.

#### n-dimensional vector
Let us represent the n-dimensional vector by an array. We can do the above computation in a "parallel" by going from `0 to m-1` and `m to n-1` separately and performing operation1 on both simultaneously, and then joining the results and doing operation2 on it. 

Thus, here we split the array into 2 segments and in parallel compute the operation1, and the join the results and compute the operation2. So we have
```scala
val (sum1, sum2) = parallel(sumSegment(a, p, 0, m1), sumSegment(a, p, m1, m2))
```

How do we do this if we want to break the array in 4 segments? We do a `parallel(parallel(sum1, sum2), parallel(sum3, sum4))`. So:
```scala
val ((sum1, sum2),(sum3,sum4)) = parallel( parallel(sumSegment(a, p, 0, m1), sumSegment(a, p, m1, m2)),
                                           parallel(sumSegment(a, p, m2, m3), sumSegment(a, p, m3, a.length)) )
```


We can **generalize this using Recursion**:
```scala
def pNormRec(a: Array[Int], p: Double): Int = {
  power(segmentRec(a, p, 0, a.length), 1/p)
  // like sumSegment but parallel
  def segmentRec(a: Array[Int], p: Double, s: Int, t: Int) = {
    if (t - s < threshold) sumSegment(a, p, s, t) // small segment: do it sequentially
    else {
      val m = s + (t - s)/2
      val (sum1, sum2) = parallel(segmentRec(a, p, s, m),
      segmentRec(a, p, m, t))
      sum1 + sum2 
    } 
  }
}
```

#### Signature of the parallel method
```scala
def parallel[A, B](taskA: => A, taskB: => B): (A, B) = { ... }
```
* returns the same value as given
* benefit: `parallel(a,b)` can be faster than `(a,b)`
* it takes its arguments as `call by name`, indicated with `=> A` and `=> B` so that both computations are not calculated before hand, which will happen if we use `call by value` here instead. 