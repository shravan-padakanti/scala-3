## What is Parallel Computing?

* **Definition**: Type of computation in which many calculations are performed **simultaneously**.

* **Basic principle**: computation can be divided into smaller subproblems, each of which can be solved simultaneously.

* **Assumption**: we have parallel hardware at our disposal, which is capable of executing these computations in parallel.

## Why Parallel Computing?

Parallel programming is much harder than sequential programming.

* Separating sequential computations into parallel subcomputations can be challenging, or even impossible.
* Ensuring program correctness is more difficult, due to new types of errors.
**Speedup** is the only reason why we bother paying for this complexity.

## Parallel Programming vs. Concurrent Programming

Parallelism and concurrency are closely related concepts.

* Parallel program – uses parallel hardware to execute computation more quickly. Speedup/Efficiency is its main concern.
Eg. Bigdata, algorithmic problems, etc like matrix multiplications, computer graphics, big data processing.
* Concurrent program – _may_ or _may not_ execute multiple executions at the same time. Modularity, responsiveness, maintainability is the main concern. Eg. Asynchronous applications like Web Servers, User interfaces, Databases, etc.

## Parallelism Granularity

Parallelism manifests itself at different granularity levels.

* **bit-level parallelism** – processing multiple bits of data in parallel. (64 bit architecture, 64 bits are processed simultaneously.
* **instruction-level parallelism** – executing different instructions from the same instruction stream in parallel. Eg. In the below example, instructions b and c can be executed in parallel.

    ```
    val b = a1 + a2
    val c = a3 + a4
    val d = b + c
    ```
* **task-level parallelism** – executing separate streams of instructions in parallel. 

In this course, we focus on task-level parallelism.

## Classes of Parallel Computers

* multi-core processors: contain more than one core
* symmetric multiprocessors: contain more than 1 multi-core processors who share memory and other resources.
* general purpose graphics processing unit: contains several cores which are invoked for special tasks when requested explicitly by the host processor.
* field-programmable gate arrays: Core processor which can rewire themselves for a particular task.  
* computer clusters

Our focus will be programming for multi-cores and SMPs.

## Course structure:

* week 1 – basics of parallel computing and parallel program analysis
* week 2 – task-parallelism, basic parallel algorithms
* week 3 – data-parallelism, Scala parallel collections
* week 4 – data structures for parallel computing