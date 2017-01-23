**Performance** is the key motivation for parallelism

How to estimate it?

* empirical measurement
* asymptotic analysis

Asymptotic analysis is important to understand how algorithms scale when:

* inputs get larger
* we have more hardware parallelism available

We examine worst-case (as opposed to average) bounds

## Asymptotic analysis of sequential running time

You have previously learned how to concisely characterize behavior of sequential programs using the number of operations they perform as a function of arguments.
* inserting into an integer into a sorted linear list takes time `O(n)`, for list storing n integers
* inserting into an integer into a balanced binary tree of n integers takes time `O(log n)`, for tree storing n integers.

### Example 1: Simple function

Let us review these techniques by applying them to our sum segment example: Find time bound on sequential sumSegment as a function of s and t

```scala
def sumSegment(a: Array[Int], p: Double, s: Int, t: Int): Int = {
    var i = s; 
    var sum: Int = 0
    while (i < t) {
        sum = sum + power(a(i), p)
        i = i + 1
    }
    sum 
}
```
**Answer:** Linear bound: `W(s,t) = O(t − s)`, a function of the form: `c1(t − s) + c2`.

* t − s loop iterations
* a constant amount of work in each iteration

### Example 2: Sequential Recursive function

```scala
def segmentRec(a: Array[Int], p: Double, s: Int, t: Int) = {
  if (t - s < threshold) {
    sumSegment(a, p, s, t)
  } else {
    val m = s + (t - s)/2
    val (sum1, sum2)= ( segmentRec(a, p, s, m), segmentRec(a, p, m, t) )
    sum1 + sum2 
  } 
}
```
**Answer**:
we can construct a tree to find the time bound:
![recursive_function_analysis](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/recursive_function_analysis.png)
```
W(s,t) =  c1(t − s) + c2,             if t − s < threshold
          W(s, m) + W(m,t) + c3       otherwise, for m = ⌊(s + t)/2⌋
```
The instructor then simplifies the above to:
```
W(s,t) is in O(t − s). 
```
Thus the Sequential segmentRec is linear in `t − s`.

### Example 3: Parallel Recursive function

Same as the above code, except the `parallel()` method is used to get (`sum1`,`sum2`).

```scala
def segmentRec(a: Array[Int], p: Double, s: Int, t: Int) = {
  if (t - s < threshold) {
    sumSegment(a, p, s, t)
  } else {
    val m = s + (t - s)/2
    val (sum1, sum2)= parallel(segmentRec(a, p, s, m), segmentRec(a, p, m, t))
    sum1 + sum2 
  } 
}
```
**Answer**:
![recursive_function_parallel_analysis](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/recursive_function_parallel_analysis.png)
```scala
D(s,t) = c1(t − s) + c2,               if t − s < threshold
         max(D(s, m), D(m,t)) + c3     otherwise, for m = ⌊(s + t)/2⌋
```
The instructor then simplifies the above to:
```
O(N)       // N is the depth of the tree
```
This in turn gives us:
```
O(log(t-s))
```
Thus the parallel solution is much faster as it gives us logarithmic time as opposed to linear time in case of sequential solution.