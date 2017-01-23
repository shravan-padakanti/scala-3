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

#### Example 1: Linear function

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
**Answer:** `W(s,t) = O(t − s)`, a function of the form: `c1(t − s) + c2`.

* t − s loop iterations
* a constant amount of work in each iteration

#### Example 2: Recursive function
```scala
def segmentRec(a: Array[Int], p: Double, s: Int, t: Int) = {
  if (t - s < threshold) {
    sumSegment(a, p, s, t)
  } else {
    val m= s + (t - s)/2
    val (sum1, sum2)= (segmentRec(a, p, s, m),
    segmentRec(a, p, m, t))
    sum1 + sum2 
  } 
}
```
**Answer**:
```
W(s,t) =  c1(t − s) + c2,             if t − s < threshold
          W(s, m) + W(m,t) + c3       otherwise, for m = ⌊(s + t)/2⌋
```