## Two-Phase Construction

Most data structures can be constructed in parallel using **two-phase construction**.

Previously we insisted that a combiner and the resulting collection have the **same** underlying data structure. For example, we assume that a combiner that produces an array must internally contain an array at the point when its combine method is called. 

The intermediate data structure is a data structure that:

* has an efficient `combine` method: `O(log n + log m)` or better
* has an efficient `+=` method (thus individual processors can efficiently modify the data structure)
* can be converted to the resulting data structure in `O(n/P)` time. n is the size of data structure, P is the no. of processors.

These properties of the intermediate data structure allows us to build a parallel data structure in 2 phases. In the **first** phase, different processors build intermediate data structures in parallel by invoking the `+=` method.  These intermediate data structures are then combined in a parallel reduction tree until there is a single intermediate data structure at the root. In the **second** phase, the result method uses the intermediate data structure to create the final data structure in parallel.

![two phase](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/intermediate_data_structure.png) 

#### Example: Array Combiner

Let’s implement a combiner for arrays.

Two arrays cannot be efficiently concatenated, so we will do a two-phase construction.

To keep things simple, we will limit our ArrayCombiner class to reference objects, expressed with a time bound of the type parameter T. We also add the ClassTag context bound to be able to instantiate the resulting array and the parallelism level argument. Internally, the ArrayCombiner keeps the field numElems to store the number of elements in the combiner, and the nested ArrayBuffer used to store the elements. The actual elements will be stored in these entries. We use a nested ArrayBuffer instead of a normal one for reasons that should soon become apparent.

```scala
class ArrayCombiner[T <: AnyRef: ClassTag](val parallelism: Int) {
    private var numElems = 0
    private val buffers = new ArrayBuffer[ArrayBuffer[T]]
    buffers += new ArrayBuffer[T]
}
```
First, we implement the **+=** method:

This method finds the last nested array buffer in buffers and appends the element x to it. If the last nested ArrayBuffer ever gets full, it is expanded to accommodate more elements. 

```scala
def +=(x: T) = {
    buffers.last += x
    numElems += 1
    this
}
```
Takes amortized O(1) with low constants – as efficient as an array buffer.

Next, we implement the **combine** method.

The combine method simply copies the references of the argument combiners buffers to its own buffers field. It does not need to copy the actual contents of those nested buffers, only a pointer to them.

```scala
def combine(that: ArrayCombiner[T]) = {
    buffers ++= that.buffers
    numElems += that.numElems
    this
}
```

`O(P)`, assuming that buffers contains no more than `O(P)` nested array buffers.

Finally, we implement the result method.

Once we have the root intermediate data structure, we know the required size of the array from the numElems field, so we allocate the resulting array. We then divide the array indices into chunks, pairs of starting and ending indices that each parallel task should in parallel copy. We start these tasks, wait for their completion, and then return the array. 

```scala
def result: Array[T] = {
    val array = new Array[T](numElems)
    val step = math.max(1, numElems / parallelism)
    val starts = (0 until numElems by step) :+ numElems
    val chunks = starts.zip(starts.tail)
    val tasks = for ((from, end) <- chunks) yield task {
        copyTo(array, from, end)
    }
    tasks.foreach(_.join())
    array
}
```

### Two-Phase Construction for Arrays

Two-phase construction works for in a similar way for other data structures. First, partition the elements, then construct parts of the final data structure in parallel:

1. partition the indices into subintervals
2. initialize the array in parallel

### Two-Phase Construction for Hash Tables

1. partition the hash codes into buckets
2. allocate the table, and map hash codes from different buckets into different regions

### Two-Phase Construction for Search Trees

1. partition the elements into non-overlapping intervals according to their ordering
2. construct search trees in parallel, and link non-overlapping trees

### Two-Phase Construction for Spatial Data Structures

1. spatially partition the elements
2. construct non-overlapping subsets and link them

### Implementing combiners

How can we implement combiners?

1. Two-phase construction: the combiner uses an intermediate data structure with an efficient `combine` method to partition the elements. When `result` is called, the final data structure is constructed in parallel from the intermediate data structure.
2. An efficient concatenation or union operation: a preferred way when the resulting data structure allows this.
3. Concurrent data structure – different combiners share the same
underlying data structure, and rely on _synchronization_ to correctly update the data structure when `+=` is called.