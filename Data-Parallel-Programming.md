## Data-Parallelism (task-parallel vs data-parallel programming)

Previously, we learned about **task-parallel programming**: _A form of parallelization that distributes **execution processes** across computing nodes._ 

We know how to express parallel programs with task and parallel constructs.

Next, we learn about the **data-parallel programming**: _A form of parallelization that distributes **data** across computing nodes._

> Synchronous vs. Asynchronous: http://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean <br/>
> Data parallelism vs. Task parallelism: https://en.wikipedia.org/wiki/Data_parallelism#Data_parallelism_vs._task_parallelism

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
* parallel for loop using scala-parallel-collections - intermediate.
* parallel for loop using experimental data parallel scheduler – about 2× faster.

### Workload

Different data-parallel programs have different workloads.

Workload is a function that maps each input element to the amount of work required to process it:

* Uniform Workload: Defined by a constant function: `w(i) = const`. (Easy to parallelize) - left image.
* Irregular Workload: Defined by an arbitrary function: `w(i) = f(i)`  - right image.

![workload_types](https://github.com/rohitvg/scala-parallel-programming-3/blob/master/resources/images/workload_types.png)

The goal of the data parallel scheduler is to efficiently balance the workload across processors without necessarily having any knowledge about w(i). Thanks to the scheduler, the task of balancing the workload is shifted away from the programmer. This is one of the advantages of data parallel programming. 
