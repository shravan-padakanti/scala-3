## Data-Parallelism (task-parallel vs data-parallel programming)

Previously, we learned about **task-parallel programming**: _A form of parallelization that distributes **execution processes** across computing nodes._ 

We know how to express parallel programs with task and parallel constructs.

Next, we learn about the **data-parallel programming**: _A form of parallelization that distributes **data** across computing nodes._

### Data-Parallel Programming Model

The simplest form of data-parallel programming is the parallel `for` loop.

#### Example 1: initializing the array values. 

The method takes in an array and an int, and writes the int to every array entry in parallel. All iterations of the loop are executed concurrently with each other.

```scala
def initializeArray(xs: Array[Int])(v: Int): Unit = {
    for (i <- (0 until xs.length).par) {    // <- notice the .par
        xs(i) = v
    }
}
```

The parallel for loop is not functional – it can only affect the program through side-effects. **As long as iterations of the parallel loop write to separate memory locations, the program is correct**. This is valid for our example.

#### Example 2: Mandelbrot Set. 

Mandelbrot Set is a set of _complex numbers_ in the plane for which the sequence: 

Z<sup>n+1</sup> = Z<sup>2</sup><sub>n</sub> + c 

does not approach infinity.

Demo Summary:
* task-parallel implementation – the slowest.
* scala-parallel-collections - intermediate.
* data-parallel implementation – about 2× faster.


### Workload

Different data-parallel programs have different workloads.

Workload is a function that maps each input element to the amount of work required to process it:

* Uniform Workload: Defined by a constant function: `w(i) = const`
* Irregular Workload: Defined by an arbitrary function: `w(i) = f(i)`


