**Performance** is the key motivation for parallelism

How to estimate it?

* empirical measurement
* asymptotic analysis

Asymptotic analysis is important to understand how algorithms scale when:

* inputs get larger
* we have more hardware parallelism available

We examine worst-case (as opposed to average) bounds

## Asymptotic analysis of running time

You have previously learned how to concisely characterize behavior of sequential programs using the number of operations they perform as a function of arguments.
* inserting into an integer into a sorted linear list takes time `O(n)`, for list storing n integers
* inserting into an integer into a balanced binary tree of n integers takes time `O(log n)`, for tree storing n integers.

### Example 1: Sequential simple function

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
**Answer**: We have the same tree as above, as the recursive calls are same but just in parallel:
![recursive_function_analysis](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/recursive_function_analysis.png)
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

## Work and depth

We would like to speak about the asymptotic complexity of parallel code, but it depends on available parallel resources.  So we introduce two measures for a program:

1. **Work W(e)**: number of steps the expression e would take if there was no parallelism
    * this is simply the sequential execution time
    * treat all `parallel(e1, e2)` as `(e1, e2)`
2. **Depth D(e)**: number of steps if we had unbounded parallelism
    * we take maximum of running times for arguments of parallel

### Rules for depth (span) and work

Key rules are:
* W(parallel(e1, e2)) = W(e1) + W(e2) + c2         // sum of work done for each expression 
* D(parallel(e1, e2)) = max(D(e1), D(e2)) + c1     // max of work done on each branch of the tree

If we divide work in equal parts, for depth it counts only once!

For parts of code where we do not use parallel explicitly (thus call-by-value args), we must add up costs. For function call or operation `f(e1, ..., en)`:
* W(f(e1, ..., en)) = W(e1) + ... + W(en) + W(f)(v1, ..., vn)
* D(f(e1, ..., en)) = D(e1) + ... + D(en) + D(f)(v1, ..., vn)

Here `vi` denotes values of `ei`. If `f` is primitive operation on integers, then `W(f)` and `D(f)` are constant functions, regardless of `vi`.

Note: we assume (reasonably) that constants are such that `D ≤ W`.

### Computing time bound for given parallelism

We use this estimate:
```
D(e) + W(e)/P
```
where P is the no. of parallel threads.

Given `W` and `D`, we can estimate how programs behave for different `P`:
* If `P` is constant but inputs grow, parallel programs have same asymptotic time complexity as sequential ones
* Even if we have infinite resources, (P -> Infinity), running time does not go to zero, but depends on the depth of the tree i.e. `D(e)`

### Consequences for segmentRec

The call to parallel function segmentRec had:

* work W: O(t − s)
* depth D: O(log(t − s))

On a platform with P parallel threads the running time is, for some constants b1, b2, b3, b4:
```
b1*log(t − s) + b2 + (b3(t − s) + b4)/P
```
* if `P` is bounded, we have linear behavior in `t − s`
    * possibly faster than sequential, depending on constants
* if P grows, the depth starts to dominate the cost and the running time becomes logarithmic in `t − s`

## Parallelism and Amdahl’s Law

Suppose that we have two parts of a sequential computation:

* part1 takes fraction f of the computation time (e.g. 40%)
* part2 take the remaining 1 − f fraction of time (e.g. 60%) and we can speed it up

If we make part2 P times faster the speedup is
```
1/(f + (1 − f)/P)
```
For `P = 100` and `f = 0.4` we obtain `2.46`.

Even if we speed the second part infinitely, we can obtain at most `1/0.4 = 2.5` speed up.