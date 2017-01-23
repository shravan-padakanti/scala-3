## More flexible construct for parallel computation

```scala
val (v1, v2) = parallel(e1, e2)
```
we can write alternatively using a new construct called `task`:
```scala
val t1 = task(e1)
val t2 = task(e2)
val v1 = t1.join
val v2 = t2.join
```

#### The task construct
`t = task(e)` starts computation of a `call-by-name` parameter `e` "in the background"

* t is a task, which performs computation of e.
* current computation on the main thread proceeds in parallel with t
* to obtain the result of e, use `t.join`
* `t.join` blocks the parent thread and waits until the result is computed
* subsequent `t.join` calls quickly return the same result

In short, we use `task` to represent each expression/computation that we want to do in parallel.

Here is a minimal interface for tasks:
```scala
def task(c: => A) : Task[A]

trait Task[A] {
    def join: A
}
```
`task` and `join` map between computations and tasks that perform these computations. 

In terms of the value computed the equation `task(e).join == e` holds i.e. values gotten from LHS and RHS are same, LHS being faster as it happens in parallel. 

We can omit writing `.join` if we also define an implicit conversion:

```scala
implicit def getJoin[T](x:Task[T]): T = x.join
```

#### Example

We have seen four-way parallel p-norm:
```scala
val ((part1, part2),(part3,part4)) = parallel( parallel(sumSegment(a, p, 0, mid1), sumSegment(a, p, mid1, mid2)),
                                     parallel(sumSegment(a, p, mid2, mid3), sumSegment(a, p, mid3, a.length)))
power(part1 + part2 + part3 + part4, 1/p)
```
Here is essentially the same computation expressed using task:
```scala
val t1 = task {sumSegment(a, p, 0, mid1)}
val t2 = task {sumSegment(a, p, mid1, mid2)}
val t3 = task {sumSegment(a, p, mid2, mid3)}
val t4 = task {sumSegment(a, p, mid3, a.length)}
power(t1 + t2 + t3 + t4, 1/p) // join is implicit
```

### Can we define parallel using task?
We defined the [signature of `parallel` previously](https://github.com/rohitvg/scala-parallel-programming-3/wiki/Running-Computations-in-Parallel#signature-of-the-parallel-method).

Can we implement `parallel` construct as a method using `task`:
```scala
def parallel[A, B](cA: => A, cB: => B): (A, B) = {
    val tB: Task[B] = task { cB }
    val tA: A = cA
    (tA, tB.join)
}
```
Note that we need only one extra thread to the main thread, so we use the main thread for tA.

#### Wrong example of parallel using task
```scala
def parallelWrong[A, B](cA: => A, cB: => B): (A, B) = {
    val tB: B = (task { cB }).join
    val tA: A = cA
    (tA, tB.join)
}
```
Here `join` is called on `tb` where we are creating the task to be executed in parallel. So first we wait until `tb` is calculated. Then we calculated `ta`. So we are not doing these tasks in parallel.