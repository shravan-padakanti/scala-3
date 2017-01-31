## Data-Parallelism (task-parallel vs data-parallel programming)

Previously, we learned about **task-parallel programming**: A form of parallelization that distributes **execution processes**
across computing nodes. We know how to express parallel programs with task and parallel constructs.

Next, we learn about the **data-parallel programming**: A form of parallelization that distributes **data** across computing
nodes.

### Data-Parallel Programming Model

The simplest form of data-parallel programming is the parallel `for` loop.

Example: initializing the array values.

```scala
def initializeArray(xs: Array[Int])(v: Int): Unit = {
    for (i <- (0 until xs.length).par) {
        xs(i) = v
    }
}
```

The parallel for loop is not functional – it can only affect the program through side-effects. **As long as iterations of the parallel loop write to separate memory locations, the program is correct**.

### Workload

Different data-parallel programs have different workloads.

Workload is a function that maps each input element to the amount of work required to process it:

* Uniform Workload: Defined by a constant function: `w(i) = const`
* Irregular Workload: Defined by an arbitrary function: `w(i) = f(i)`
