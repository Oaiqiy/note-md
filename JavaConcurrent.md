---
title: "Java Concurrent"
date: 2021-3-18T08:11:45Z
draft: false
tags: ["java"]
categories: ["note"]
---

## Introduction

### Thread's advantage

1. unleash the performance of processor
2. simplification of modeling
3. simplified handling of asynchronous events
4. responsive user interface

### Thread's disadvantage

1. security problems
2. activation problems
3. performance problems

## Thread Safety

keyword *synchronized* *volatile*

solve problems of multiple threads access mutable variables:

* don't share the variable in threads
* change the variable to immutable
* use sync when access variables

**Thead Safety**: When multiple threads access a class, no longer the running environment uses any scheduling methods or the threads alternately execute, and the main code needn't any other sync or synergy, the class can perform rightly.

Sateless object is thread safe.

### Atomic

Race condition: Check-Then-Act.

### Lock

Built-in lock: **synchronized**. equivalent to a mutual exclusion lock. can re-entry. 

## Sharing of Object

### Visibility

use **volatile**

don't let *this* escape when construct.

### Thread Closure

Ad-hoc thread closure.  
Stack closure.  
ThreadLocal

### Immutability

Immutable object must be thread safe.

## Composition of Object

* find all variables constructing the object status
* find the immutable condition binging status variables
* build concurrent manage strategies of object status

## Basic Building Blocks

1. Sync container class (Vector and Hashtable)
2. Concurrent container class (ConcurrentHashMap CopyOnWriteArrayList)
3. Blocking queue ans Producer and Customer
4. Blocked method and Interrupted method
5. Sync util class

## Task execution

### Execute task in thread

Serial execution, create explicitly thread for task.

Disadvantage of infinite threads

* Thread lifecycle overhead high
* resource consumption
* stability

### Executor framework

```java
    public interface Executor {
        void execute(Runnable command)
    }
```

Thread pool: use static methods in class `Executor`

* `newFixedThreadPool` create a fixed length pool
* `newCachedThreadPool` create a pool can cache
* `newSingleThreadExecutor` create a single thread executor
* `newScheduledThreadPool` schedule tasks

use `shutdown()` in `interface ExecutorService extents Executor` to shutdown an executor.

Executor has three status in lifecycle,running, closed, terminated.

### Find available concurrent

`Callback and Future`

`CompletionService` include `Executor` and `BlockingQueue`. use blocked method `take` and `poll` get result.

## Cancel and Close

Java doesn't apply any mechanisms to stop thread safely. And should avoid using `Thread.stop` ro `suspend`.

### Task cancel

cancel reasons:

* user request cancel
* operation with time limit
* application event
* error
* close

interrupt methods:

* response interrupt (wait, sleep)
* use Future implement cancel

Calling `interrupt` doesn't mean the thread stops its work immediately, rather than send a message of requesting interrupt.

Usually interrupt is the best method to implement cancel.z

uninterrupted block

1. Java.io sync Socket I/O
2. Java.io sync I/O
3. Selector async I/O
4. get some lock

package standard cancel operation to process uninterrupted block.

use `newTaskFor` to package substandard cancel

### Stop service based on thread

For the service holding threads, as long as the service's existing time is longer than the threads', the service should apply lifecycle methods.

## Thread Pool

### Implicit Coupling between Tasks and Execution Strategies

tasks need specified execution strategies

1. dependent tasks
2. tasks using thread closure
3. time sensitive tasks

Thread pool problems:

1. Deadlock
2. long-time tasks

### Set Thread Pool size

W/C = ratio of wait time to compute time

N(threads) = N(cpu) \* U(cpu) \* (1 + W / C)

`int N_CPUs = Runtime.getRuntime().availableProcessors();`

### Configure ThreadPoolExecutor

```java
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) 
```

`int corePoolSize` basic size

`BlockingQueue workQueue` has three basic kinds

1. boundless queue: boundless `LinkedBlockingQueue` in `newFixedThreadPool` and `newSingleThreadExecutor`
2. bounded queue: `ArrayBlockingQueue`, bounded `LinkedBlockingQueue`, `PriorityBlockingQueue`
3. `SynchronousQueue` only in boundless thread pool or rejected pool.

`RejectedExecutionHandler handler` when queue is filled up, saturation strategy takes effect.

* `Abort` default strategy. throw `RejectedExecutionException`
* `Discard` discard this task
* `Discard-Oldest` discard next task, and submit this task again
* `Caller-Runs`run task in this thread

`ThreadFactory threadFactory` custom threads

### Expand ThreadPoolExecutor

`BeforeExecute` `afterExecute` `terminated`

### Concurrent Recursion

DFS

## GUI Application

separate threads for GUI and task.

## Avoid Active Hazards

### Deadlock

1. lock sequence deadlock: change to the same sequence
2. dynamic lock sequence deadlock: use some methods like hash to control lock sequence.
3. deadlock between collaborating objects: open call, only protect code blocks about concurrent.

avoid deadlock:

1. use time lock
2. use Thread Dump to analyze

### Other Hazards

1. Starvation
2. Terrible Responsiveness
3. LiveLock

## Performance and Scalability

### Thread overhead

1. context switch
2. memory sync
3. block

### Reduce Lock Competition

1. reduce lock's scope
2. reduce lock's granularity
3. lock segment
4. avoid hot spot area
5. give up exclusive lock
6. monitor CPU usage
7. don't use object pool
8. reduce context switch's overhead

## Explicit Lock

### ReentrantLock

`ReentrantLock` implement `Lock` interface, and apply the same mutual exclusivity and memory visibility as `synchronized`

choose between `synchronized` and `ReentrantLock`: Choose `ReentrantLock` when need some advanced functions include timed, pollable, interruptible, fair queue, and non-block structure.

### Read-Write Lock

`ReadWriteLock`

## Custom Sync Tool

### Condition Queue

`wait()` `notify()` `notifyAll()` a queue of threads wait for some specified conditions to be true.

usually use `notifyAll()`, use `notify()` when:

1. all threads use the same **condition**
2. single in and out

### Explicit Condition Object

Like `ReentrantLock` with `Lock`, `Condition` is kind of general condition queue.

### Synchronizer

`ReentrantLock` and `Semaphore` use `AbstractQueuedSynchronized` to implement fair and unfair lock.

### AQS

TODO

## Atomic Variable and non-block sync

### Disadvantages of Lock

schedule overhead

### Hardware's support for Concurrent

#### CAS

When many threads try use CAS change a variable, only one can success.

### Atomic Variable Class

![Atomic Variable classes](https://raw.githubusercontent.com/Oaiqiy/PicGoRepo/main/20220621220614.png)

`Atomic Variable` is a kind of gooder `volatile`

### Non-Block Algorithms

Treiber Algorithms

![20220621221128](https://raw.githubusercontent.com/Oaiqiy/PicGoRepo/main/20220621221128.png)

Michael-Scott Algorithms

![20220621222010](https://raw.githubusercontent.com/Oaiqiy/PicGoRepo/main/20220621222010.png)

ABA problem: when use CAS, variable change from A to, and form B to A.

solve: use `AtomicStampedReference` or `AtomicMarkableReference`

## Java Memory Model

from *Deeper Understanding JVM*

### Happens-Before

1. Program Order Rule
2. Monitor Loc Rule
3. Volatile Variable Rule
4. Thread Start Rule
5. Thread Termination Rule
6. Thread Interruption Rule
7. Finalizer Rule
8. Transitivity