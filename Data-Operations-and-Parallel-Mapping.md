## Parallelism and collections

**Parallel processing of collections** is important. It is one of the main applications of parallelism today. We examine conditions when this can be done:

* properties of collections: ability to split, combine
* properties of operations: associativity, independence

### Functional programming and collections

Operations on collections are key to functional programming 

* **map**: apply function to each element

    ```scala
    List(1,3,8).map(x => x*x) == List(1, 9, 64)
    ```
* **fold**: combine elements with a given operation

    ```scala
    List(1,3,8).fold(100)((s,x) => s + x) == 112
    ```
* **scan**: combine folds of all list prefixes

    ```scala
    List(1,3,8).scan(100)((s,x) => s + x) == List(100, 101, 104, 112)
    ```

These operations are even more important for parallel than sequential collections: they encapsulate more complex algorithms.

## Choice of data structures

We use `List` to specify the results of operations. **Lists are not good for parallel implementations** because we cannot
efficiently:

* split them in half (need to search for the middle)
* combine them (concatenation needs linear time)

We use these alternatives for now:

* **arrays**: imperative (recall array sum)
* **trees**: can be implemented functionally

Subsequent lectures examine Scala’s parallel collection libraries

* includes many more data structures, implemented efficiently

## map: meaning and properties

`Map` applies a given function to each list element

```scala
List(1,3,8).map(x => x*x) == List(1, 9, 64)
List(a1, a2, …, an).map(f) == List(f(a1), f(a2), …, f(an))
```

Properties to keep in mind:
* `list.map(x => x) == list`
* `list.map(f.compose(g)) == list.map(g).map(f)`

Recall that `(f.compose(g))(x) = f(g(x))`

### Map as function on lists

Sequential definition:

```scala
def mapSeq[A,B](lst: List[A], f : A => B): List[B] = lst match {
    case Nil => Nil
    case h :: t => f(h) :: mapSeq(t,f)
}
```
We would like a version that parallelizes:

* computations of `f(h)` for different elements `h`
* finding the elements themselves (list is not a good choice)

### Sequential map of an array producing an array

```scala
def mapASegSeq[A,B](inp: Array[A], left: Int, right: Int, f : A => B, out: Array[B]) = {
  // Writes to out(i) for left <= i <= right-1
  var i= left
  while (i < right) {
    out(i)= f(inp(i))
    i= i+1
  } 
}
val in= Array(2,3,4,5,6)
val out= Array(0,0,0,0,0)
val f= (x:Int) => x*x
mapASegSeq(in, 1, 3, f, out)
out
res1: Array[Int] = Array(0, 9, 16, 0, 0)
```

### Parallel map of an array producing an array
```scala
def mapASegPar[A,B](inp: Array[A], left: Int, right: Int, f : A => B, out: Array[B]): Unit = {
  // Writes to out(i) for left <= i <= right-1
  if (right - left < threshold)
  mapASegSeq(inp, left, right, f, out)
  else {
    val mid = left + (right - left)/2
    parallel(mapASegPar(inp, left, mid, f, out),
    mapASegPar(inp, mid, right, f, out))
  }
}
```
**Note:**

* writes need to be disjoint (otherwise: non-deterministic behavior)
* threshold needs to be large enough (otherwise we lose efficiency)


