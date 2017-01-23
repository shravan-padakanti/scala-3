## Processes

Usually an **Operating system** can run multiple processes simultaneously. 

A **process** is an instance of a program that is executing in the OS. The same program can be started as a process more than once, or even simultaneously in the same OS.

The OS multiplexes many different processes and a limited number of CPUs, so that they get **time slices of execution**. This mechanism is called **multitasking**. Thus OS exhibits parallelism. Two different processes cannot access each other’s memory directly – they are isolated. Thus sharing data between process is hard.

Each process can contain multiple independent **concurrency units called threads**. Threads can be started from within the same program, and they share the same memory address space. Each thread its own program counter and a program stack. Program counter keeps track of the position of the program on that program stack.

## JVM Threads

They **cannot** modify each other’s stack memory. They **can** only modify the heap memory.

Each JVM process starts with a **main** thread. To start additional threads:

1. Define a `Thread` subclass.
2. Instantiate the subclass. Override the `run` method.
3. Call `start` on the instance.
4. Call `join` on the instance to block execution of the main thread and get our created thread when it finishes executing and the resume the main thread.

The `Thread` subclass defines the code that the thread will execute. The same custom Thread subclass can be used to start multiple threads.

Example: 

```scala
class HelloThread extends Thread {
    override def run() {
        println("Hello")
        println("World")
    }
}

val t1 = new HelloThread
val t2 = new HelloThread
t1.start()
t2.start()
t1.join()
t2.join()
```

Output:
```scala
// first run
> Hello
> World
> Hello
> World

// second run
> Hello
> Hello
> World
> World
```

### Atomicity and Synchronized

As seen above, separate statements in two threads can overlap since they execute in parallel. In some cases, we want to ensure that a sequence of statements in a specific thread executes at once without interruption. Eg. Withdrawing money from a bank account and updating the remaining balance.

An operation is atomic if it appears as if it occurred instantaneously from the point of view of other threads.

Atomicity is achieved by using the `synchronized` block. This block must be invoked on instance of some object. Code inside this block is never executed by 2 threads at the same time. JVM ensures this by something called as a **monitor** in each object. Atmost 1 thread can own a monitor at one time.

Example: The above example can be modified so that the print operations are atomic as below:
```scala
class HelloThread extends Thread {
  private val x = new AnyRef {}
  override def run() = x.synchronized {
    println("Hello")
    println("World")
  }
}
```

Different threads use the synchronized block to agree on unique values. The synchronized block is an example of a **synchronization primitive**.

### Composition with the synchronized block

```scala
  class Account(private var amount: Int = 0) {
    def transfer(target: Account, n: Int) = this.synchronized {
      target.synchronized {
        this.amount -= n
        target.amount += n
      }
    }
  }
```

### Deadlocks

Deadlock is a scenario in which two or more threads compete for resources (such as monitor ownership), and wait for each to finish without releasing the already acquired resources.

```scala
val a = new Account(50)
val b = new Account(70)

// thread T1
a.transfer(b, 10)
// thread T2
b.transfer(a, 10)
```

#### Resolving Deadlocks: 

One approach is to always acquire resources in the same order. This assumes an ordering relationship on the resources.

```scala
val uid = getUniqueUid()
private def lockAndTransfer(target: Account, n: Int) = this.synchronized {
  target.synchronized {
    this.amount -= n
    target.amount += n
  }
}
def transfer(target: Account, n: Int) = {
  if (this.uid < target.uid) this.lockAndTransfer(target, n)
  else target.lockAndTransfer(this, -n)
}
```

## Java Memory model

A **Memory model** is a set of rules that describes how threads interact when accessing shared memory. The memory model for the JVM is the **Java Memory Model**.

1. Two threads writing to separate locations in memory do not need synchronization.
2. A thread X that calls join on another thread Y is guaranteed to observe all the writes by thread Y after join returns.