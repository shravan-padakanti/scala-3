##  Testing and Benchmarking


**Testing**:

* ensures that parts of the program are behaving according to the intended behavior. Eg. a JUnit for checking if a function to reverse lists does reverse a list.
* Typically yields a binary output – a program or its part is either correct or it is not. (We use `assert` statements in test)

**Benchmarking**:

* computes performance metrics for parts of the program. Eg running time, memory footpring, etc. For the reverse list function we could record the start and the end time and get the difference to get the running time. 
* Usually yields a continuous value, which denotes the extent to which the program is correct.

### Benchmarking Parallel Programs

Why do we benchmark parallel programs?

Performance benefits are the main reason why we are writing parallel programs in the first place. So Benchmarking parallel programs is even more important than benchmarking sequential programs.

### Performance Factors

Performance (specifically, running time) is subject to many factors:

* Processor speed
* Number of processors
* Memory access latency (amount of time the processor must wait to get data back from memory after requesting) and throughput (amount of data that can be retrieved from the memory per time unit). These 2 factors affects the degree of contention. 

> To reduce the effects of memory latency and throughput, we have L1-L2 Cache memories are used which is a small amount of memory close to the processor and it mirrors parts of the main memory. These can be accessed by the processor without going through any bus. Thus they improve performance. 

* Cache behavior. Cache improves performance, however, they make performance analysis more complicated, and although they exist to increase program performance, they can sometimes negatively impact running time with effects such as false sharing. (e.g. false sharing, associativity effects)
* Runtime behavior (e.g. garbage collection, JIT compilation, thread scheduling)

To learn more, see **What Every Programmer Should Know About Memory**, by Ulrich Drepper.

### Measurement Methodologies

Measuring performance is difficult – usually, the a performance metric is a random variable.

As a consequence of different performance vectors, it is difficult to accurately measure a metric such as the running time. Two runs of the same program usually have a similar but different running times. 

* multiple repetitions
* statistical treatment – computing mean and variance
* eliminating outliers (measurements that deviate from the rest of the group)
* ensuring steady state (warm-up): Our measurements should be taken in the state where there are no concurrently executing programs and the run time effects of dynamic compilation and memory and management are eliminated. This usually happens after the program has been running long enough, which we refer to as a warm up
* preventing anomalies (GC, JIT compilation, aggressive optimizations)

To learn more, see **Statistically Rigorous Java Performance Evaluation**, by Georges, Buytaert, and Eeckhout.

## ScalaMeter

ScalaMeter is a benchmarking and performance regression testing framework for the JVM.

* Performance regression testing – comparing performance of the current program run against known previous runs
* Benchmarking – measuring performance of the current (part of the) program

We will focus on benchmarking.

### Using ScalaMeter

First, add ScalaMeter as a dependency.
```
libraryDependencies += "com.storm-enroute" %% ”scalameter-core" % "0.6"

Then, import the contents of the ScalaMeter package, and measure:

```scala
import org.scalameter._

val time = measure {
    (0 until 1000000).toArray // returns a double (milli seconds)
}
println("Array initialization time: $time ms")
```

### JVM Warmup

We can/usualyy get different running times on two consecutive runs of the same program.
When a JVM program starts, it undergoes a period of **warmup**, after which it achieves its maximum performance.

* first, the program is interpreted
* then, parts of the program are compiled into machine code
* later, the JVM may choose to apply additional dynamic optimizations
* eventually, the program reaches steady state - here we say that it has _warmed up_.

### ScalaMeter Warmers

Usually, we want to measure steady state program performance. ScalaMeter `Warmer` objects run the benchmarked code until detecting steady state.

```scala
import org.scalameter._
val time = withWarmer(new Warmer.Default) measure {
    (0 until 1000000).toArray
}
```
when we measure running times with this, we get more consistent and close run-times. ** So we used these time, i.e. the time required by a program to exeute after warming to compare between the parallel and sequential version of a program**.

### ScalaMeter Configuration
ScalaMeter configuration clause allows specifying various parameters, such as the minimum and maximum number of warmup runs. If we are not satisfied  with the default configuration provided by ScalaMeter, then we can change them as below:

```scala
val time = config(
    Key.exec.minWarmupRuns -> 20,
    Key.exec.maxWarmupRuns -> 60,
    Key.verbose -> true
) withWarmer(new Warmer.Default) measure {
    (0 until 1000000).toArray
}
```

### ScalaMeter Measurers
Finally, ScalaMeter can measure more than just the running time.

* `Measurer.Default` – plain running time // what we saw above
* `IgnoringGC` – running time without GC pauses
* `OutlierElimination` – removes statistical outliers
* `MemoryFootprint` – memory footprint of an object
* `GarbageCollectionCycles` – total number of GC pauses
* newer ScalaMeter versions can also measure method invocation counts and boxing counts