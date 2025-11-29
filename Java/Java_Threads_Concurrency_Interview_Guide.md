# Java Threads & Concurrency - Interview Preparation Guide
## Java 8 through Java 17 Features

---

## Table of Contents

1. [Thread Fundamentals](#1-thread-fundamentals)
2. [Thread Lifecycle and States](#2-thread-lifecycle-and-states)
3. [Synchronization and Locks](#3-synchronization-and-locks)
4. [Java Memory Model](#4-java-memory-model)
5. [Executor Framework](#5-executor-framework)
6. [CompletableFuture (Java 8+)](#6-completablefuture-java-8)
7. [Concurrent Collections](#7-concurrent-collections)
8. [Atomic Classes](#8-atomic-classes)
9. [Locks and Conditions](#9-locks-and-conditions)
10. [Fork/Join Framework](#10-forkjoin-framework)
11. [Java 9-17 Concurrency Enhancements](#11-java-9-17-concurrency-enhancements)
12. [Virtual Threads (Java 19+ Preview)](#12-virtual-threads-java-19-preview)
13. [Common Concurrency Patterns](#13-common-concurrency-patterns)
14. [Common Interview Questions](#14-common-interview-questions)
15. [Best Practices and Pitfalls](#15-best-practices-and-pitfalls)

---

## 1. Thread Fundamentals

### 1.1 Creating Threads

**Method 1: Extending Thread Class**
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

MyThread thread = new MyThread();
thread.start(); // Not run()!
```

**Method 2: Implementing Runnable Interface (Preferred)**
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}

Thread thread = new Thread(new MyRunnable());
thread.start();

// Lambda expression (Java 8+)
Thread thread2 = new Thread(() -> System.out.println("Lambda thread"));
thread2.start();
```

**Method 3: Implementing Callable Interface (Returns Result)**
```java
Callable<Integer> callable = () -> {
    TimeUnit.SECONDS.sleep(1);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(callable);
Integer result = future.get(); // Blocks until result available
executor.shutdown();
```

### 1.2 Thread vs Runnable vs Callable

| Aspect | Thread | Runnable | Callable |
|--------|--------|----------|----------|
| Type | Class | Interface | Interface |
| Method | `run()` | `run()` | `call()` |
| Return | void | void | Generic V |
| Exception | No checked | No checked | Can throw |
| Inheritance | Single only | Multiple interfaces | Multiple interfaces |
| Use with Executor | No | Yes | Yes |

### 1.3 Thread Properties and Methods

```java
Thread thread = new Thread(() -> {
    // Thread work
});

// Thread properties
thread.setName("WorkerThread");
thread.setPriority(Thread.MAX_PRIORITY); // 1-10, default 5
thread.setDaemon(true); // Must set before start()

// Static methods
Thread.currentThread();           // Current executing thread
Thread.sleep(1000);              // Sleep in milliseconds
Thread.yield();                  // Hint to scheduler
Thread.interrupted();            // Check and clear interrupt flag

// Instance methods
thread.start();                  // Start execution
thread.join();                   // Wait for completion
thread.join(1000);              // Wait with timeout
thread.interrupt();              // Set interrupt flag
thread.isAlive();               // Check if running
thread.isInterrupted();         // Check interrupt flag (doesn't clear)
thread.getState();              // Get thread state
```

### 1.4 Daemon vs User Threads

| Aspect | User Thread | Daemon Thread |
|--------|-------------|---------------|
| JVM Shutdown | JVM waits | JVM doesn't wait |
| Purpose | Main application tasks | Background services |
| Example | Main thread, worker threads | GC, finalizer thread |
| Creation | Default | `setDaemon(true)` before `start()` |

```java
Thread daemon = new Thread(() -> {
    while (true) {
        System.out.println("Daemon running...");
        try { Thread.sleep(1000); } catch (InterruptedException e) { break; }
    }
});
daemon.setDaemon(true);
daemon.start();
// JVM will exit when main thread completes, daemon stops abruptly
```

---

## 2. Thread Lifecycle and States

### 2.1 Thread States

```
        ┌─────────────────────────────────────────────────────────────┐
        │                                                             │
        ▼                                                             │
    ┌───────┐     start()     ┌──────────┐                           │
    │  NEW  │ ───────────────►│ RUNNABLE │◄──────────────────────────┤
    └───────┘                 └────┬─────┘                           │
                                   │                                  │
                    ┌──────────────┼──────────────┐                  │
                    │              │              │                   │
                    ▼              ▼              ▼                   │
            ┌───────────┐   ┌───────────┐  ┌──────────┐             │
            │  BLOCKED  │   │  WAITING  │  │  TIMED   │             │
            │           │   │           │  │ WAITING  │             │
            └─────┬─────┘   └─────┬─────┘  └────┬─────┘             │
                  │               │              │                   │
                  └───────────────┴──────────────┘                   │
                                  │                                  │
                         notify/notifyAll                            │
                         lock acquired                               │
                         timeout expired                             │
                                  │                                  │
                                  └──────────────────────────────────┘
                                                                     
                              run() completes
                                    │
                                    ▼
                             ┌────────────┐
                             │ TERMINATED │
                             └────────────┘
```

### 2.2 State Descriptions

| State | Description | Caused By |
|-------|-------------|-----------|
| NEW | Created but not started | `new Thread()` |
| RUNNABLE | Ready to run or running | `start()` |
| BLOCKED | Waiting for monitor lock | `synchronized` block entry |
| WAITING | Indefinite wait | `wait()`, `join()`, `park()` |
| TIMED_WAITING | Timed wait | `sleep()`, `wait(timeout)`, `join(timeout)` |
| TERMINATED | Completed execution | `run()` finished or exception |

```java
Thread thread = new Thread(() -> {
    try {
        synchronized (lock) {
            lock.wait(); // WAITING
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

System.out.println(thread.getState()); // NEW
thread.start();
Thread.sleep(100);
System.out.println(thread.getState()); // WAITING
```

### 2.3 Thread Interruption

```java
Thread worker = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            // Do work
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            // InterruptedException clears interrupt flag
            // Re-interrupt to preserve status
            Thread.currentThread().interrupt();
            break;
        }
    }
    System.out.println("Worker stopped gracefully");
});

worker.start();
Thread.sleep(3000);
worker.interrupt(); // Request interruption
```

**Interrupt Handling Best Practices:**
1. Check `isInterrupted()` in loops
2. Catch `InterruptedException` and either re-interrupt or exit
3. Never swallow `InterruptedException` silently
4. Use interrupt for cooperative cancellation

---

## 3. Synchronization and Locks

### 3.1 synchronized Keyword

**Synchronized Method:**
```java
public class Counter {
    private int count = 0;
    
    // Instance method - locks on 'this'
    public synchronized void increment() {
        count++;
    }
    
    // Static method - locks on Class object
    public static synchronized void staticMethod() {
        // ...
    }
}
```

**Synchronized Block:**
```java
public class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized (lock) { // Explicit lock object
            count++;
        }
    }
    
    public void decrement() {
        synchronized (this) { // Lock on 'this'
            count--;
        }
    }
}
```

### 3.2 Monitor Concept

Every Java object has an intrinsic lock (monitor):

```
┌─────────────────────────────────────┐
│           Object Monitor            │
├─────────────────────────────────────┤
│  ┌─────────────────────────────┐   │
│  │      Entry Set (BLOCKED)    │   │
│  │  Thread1, Thread2, Thread3  │   │
│  └──────────────┬──────────────┘   │
│                 │ acquire lock     │
│                 ▼                   │
│  ┌─────────────────────────────┐   │
│  │         Owner               │   │
│  │      (RUNNABLE)             │   │
│  │      ThreadX                │   │
│  └──────────────┬──────────────┘   │
│                 │ wait()           │
│                 ▼                   │
│  ┌─────────────────────────────┐   │
│  │      Wait Set (WAITING)     │   │
│  │  Thread4, Thread5           │   │
│  └─────────────────────────────┘   │
│         ▲                          │
│         │ notify()/notifyAll()     │
│         └──────────────────────────│
└─────────────────────────────────────┘
```

### 3.3 wait(), notify(), notifyAll()

```java
public class ProducerConsumer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity = 10;
    
    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait(); // Release lock and wait
        }
        queue.add(item);
        notifyAll(); // Wake up all waiting threads
    }
    
    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        int item = queue.poll();
        notifyAll();
        return item;
    }
}
```

**Critical Rules:**
1. Must hold monitor lock to call wait/notify
2. Always call wait() in a loop (spurious wakeups)
3. Prefer notifyAll() over notify() (avoid missed signals)
4. wait() releases the lock, notify() does not

### 3.4 Deadlock

**Deadlock Conditions (All 4 required):**
1. Mutual Exclusion
2. Hold and Wait
3. No Preemption
4. Circular Wait

```java
// Deadlock Example
Object lock1 = new Object();
Object lock2 = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lock1) {
        Thread.sleep(100);
        synchronized (lock2) { /* ... */ }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lock2) {
        Thread.sleep(100);
        synchronized (lock1) { /* ... */ }
    }
});

// Prevention: Always acquire locks in same order
Thread t1Fixed = new Thread(() -> {
    synchronized (lock1) {
        synchronized (lock2) { /* ... */ }
    }
});

Thread t2Fixed = new Thread(() -> {
    synchronized (lock1) { // Same order as t1
        synchronized (lock2) { /* ... */ }
    }
});
```

### 3.5 Livelock and Starvation

**Livelock:** Threads are active but make no progress (keep responding to each other)

```java
// Livelock Example
while (resourceNeeded) {
    if (resourceAvailable) {
        useResource();
    } else {
        releaseMyResource(); // Both threads keep releasing
    }
}
```

**Starvation:** Thread never gets CPU time due to priority or lock contention

---

## 4. Java Memory Model

### 4.1 JMM Overview

```
┌─────────────────────────────────────────────────────┐
│                    Main Memory                       │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐       │
│  │  Var A    │  │  Var B    │  │  Var C    │       │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘       │
└────────┼──────────────┼──────────────┼──────────────┘
         │              │              │
    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
    │         │    │         │    │         │
┌───▼───┐ ┌───▼───┐ ┌───▼───┐ ┌───▼───┐ ┌───▼───┐
│Thread1│ │Thread1│ │Thread2│ │Thread2│ │Thread3│
│Local  │ │Cache  │ │Local  │ │Cache  │ │Local  │
│Memory │ │       │ │Memory │ │       │ │Memory │
└───────┘ └───────┘ └───────┘ └───────┘ └───────┘
```

### 4.2 Visibility Problem

```java
// Without volatile - may never terminate!
class VisibilityProblem {
    private boolean running = true;
    
    public void stop() {
        running = false; // May not be visible to other thread
    }
    
    public void run() {
        while (running) { // May read cached value forever
            // do work
        }
    }
}

// With volatile - guaranteed visibility
class VisibilityFixed {
    private volatile boolean running = true;
    
    public void stop() {
        running = false; // Visible immediately
    }
    
    public void run() {
        while (running) { // Always reads from main memory
            // do work
        }
    }
}
```

### 4.3 volatile Keyword

**Guarantees:**
1. Visibility: Reads always see most recent write
2. Ordering: Prevents reordering around volatile access
3. Atomicity: Only for single read/write (NOT for compound operations)

```java
// volatile use cases
private volatile boolean flag;      // ✓ Simple flag
private volatile int counter;       // ✗ counter++ is not atomic!
private volatile Reference ref;     // ✓ Reference assignment

// volatile does NOT help with:
volatile int count = 0;
count++;  // NOT atomic! (read-modify-write)

// Use AtomicInteger instead:
AtomicInteger atomicCount = new AtomicInteger(0);
atomicCount.incrementAndGet();  // Atomic
```

### 4.4 Happens-Before Relationship

| Action | Happens-Before |
|--------|----------------|
| Program order | Earlier action HB later action (same thread) |
| Monitor lock | Unlock HB subsequent lock (same monitor) |
| volatile | Write HB subsequent read (same variable) |
| Thread start | `start()` HB any action in started thread |
| Thread termination | Any action in thread HB `join()` return |
| Interruption | `interrupt()` HB detection of interrupt |

```java
// Happens-before example
class HappensBefore {
    private int x = 0;
    private volatile boolean ready = false;
    
    // Thread 1
    public void writer() {
        x = 42;           // (1)
        ready = true;     // (2) volatile write
    }
    
    // Thread 2
    public void reader() {
        if (ready) {      // (3) volatile read
            int y = x;    // (4) guaranteed to see 42
        }
    }
    // (1) HB (2) - program order
    // (2) HB (3) - volatile
    // (3) HB (4) - program order
    // Therefore (1) HB (4) - transitivity
}
```

### 4.5 Double-Checked Locking

```java
// Broken without volatile (pre-Java 5)
class Singleton {
    private static Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton(); // May be reordered!
                }
            }
        }
        return instance;
    }
}

// Correct with volatile (Java 5+)
class SingletonFixed {
    private static volatile Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Better: Initialization-on-demand holder idiom
class SingletonBest {
    private SingletonBest() {}
    
    private static class Holder {
        static final SingletonBest INSTANCE = new SingletonBest();
    }
    
    public static SingletonBest getInstance() {
        return Holder.INSTANCE;
    }
}
```

---

## 5. Executor Framework

### 5.1 Executor Hierarchy

```
┌───────────────────────────────────────────────────────────┐
│                       Executor                             │
│                    execute(Runnable)                       │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│                    ExecutorService                         │
│  submit(), invokeAll(), invokeAny(), shutdown()           │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│               ScheduledExecutorService                     │
│  schedule(), scheduleAtFixedRate(), scheduleWithFixedDelay│
└───────────────────────────────────────────────────────────┘
```

### 5.2 ThreadPoolExecutor

```java
// Core components
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    corePoolSize,      // Minimum threads kept alive
    maximumPoolSize,   // Maximum threads allowed
    keepAliveTime,     // Idle time before non-core thread terminates
    TimeUnit.SECONDS,
    workQueue,         // Queue for tasks
    threadFactory,     // Creates new threads
    rejectionHandler   // Policy when queue is full
);
```

**Thread Pool Behavior:**

```
New Task Arrives
       │
       ▼
┌──────────────────┐
│ threads < core?  │───Yes──► Create new thread
└────────┬─────────┘
         │No
         ▼
┌──────────────────┐
│  queue not full? │───Yes──► Add to queue
└────────┬─────────┘
         │No
         ▼
┌──────────────────┐
│ threads < max?   │───Yes──► Create new thread
└────────┬─────────┘
         │No
         ▼
    Reject Task
```

### 5.3 Executors Factory Methods

```java
// Fixed thread pool
ExecutorService fixed = Executors.newFixedThreadPool(10);
// corePoolSize = maxPoolSize = 10
// LinkedBlockingQueue (unbounded)

// Cached thread pool
ExecutorService cached = Executors.newCachedThreadPool();
// corePoolSize = 0, maxPoolSize = Integer.MAX_VALUE
// SynchronousQueue (no buffering)
// Good for short-lived tasks

// Single thread executor
ExecutorService single = Executors.newSingleThreadExecutor();
// corePoolSize = maxPoolSize = 1
// LinkedBlockingQueue (unbounded)
// Tasks execute sequentially

// Scheduled thread pool
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(5);
// For delayed/periodic tasks

// Work stealing pool (Java 8+)
ExecutorService workStealing = Executors.newWorkStealingPool();
// Uses ForkJoinPool
// Parallelism = available processors
```

### 5.4 Rejection Policies

| Policy | Behavior |
|--------|----------|
| AbortPolicy (default) | Throws `RejectedExecutionException` |
| CallerRunsPolicy | Caller thread executes the task |
| DiscardPolicy | Silently discards task |
| DiscardOldestPolicy | Discards oldest queued task |

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2, 4, 60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

### 5.5 ExecutorService Lifecycle

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submit tasks
Future<String> future = executor.submit(() -> "result");
executor.execute(() -> System.out.println("Task"));

// Graceful shutdown
executor.shutdown(); // No new tasks, complete existing
boolean terminated = executor.awaitTermination(60, TimeUnit.SECONDS);

// Forceful shutdown
List<Runnable> pending = executor.shutdownNow(); // Attempt to stop all

// Check state
executor.isShutdown();    // shutdown() called
executor.isTerminated();  // All tasks completed
```

### 5.6 ScheduledExecutorService

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// One-time delayed execution
ScheduledFuture<?> future = scheduler.schedule(
    () -> System.out.println("Delayed"),
    5, TimeUnit.SECONDS
);

// Fixed rate: next execution starts at fixed intervals
// If task takes longer than period, next execution starts immediately
scheduler.scheduleAtFixedRate(
    () -> System.out.println("Fixed rate"),
    0,    // initial delay
    1,    // period
    TimeUnit.SECONDS
);

// Fixed delay: next execution starts after fixed delay from previous completion
scheduler.scheduleWithFixedDelay(
    () -> System.out.println("Fixed delay"),
    0,    // initial delay
    1,    // delay after completion
    TimeUnit.SECONDS
);
```

**scheduleAtFixedRate vs scheduleWithFixedDelay:**

```
scheduleAtFixedRate (period = 5s, task takes 2s):
|--Task--|     |--Task--|     |--Task--|
0        2     5        7     10       12

scheduleAtFixedRate (period = 5s, task takes 7s):
|----Task----|----Task----|----Task----|
0            7           14           21
(No gap, next starts immediately)

scheduleWithFixedDelay (delay = 5s, task takes 2s):
|--Task--|          |--Task--|          |--Task--|
0        2          7        9          14       16
         <---5s---->         <---5s---->
```

---

## 6. CompletableFuture (Java 8+)

### 6.1 Creating CompletableFuture

```java
// Run async without result
CompletableFuture<Void> cf1 = CompletableFuture.runAsync(() -> {
    System.out.println("Running async");
});

// Run async with result
CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> {
    return "Result";
});

// With custom executor
ExecutorService executor = Executors.newFixedThreadPool(10);
CompletableFuture<String> cf3 = CompletableFuture.supplyAsync(
    () -> "Result",
    executor
);

// Already completed
CompletableFuture<String> completed = CompletableFuture.completedFuture("Done");

// Completed exceptionally
CompletableFuture<String> failed = CompletableFuture.failedFuture(
    new RuntimeException("Error")
);
```

### 6.2 Transformation Methods

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello");

// thenApply - transform result (sync)
CompletableFuture<Integer> length = future.thenApply(String::length);

// thenApplyAsync - transform result (async)
CompletableFuture<Integer> lengthAsync = future.thenApplyAsync(String::length);

// thenAccept - consume result (sync)
CompletableFuture<Void> consumed = future.thenAccept(System.out::println);

// thenRun - run after completion (no access to result)
CompletableFuture<Void> after = future.thenRun(() -> System.out.println("Done"));

// thenCompose - flatMap (returns CompletableFuture)
CompletableFuture<String> composed = future.thenCompose(s -> 
    CompletableFuture.supplyAsync(() -> s + " World")
);
```

### 6.3 Combining Multiple Futures

```java
CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> "World");

// thenCombine - combine two futures
CompletableFuture<String> combined = cf1.thenCombine(cf2, 
    (s1, s2) -> s1 + " " + s2
);

// thenAcceptBoth - consume both results
cf1.thenAcceptBoth(cf2, (s1, s2) -> System.out.println(s1 + " " + s2));

// runAfterBoth - run after both complete
cf1.runAfterBoth(cf2, () -> System.out.println("Both done"));

// applyToEither - first to complete
cf1.applyToEither(cf2, String::toUpperCase);

// acceptEither - consume first to complete
cf1.acceptEither(cf2, System.out::println);

// runAfterEither - run after first completes
cf1.runAfterEither(cf2, () -> System.out.println("One done"));
```

### 6.4 Waiting for Multiple Futures

```java
List<CompletableFuture<String>> futures = List.of(
    CompletableFuture.supplyAsync(() -> "A"),
    CompletableFuture.supplyAsync(() -> "B"),
    CompletableFuture.supplyAsync(() -> "C")
);

// allOf - wait for all (returns Void)
CompletableFuture<Void> allDone = CompletableFuture.allOf(
    futures.toArray(new CompletableFuture[0])
);

// Get all results after allOf
CompletableFuture<List<String>> allResults = allDone.thenApply(v ->
    futures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList())
);

// anyOf - wait for first (returns Object)
CompletableFuture<Object> anyDone = CompletableFuture.anyOf(
    futures.toArray(new CompletableFuture[0])
);
```

### 6.5 Exception Handling

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) throw new RuntimeException("Error");
    return "Success";
});

// exceptionally - handle exception, return fallback
CompletableFuture<String> recovered = future.exceptionally(ex -> "Fallback");

// handle - handle both success and failure
CompletableFuture<String> handled = future.handle((result, ex) -> {
    if (ex != null) return "Error: " + ex.getMessage();
    return result;
});

// whenComplete - peek at result/exception (doesn't transform)
CompletableFuture<String> peeked = future.whenComplete((result, ex) -> {
    if (ex != null) System.err.println("Failed: " + ex);
    else System.out.println("Success: " + result);
});

// Java 12+ exceptionallyCompose
CompletableFuture<String> retried = future.exceptionallyCompose(ex ->
    CompletableFuture.supplyAsync(() -> "Retry result")
);
```

### 6.6 Timeout Handling (Java 9+)

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try { Thread.sleep(5000); } catch (InterruptedException e) {}
    return "Result";
});

// orTimeout - completes exceptionally on timeout
future.orTimeout(1, TimeUnit.SECONDS);

// completeOnTimeout - completes with default value on timeout
future.completeOnTimeout("Default", 1, TimeUnit.SECONDS);
```

### 6.7 CompletableFuture Best Practices

```java
// 1. Always specify executor for CPU-intensive tasks
ExecutorService cpuExecutor = Executors.newFixedThreadPool(
    Runtime.getRuntime().availableProcessors()
);
CompletableFuture.supplyAsync(() -> cpuIntensiveTask(), cpuExecutor);

// 2. Don't block in async pipeline
// Bad
future.thenApply(s -> {
    return blockingCall(); // Blocks common pool thread!
});

// Good
future.thenCompose(s -> 
    CompletableFuture.supplyAsync(() -> blockingCall(), ioExecutor)
);

// 3. Handle exceptions at each stage if needed
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(this::fetchData)
    .exceptionally(ex -> "default")
    .thenApply(this::process)
    .exceptionally(ex -> "processed default");

// 4. Use join() in streams (not get())
List<String> results = futures.stream()
    .map(CompletableFuture::join) // Unchecked exception
    .collect(Collectors.toList());
```

---

## 7. Concurrent Collections

### 7.1 ConcurrentHashMap

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Basic operations (atomic)
map.put("a", 1);
map.get("a");
map.remove("a");
map.putIfAbsent("b", 2);
map.replace("b", 2, 3);
map.remove("b", 3);

// Compute operations (atomic)
map.compute("counter", (k, v) -> v == null ? 1 : v + 1);
map.computeIfAbsent("lazy", k -> expensiveComputation());
map.computeIfPresent("exists", (k, v) -> v + 1);
map.merge("counter", 1, Integer::sum);

// Bulk operations (Java 8+) with parallelism threshold
// If size > threshold, operations run in parallel
long threshold = 1; // Always parallel

map.forEach(threshold, (k, v) -> System.out.println(k + "=" + v));

long sum = map.reduceValues(threshold, Integer::sum);

String found = map.search(threshold, (k, v) -> v > 100 ? k : null);

// Size operations
map.size();          // May be approximate under contention
map.mappingCount();  // Long version, more accurate
```

### 7.2 ConcurrentHashMap Internals (Java 8+)

```
┌─────────────────────────────────────────────────────────┐
│                    Node[] table                          │
├─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬────────┤
│  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  ...   │
└──┬──┴──┬──┴──┬──┴─────┴─────┴─────┴─────┴─────┴────────┘
   │     │     │
   ▼     ▼     ▼
 null  Node  Node──►Node──►Node  (Linked list if < 8 nodes)
              │
              ▼
           TreeNode──►TreeNode  (Red-Black tree if >= 8 nodes)

Locking: CAS for reads, synchronized on first node for writes
```

### 7.3 CopyOnWriteArrayList

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

// Write operations create a new copy
list.add("a");
list.set(0, "b");
list.remove(0);

// Iteration is snapshot-based - never throws ConcurrentModificationException
for (String item : list) {
    list.add("new"); // Doesn't affect this iteration
}

// Best for: read-heavy, write-rare scenarios
// Listeners, observers, configuration
List<EventListener> listeners = new CopyOnWriteArrayList<>();
```

### 7.4 BlockingQueue Implementations

| Implementation | Bounded | FIFO | Notes |
|---------------|---------|------|-------|
| ArrayBlockingQueue | Yes | Yes | Fixed capacity array |
| LinkedBlockingQueue | Optional | Yes | Optionally bounded linked list |
| PriorityBlockingQueue | No | No | Priority heap |
| SynchronousQueue | No | - | Direct handoff, no capacity |
| DelayQueue | No | No | Elements available after delay |
| LinkedTransferQueue | No | Yes | Transfer to waiting consumer |

```java
// Producer-Consumer with BlockingQueue
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);

// Producer
Thread producer = new Thread(() -> {
    while (true) {
        Task task = createTask();
        queue.put(task); // Blocks if full
    }
});

// Consumer
Thread consumer = new Thread(() -> {
    while (true) {
        Task task = queue.take(); // Blocks if empty
        process(task);
    }
});

// With timeout
boolean added = queue.offer(task, 1, TimeUnit.SECONDS);
Task polled = queue.poll(1, TimeUnit.SECONDS);
```

### 7.5 ConcurrentSkipListMap/Set

```java
// Thread-safe sorted map (like TreeMap but concurrent)
ConcurrentSkipListMap<String, Integer> map = new ConcurrentSkipListMap<>();

// NavigableMap operations
map.firstKey();
map.lastKey();
map.lowerKey("c");
map.floorKey("c");
map.ceilingKey("c");
map.higherKey("c");
map.subMap("a", "d");

// Thread-safe sorted set
ConcurrentSkipListSet<String> set = new ConcurrentSkipListSet<>();
```

### 7.6 Comparison

| Collection | Thread-Safe | Null | Ordered | Performance |
|------------|-------------|------|---------|-------------|
| HashMap | No | Yes | No | O(1) |
| Hashtable | Yes (sync) | No | No | O(1), slow |
| ConcurrentHashMap | Yes | No | No | O(1), fast |
| TreeMap | No | No | Sorted | O(log n) |
| ConcurrentSkipListMap | Yes | No | Sorted | O(log n) |
| ArrayList | No | Yes | Insertion | O(1) random |
| CopyOnWriteArrayList | Yes | Yes | Insertion | O(n) write |

---

## 8. Atomic Classes

### 8.1 Atomic Class Hierarchy

```
java.util.concurrent.atomic
├── AtomicBoolean
├── AtomicInteger
├── AtomicLong
├── AtomicReference<V>
├── AtomicIntegerArray
├── AtomicLongArray
├── AtomicReferenceArray<E>
├── AtomicMarkableReference<V>
├── AtomicStampedReference<V>
├── AtomicIntegerFieldUpdater<T>
├── AtomicLongFieldUpdater<T>
├── AtomicReferenceFieldUpdater<T,V>
├── LongAdder (Java 8+)
├── LongAccumulator (Java 8+)
├── DoubleAdder (Java 8+)
└── DoubleAccumulator (Java 8+)
```

### 8.2 AtomicInteger Operations

```java
AtomicInteger counter = new AtomicInteger(0);

// Basic operations
counter.get();                    // Read
counter.set(10);                  // Write
counter.lazySet(10);             // Eventual write (no memory barrier)

// Atomic increment/decrement
counter.incrementAndGet();        // ++x
counter.getAndIncrement();        // x++
counter.decrementAndGet();        // --x
counter.getAndDecrement();        // x--
counter.addAndGet(5);            // x += 5, returns new
counter.getAndAdd(5);            // x += 5, returns old

// Compare-and-swap
boolean success = counter.compareAndSet(10, 20); // If 10, set to 20

// Update with function
counter.updateAndGet(x -> x * 2);        // Atomic multiply
counter.getAndUpdate(x -> x * 2);        // Returns old value
counter.accumulateAndGet(5, Integer::sum); // Atomic accumulate

// Weak compare-and-set (may fail spuriously, faster on some platforms)
counter.weakCompareAndSet(10, 20);
```

### 8.3 AtomicReference

```java
AtomicReference<User> userRef = new AtomicReference<>(new User("Alice"));

// CAS for objects
User oldUser = userRef.get();
User newUser = new User("Bob");
boolean updated = userRef.compareAndSet(oldUser, newUser);

// Update with function
userRef.updateAndGet(user -> new User(user.getName().toUpperCase()));

// Atomically set and get old value
User previous = userRef.getAndSet(new User("Charlie"));
```

### 8.4 AtomicStampedReference (ABA Problem Solution)

```java
// ABA Problem: Value changes A → B → A, CAS succeeds but misses changes
// AtomicStampedReference tracks a version stamp

AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

// Get current value and stamp
int[] stampHolder = new int[1];
String value = ref.get(stampHolder);
int stamp = stampHolder[0];

// CAS with stamp check
boolean success = ref.compareAndSet("A", "B", stamp, stamp + 1);

// This fails even if value is back to "A" (stamp mismatch)
ref.set("A", 2); // stamp is now 2
ref.compareAndSet("A", "C", 0, 1); // fails: stamp 0 != current stamp 2
```

### 8.5 LongAdder and LongAccumulator (Java 8+)

```java
// LongAdder - faster than AtomicLong for high contention
LongAdder adder = new LongAdder();
adder.add(1);
adder.increment();
adder.decrement();
long sum = adder.sum();      // Get current total
adder.reset();               // Reset to 0
long sumThenReset = adder.sumThenReset();

// LongAccumulator - custom accumulation function
LongAccumulator accumulator = new LongAccumulator(Long::max, Long.MIN_VALUE);
accumulator.accumulate(10);
accumulator.accumulate(5);
accumulator.accumulate(15);
long max = accumulator.get(); // 15
```

**When to use LongAdder vs AtomicLong:**

| Scenario | Use |
|----------|-----|
| Low contention | AtomicLong |
| High contention (many threads incrementing) | LongAdder |
| Need exact current value frequently | AtomicLong |
| Sum needed only at end | LongAdder |
| Fine-grained sequence numbers | AtomicLong |
| Statistics counters | LongAdder |

### 8.6 Field Updaters

```java
// Update volatile fields atomically without AtomicInteger overhead
class Counter {
    volatile int count;
    
    private static final AtomicIntegerFieldUpdater<Counter> UPDATER =
        AtomicIntegerFieldUpdater.newUpdater(Counter.class, "count");
    
    public void increment() {
        UPDATER.incrementAndGet(this);
    }
}

// Requirements:
// - Field must be volatile
// - Field must be accessible (not private in different class)
// - Cannot be static
// - Cannot be final
```

---

## 9. Locks and Conditions

### 9.1 Lock Interface

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

### 9.2 ReentrantLock

```java
ReentrantLock lock = new ReentrantLock();

// Basic usage
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock(); // Always in finally!
}

// Try lock with timeout
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
} else {
    // Handle timeout
}

// Interruptible lock
try {
    lock.lockInterruptibly();
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}

// Lock info
lock.isLocked();
lock.isHeldByCurrentThread();
lock.getHoldCount();
lock.getQueueLength();
```

### 9.3 Fair vs Unfair Lock

```java
// Unfair lock (default) - better throughput
ReentrantLock unfairLock = new ReentrantLock(); // or new ReentrantLock(false)

// Fair lock - FIFO ordering, prevents starvation
ReentrantLock fairLock = new ReentrantLock(true);
```

| Aspect | Fair Lock | Unfair Lock |
|--------|-----------|-------------|
| Order | FIFO | No guarantee |
| Throughput | Lower | Higher |
| Starvation | Prevented | Possible |
| Overhead | Higher | Lower |

### 9.4 Condition

```java
ReentrantLock lock = new ReentrantLock();
Condition notEmpty = lock.newCondition();
Condition notFull = lock.newCondition();
Queue<Integer> queue = new LinkedList<>();
int capacity = 10;

// Producer
public void produce(int item) throws InterruptedException {
    lock.lock();
    try {
        while (queue.size() == capacity) {
            notFull.await(); // Wait until not full
        }
        queue.add(item);
        notEmpty.signal(); // Signal consumer
    } finally {
        lock.unlock();
    }
}

// Consumer
public int consume() throws InterruptedException {
    lock.lock();
    try {
        while (queue.isEmpty()) {
            notEmpty.await(); // Wait until not empty
        }
        int item = queue.poll();
        notFull.signal(); // Signal producer
        return item;
    } finally {
        lock.unlock();
    }
}
```

### 9.5 ReadWriteLock

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// Multiple readers can read concurrently
public String read() {
    readLock.lock();
    try {
        return data;
    } finally {
        readLock.unlock();
    }
}

// Writers have exclusive access
public void write(String newData) {
    writeLock.lock();
    try {
        data = newData;
    } finally {
        writeLock.unlock();
    }
}
```

**Read-Write Lock Semantics:**
- Multiple threads can hold read lock simultaneously
- Write lock is exclusive (blocks both reads and writes)
- Read lock can be upgraded to write lock (in same thread)
- Write lock can be downgraded to read lock

### 9.6 StampedLock (Java 8+)

```java
StampedLock lock = new StampedLock();

// Writing (exclusive)
long stamp = lock.writeLock();
try {
    // Write operation
} finally {
    lock.unlockWrite(stamp);
}

// Reading (shared)
long stamp = lock.readLock();
try {
    // Read operation
} finally {
    lock.unlockRead(stamp);
}

// Optimistic reading (best performance for read-heavy)
public double readOptimistically() {
    long stamp = lock.tryOptimisticRead(); // Non-blocking!
    double currentX = x, currentY = y;
    if (!lock.validate(stamp)) { // Check if write occurred
        stamp = lock.readLock(); // Fallback to read lock
        try {
            currentX = x;
            currentY = y;
        } finally {
            lock.unlockRead(stamp);
        }
    }
    return Math.sqrt(currentX * currentX + currentY * currentY);
}
```

**Lock Comparison:**

| Feature | synchronized | ReentrantLock | ReadWriteLock | StampedLock |
|---------|-------------|---------------|---------------|-------------|
| Fairness | No | Optional | Optional | No |
| Interruptible | No | Yes | Yes | Yes |
| Try lock | No | Yes | Yes | Yes |
| Multiple conditions | No | Yes | Yes | No |
| Read/write separation | No | No | Yes | Yes |
| Optimistic read | No | No | No | Yes |
| Reentrant | Yes | Yes | Yes | No |

---

## 10. Fork/Join Framework

### 10.1 Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      ForkJoinPool                            │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Worker 1 │  │ Worker 2 │  │ Worker 3 │  │ Worker 4 │   │
│  │ [Deque]  │  │ [Deque]  │  │ [Deque]  │  │ [Deque]  │   │
│  │ Task A   │  │ Task C   │  │          │  │ Task E   │   │
│  │ Task B   │  │ Task D   │  │  steal→  │  │          │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                             │
│  Work Stealing: Idle workers steal from busy workers' deques │
└─────────────────────────────────────────────────────────────┘
```

### 10.2 RecursiveTask (Returns Result)

```java
public class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10_000;
    private final long[] array;
    private final int start, end;
    
    public SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Long compute() {
        int length = end - start;
        
        // Base case: small enough to compute directly
        if (length <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        }
        
        // Recursive case: split and fork
        int mid = start + length / 2;
        SumTask leftTask = new SumTask(array, start, mid);
        SumTask rightTask = new SumTask(array, mid, end);
        
        leftTask.fork(); // Submit to pool
        long rightResult = rightTask.compute(); // Compute directly
        long leftResult = leftTask.join(); // Wait for forked task
        
        return leftResult + rightResult;
    }
}

// Usage
ForkJoinPool pool = ForkJoinPool.commonPool();
long[] array = new long[1_000_000];
Long sum = pool.invoke(new SumTask(array, 0, array.length));
```

### 10.3 RecursiveAction (No Result)

```java
public class SortTask extends RecursiveAction {
    private static final int THRESHOLD = 10_000;
    private final int[] array;
    private final int start, end;
    
    public SortTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected void compute() {
        if (end - start <= THRESHOLD) {
            Arrays.sort(array, start, end);
            return;
        }
        
        int mid = start + (end - start) / 2;
        invokeAll(
            new SortTask(array, start, mid),
            new SortTask(array, mid, end)
        );
        
        merge(array, start, mid, end);
    }
}
```

### 10.4 ForkJoinPool

```java
// Common pool (shared by parallel streams)
ForkJoinPool commonPool = ForkJoinPool.commonPool();

// Custom pool
ForkJoinPool customPool = new ForkJoinPool(
    4,                                    // parallelism
    ForkJoinPool.defaultForkJoinWorkerThreadFactory,
    null,                                 // exception handler
    true                                  // async mode
);

// Submit task
Future<Long> future = pool.submit(task);
Long result = pool.invoke(task); // Submit and wait

// Pool stats
pool.getParallelism();
pool.getActiveThreadCount();
pool.getQueuedTaskCount();
pool.getStealCount();
```

### 10.5 Parallel Streams and ForkJoinPool

```java
// Parallel stream uses common pool
List<Integer> numbers = IntStream.range(0, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

long sum = numbers.parallelStream()
    .mapToLong(Integer::longValue)
    .sum();

// Use custom pool for parallel stream
ForkJoinPool customPool = new ForkJoinPool(8);
long sum = customPool.submit(() ->
    numbers.parallelStream()
        .mapToLong(Integer::longValue)
        .sum()
).join();
```

---

## 11. Java 9-17 Concurrency Enhancements

### 11.1 Java 9: Reactive Streams (Flow API)

```java
// Publisher-Subscriber pattern
public class SimplePublisher implements Flow.Publisher<String> {
    private final List<String> items;
    
    public SimplePublisher(List<String> items) {
        this.items = items;
    }
    
    @Override
    public void subscribe(Flow.Subscriber<? super String> subscriber) {
        subscriber.onSubscribe(new Flow.Subscription() {
            private int index = 0;
            
            @Override
            public void request(long n) {
                for (long i = 0; i < n && index < items.size(); i++) {
                    subscriber.onNext(items.get(index++));
                }
                if (index >= items.size()) {
                    subscriber.onComplete();
                }
            }
            
            @Override
            public void cancel() {
                index = items.size();
            }
        });
    }
}

// SubmissionPublisher - concrete implementation
SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
publisher.subscribe(new Flow.Subscriber<>() {
    private Flow.Subscription subscription;
    
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }
    
    @Override
    public void onNext(String item) {
        System.out.println("Received: " + item);
        subscription.request(1);
    }
    
    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();
    }
    
    @Override
    public void onComplete() {
        System.out.println("Done");
    }
});

publisher.submit("Hello");
publisher.submit("World");
publisher.close();
```

### 11.2 Java 9: CompletableFuture Enhancements

```java
// orTimeout - fails with TimeoutException
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> slowOperation())
    .orTimeout(5, TimeUnit.SECONDS);

// completeOnTimeout - completes with default value
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> slowOperation())
    .completeOnTimeout("default", 5, TimeUnit.SECONDS);

// copy - create defensive copy
CompletableFuture<String> copy = future.copy();

// minimalCompletionStage - returns CompletionStage view
CompletionStage<String> stage = future.minimalCompletionStage();

// completeAsync
CompletableFuture<String> future = new CompletableFuture<>();
future.completeAsync(() -> "result");
future.completeAsync(() -> "result", executor);

// New factory methods
CompletableFuture<String> failed = CompletableFuture.failedFuture(
    new RuntimeException("Error")
);
CompletionStage<String> failedStage = CompletableFuture.failedStage(
    new RuntimeException("Error")
);
```

### 11.3 Java 9: Process API Enhancements

```java
// Get current process
ProcessHandle current = ProcessHandle.current();
System.out.println("PID: " + current.pid());

// Process info
ProcessHandle.Info info = current.info();
info.command().ifPresent(System.out::println);
info.arguments().ifPresent(args -> System.out.println(Arrays.toString(args)));
info.startInstant().ifPresent(System.out::println);
info.totalCpuDuration().ifPresent(System.out::println);
info.user().ifPresent(System.out::println);

// All processes
ProcessHandle.allProcesses()
    .filter(p -> p.info().command().isPresent())
    .limit(10)
    .forEach(p -> System.out.println(p.pid() + ": " + p.info().command().get()));

// Process completion
ProcessBuilder builder = new ProcessBuilder("sleep", "10");
Process process = builder.start();
process.onExit().thenAccept(p -> 
    System.out.println("Process " + p.pid() + " exited")
);

// Destroy process
process.destroy();           // Graceful
process.destroyForcibly();   // Force kill
```

### 11.4 Java 11: HTTP Client (Async Support)

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(10))
    .executor(Executors.newFixedThreadPool(5))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .timeout(Duration.ofSeconds(30))
    .header("Content-Type", "application/json")
    .GET()
    .build();

// Synchronous
HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());

// Asynchronous
CompletableFuture<HttpResponse<String>> futureResponse = 
    client.sendAsync(request, HttpResponse.BodyHandlers.ofString());

futureResponse.thenAccept(resp -> {
    System.out.println("Status: " + resp.statusCode());
    System.out.println("Body: " + resp.body());
});

// Multiple async requests
List<URI> uris = List.of(
    URI.create("https://api1.example.com"),
    URI.create("https://api2.example.com")
);

List<CompletableFuture<String>> futures = uris.stream()
    .map(uri -> HttpRequest.newBuilder(uri).build())
    .map(req -> client.sendAsync(req, HttpResponse.BodyHandlers.ofString()))
    .map(cf -> cf.thenApply(HttpResponse::body))
    .collect(Collectors.toList());

CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
    .thenRun(() -> futures.forEach(f -> System.out.println(f.join())));
```

### 11.5 Java 12: Collectors.teeing

```java
// Process stream with two collectors simultaneously
record Stats(long count, double average) {}

Stats stats = Stream.of(1, 2, 3, 4, 5)
    .collect(Collectors.teeing(
        Collectors.counting(),
        Collectors.averagingInt(i -> i),
        Stats::new
    ));
// Stats[count=5, average=3.0]
```

### 11.6 Java 14-17: Helpful NullPointerExceptions

```java
// Before Java 14
a.b.c.d = 5;
// NullPointerException (which one is null?)

// Java 14+
a.b.c.d = 5;
// NullPointerException: Cannot read field "c" because "a.b" is null
```

### 11.7 Java 15: Hidden Classes

```java
// For frameworks generating classes at runtime
// Hidden classes cannot be discovered by other classes
// Useful for lambda proxy generation
```

### 11.8 Java 16: Records (Immutable Data Carriers)

```java
// Thread-safe by default (immutable)
record Point(int x, int y) {}

// Use in concurrent context
ConcurrentHashMap<String, Point> points = new ConcurrentHashMap<>();
points.put("origin", new Point(0, 0));
```

---

## 12. Virtual Threads (Java 19+ Preview)

### 12.1 Overview

```
Traditional Threads (Platform Threads):
┌─────────────────────────────────────────────────────┐
│  Java Thread  ←→  OS Thread  ←→  CPU Core          │
│  (1:1 mapping, expensive, limited ~thousands)       │
└─────────────────────────────────────────────────────┘

Virtual Threads:
┌─────────────────────────────────────────────────────┐
│  Virtual Threads (millions)                         │
│       │    │    │    │    │                        │
│       ▼    ▼    ▼    ▼    ▼                        │
│  ┌─────────────────────────────────┐               │
│  │      Carrier Threads (few)      │               │
│  │         (Platform Threads)       │               │
│  └─────────────────────────────────┘               │
│                    │                                │
│                    ▼                                │
│            OS Threads / CPU                         │
└─────────────────────────────────────────────────────┘
```

### 12.2 Creating Virtual Threads

```java
// Direct creation
Thread vThread = Thread.ofVirtual()
    .name("virtual-", 0)
    .start(() -> System.out.println("Hello from virtual thread"));

// Using builder
Thread.Builder builder = Thread.ofVirtual().name("worker-", 0);
Thread t1 = builder.start(() -> task1());
Thread t2 = builder.start(() -> task2());

// Executor with virtual threads
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    });
} // Executor auto-closes, waits for all tasks

// Thread.startVirtualThread
Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread");
});
```

### 12.3 Virtual Thread Characteristics

| Aspect | Platform Thread | Virtual Thread |
|--------|-----------------|----------------|
| Memory | ~1MB stack | ~few KB |
| Creation cost | High | Low |
| Count limit | Thousands | Millions |
| Blocking | Blocks OS thread | Unmounts, frees carrier |
| Pooling | Recommended | Not needed |
| Use case | CPU-bound | I/O-bound |

### 12.4 Best Practices for Virtual Threads

```java
// DO: Create virtual thread per task
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Request request : requests) {
        executor.submit(() -> handle(request));
    }
}

// DON'T: Pool virtual threads (defeats the purpose)
ExecutorService pool = Executors.newFixedThreadPool(10); // Wrong!

// DO: Use blocking I/O (it's cheap with virtual threads)
String data = httpClient.send(request, BodyHandlers.ofString()).body();

// DON'T: Hold locks during blocking operations
synchronized (lock) {
    Thread.sleep(1000); // Pins the carrier thread!
}

// DO: Use ReentrantLock instead
lock.lock();
try {
    Thread.sleep(1000); // Virtual thread unmounts properly
} finally {
    lock.unlock();
}
```

### 12.5 Structured Concurrency (Preview)

```java
// Java 19+ Preview: Structured Concurrency
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser());
    Future<String> order = scope.fork(() -> fetchOrder());
    
    scope.join();           // Wait for both
    scope.throwIfFailed();  // Propagate errors
    
    return new Response(user.resultNow(), order.resultNow());
}

// ShutdownOnSuccess - return first successful result
try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
    scope.fork(() -> fetchFromServer1());
    scope.fork(() -> fetchFromServer2());
    
    scope.join();
    return scope.result(); // First successful result
}
```

---

## 13. Common Concurrency Patterns

### 13.1 Singleton Pattern (Thread-Safe)

```java
// 1. Eager initialization
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    private EagerSingleton() {}
    public static EagerSingleton getInstance() { return INSTANCE; }
}

// 2. Initialization-on-demand holder (Recommended)
public class HolderSingleton {
    private HolderSingleton() {}
    private static class Holder {
        static final HolderSingleton INSTANCE = new HolderSingleton();
    }
    public static HolderSingleton getInstance() { return Holder.INSTANCE; }
}

// 3. Enum singleton (Most robust)
public enum EnumSingleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}

// 4. Double-checked locking (if lazy initialization needed)
public class DCLSingleton {
    private static volatile DCLSingleton instance;
    private DCLSingleton() {}
    public static DCLSingleton getInstance() {
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}
```

### 13.2 Producer-Consumer Pattern

```java
// Using BlockingQueue
public class ProducerConsumer {
    private final BlockingQueue<Task> queue;
    
    public ProducerConsumer(int capacity) {
        this.queue = new LinkedBlockingQueue<>(capacity);
    }
    
    public void produce(Task task) throws InterruptedException {
        queue.put(task); // Blocks if full
    }
    
    public Task consume() throws InterruptedException {
        return queue.take(); // Blocks if empty
    }
}

// Using Condition
public class BoundedBuffer<E> {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final Object[] items;
    private int putIndex, takeIndex, count;
    
    public BoundedBuffer(int capacity) {
        items = new Object[capacity];
    }
    
    public void put(E e) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) notFull.await();
            items[putIndex] = e;
            if (++putIndex == items.length) putIndex = 0;
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    @SuppressWarnings("unchecked")
    public E take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) notEmpty.await();
            E e = (E) items[takeIndex];
            items[takeIndex] = null;
            if (++takeIndex == items.length) takeIndex = 0;
            count--;
            notFull.signal();
            return e;
        } finally {
            lock.unlock();
        }
    }
}
```

### 13.3 Read-Write Lock Pattern

```java
public class ReadWriteCache<K, V> {
    private final Map<K, V> cache = new HashMap<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    public V get(K key) {
        lock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void put(K key, V value) {
        lock.writeLock().lock();
        try {
            cache.put(key, value);
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    public V computeIfAbsent(K key, Function<K, V> function) {
        // Try read lock first
        lock.readLock().lock();
        try {
            V value = cache.get(key);
            if (value != null) return value;
        } finally {
            lock.readLock().unlock();
        }
        
        // Upgrade to write lock
        lock.writeLock().lock();
        try {
            // Double-check
            V value = cache.get(key);
            if (value == null) {
                value = function.apply(key);
                cache.put(key, value);
            }
            return value;
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

### 13.4 Thread Pool Pattern

```java
public class CustomThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<WorkerThread> workers;
    private volatile boolean isShutdown = false;
    
    public CustomThreadPool(int numThreads, int queueCapacity) {
        taskQueue = new LinkedBlockingQueue<>(queueCapacity);
        workers = new ArrayList<>(numThreads);
        
        for (int i = 0; i < numThreads; i++) {
            WorkerThread worker = new WorkerThread();
            workers.add(worker);
            worker.start();
        }
    }
    
    public void execute(Runnable task) {
        if (isShutdown) throw new IllegalStateException("Pool is shutdown");
        try {
            taskQueue.put(task);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void shutdown() {
        isShutdown = true;
        workers.forEach(Thread::interrupt);
    }
    
    private class WorkerThread extends Thread {
        @Override
        public void run() {
            while (!isShutdown || !taskQueue.isEmpty()) {
                try {
                    Runnable task = taskQueue.poll(1, TimeUnit.SECONDS);
                    if (task != null) task.run();
                } catch (InterruptedException e) {
                    // Check shutdown flag
                }
            }
        }
    }
}
```

### 13.5 CountDownLatch Pattern

```java
// Wait for multiple threads to complete
public class ServiceStarter {
    public void startServices() throws InterruptedException {
        int serviceCount = 5;
        CountDownLatch latch = new CountDownLatch(serviceCount);
        
        for (int i = 0; i < serviceCount; i++) {
            final int serviceId = i;
            new Thread(() -> {
                try {
                    initializeService(serviceId);
                } finally {
                    latch.countDown();
                }
            }).start();
        }
        
        latch.await(); // Wait for all services
        System.out.println("All services started");
    }
}
```

### 13.6 CyclicBarrier Pattern

```java
// Threads wait for each other at barrier point
public class ParallelComputation {
    public void compute() {
        int parties = 4;
        CyclicBarrier barrier = new CyclicBarrier(parties, () -> {
            System.out.println("All parties reached barrier, merging results...");
        });
        
        for (int i = 0; i < parties; i++) {
            final int partition = i;
            new Thread(() -> {
                try {
                    // Phase 1
                    processPartition(partition);
                    barrier.await();
                    
                    // Phase 2 (all threads start together)
                    aggregateResults(partition);
                    barrier.await(); // Barrier is cyclic, can reuse
                    
                } catch (InterruptedException | BrokenBarrierException e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }
}
```

### 13.7 Semaphore Pattern

```java
// Limit concurrent access to resource
public class ConnectionPool {
    private final Semaphore semaphore;
    private final BlockingQueue<Connection> pool;
    
    public ConnectionPool(int poolSize) {
        semaphore = new Semaphore(poolSize);
        pool = new LinkedBlockingQueue<>(poolSize);
        for (int i = 0; i < poolSize; i++) {
            pool.add(createConnection());
        }
    }
    
    public Connection acquire() throws InterruptedException {
        semaphore.acquire(); // Wait for permit
        return pool.take();
    }
    
    public void release(Connection conn) {
        pool.add(conn);
        semaphore.release(); // Return permit
    }
    
    // Try acquire with timeout
    public Connection tryAcquire(long timeout, TimeUnit unit) 
            throws InterruptedException {
        if (semaphore.tryAcquire(timeout, unit)) {
            return pool.take();
        }
        return null;
    }
}
```

### 13.8 Phaser Pattern (Java 7+)

```java
// Flexible barrier with dynamic party registration
public class PhasedComputation {
    public void compute() {
        Phaser phaser = new Phaser(1); // Register self
        
        for (int i = 0; i < 5; i++) {
            phaser.register(); // Register new party
            new Thread(() -> {
                try {
                    // Phase 0
                    doWork();
                    phaser.arriveAndAwaitAdvance();
                    
                    // Phase 1
                    doMoreWork();
                    phaser.arriveAndDeregister(); // Done, deregister
                } catch (Exception e) {
                    phaser.arriveAndDeregister();
                }
            }).start();
        }
        
        phaser.arriveAndDeregister(); // Deregister self
    }
}
```

---

## 14. Common Interview Questions

### 14.1 Thread Basics

**Q1: What is the difference between `start()` and `run()`?**
> `start()` creates a new thread and executes `run()` in that thread. Calling `run()` directly executes it in the current thread like a normal method call, without creating a new thread.

**Q2: Why is `wait()` called inside a loop?**
> Due to spurious wakeups - a thread can wake up without being notified. The loop re-checks the condition to ensure it's actually met before proceeding.

**Q3: What is thread starvation and how to prevent it?**
> Starvation occurs when a thread cannot get CPU time due to other high-priority threads. Prevention: use fair locks (`new ReentrantLock(true)`), avoid setting thread priorities, use proper synchronization.

### 14.2 Synchronization

**Q4: Difference between `synchronized` and `ReentrantLock`?**

| Aspect | synchronized | ReentrantLock |
|--------|--------------|---------------|
| Fairness | No | Optional |
| Interruptible | No | Yes |
| Try lock | No | Yes |
| Multiple conditions | No | Yes |
| Unlock location | Same block | Anywhere |
| Performance | Similar | Similar (Java 6+) |

**Q5: What is the difference between `notify()` and `notifyAll()`?**
> `notify()` wakes one waiting thread (unspecified which). `notifyAll()` wakes all waiting threads. Prefer `notifyAll()` to avoid missed signals when multiple threads wait on different conditions.

**Q6: Explain double-checked locking and its issues.**
> Double-checked locking is a pattern to reduce synchronization overhead in lazy initialization. Without `volatile`, it can fail due to instruction reordering - the reference may be assigned before object construction completes.

### 14.3 Memory Model

**Q7: What guarantees does `volatile` provide?**
> 1. **Visibility**: All threads see the latest value
> 2. **Ordering**: Prevents reordering of reads/writes around volatile access
> 3. Does NOT provide atomicity for compound operations like `i++`

**Q8: What is happens-before relationship?**
> A guarantee that memory writes by one action are visible to another action. Key relationships: program order, monitor lock, volatile variable, thread start, thread join.

### 14.4 Executor Framework

**Q9: Difference between `submit()` and `execute()`?**

| Aspect | execute() | submit() |
|--------|-----------|----------|
| Return | void | Future |
| Exception | Thrown to UncaughtExceptionHandler | Wrapped in Future |
| Accepts | Runnable | Runnable or Callable |

**Q10: What happens when thread pool queue is full?**
> Depends on rejection policy:
> - AbortPolicy: Throws RejectedExecutionException
> - CallerRunsPolicy: Caller thread executes task
> - DiscardPolicy: Silently discards task
> - DiscardOldestPolicy: Discards oldest queued task

### 14.5 CompletableFuture

**Q11: Difference between `thenApply()` and `thenCompose()`?**
> `thenApply()` is like `map()` - transforms result with a function returning plain value.
> `thenCompose()` is like `flatMap()` - transforms result with a function returning CompletableFuture, avoiding nested futures.

**Q12: How to handle exceptions in CompletableFuture chain?**
> - `exceptionally()`: Recover from exception with fallback value
> - `handle()`: Handle both success and failure
> - `whenComplete()`: Peek at result/exception without transforming

### 14.6 Concurrent Collections

**Q13: Why does ConcurrentHashMap not allow null keys/values?**
> Ambiguity: If `get(key)` returns null, you can't tell if key doesn't exist or value is null. In concurrent context, checking `containsKey()` then `get()` isn't atomic.

**Q14: When to use CopyOnWriteArrayList?**
> When reads vastly outnumber writes. It creates a new array copy on every write, so iteration is thread-safe and fast, but writes are expensive.

### 14.7 Atomic Classes

**Q15: What is ABA problem and how to solve it?**
> ABA problem: Value changes A→B→A, CAS succeeds but misses intermediate change.
> Solution: Use `AtomicStampedReference` which tracks a version stamp along with the value.

**Q16: When to use LongAdder vs AtomicLong?**
> LongAdder: High contention (many threads incrementing), sum needed only at end.
> AtomicLong: Low contention, need current value frequently, need sequence numbers.

### 14.8 Advanced Questions

**Q17: Explain ForkJoinPool work-stealing algorithm.**
> Each worker thread has a double-ended queue (deque). Tasks are pushed/popped from one end by owner thread. When a thread's queue is empty, it steals from another thread's queue from the opposite end, minimizing contention.

**Q18: What are Virtual Threads and when to use them?**
> Lightweight threads managed by JVM, not OS. Create millions of them. Best for I/O-bound tasks with blocking operations. Don't pool them. Avoid holding locks during blocking operations.

**Q19: Design a thread-safe LRU Cache.**
```java
public class ConcurrentLRUCache<K, V> {
    private final int capacity;
    private final Map<K, V> cache;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    
    public ConcurrentLRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > ConcurrentLRUCache.this.capacity;
            }
        };
    }
    
    public V get(K key) {
        lock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void put(K key, V value) {
        lock.writeLock().lock();
        try {
            cache.put(key, value);
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

**Q20: How would you implement a rate limiter?**
```java
public class TokenBucketRateLimiter {
    private final long maxTokens;
    private final long refillRate; // tokens per second
    private long availableTokens;
    private long lastRefillTime;
    
    public TokenBucketRateLimiter(long maxTokens, long refillRate) {
        this.maxTokens = maxTokens;
        this.refillRate = refillRate;
        this.availableTokens = maxTokens;
        this.lastRefillTime = System.nanoTime();
    }
    
    public synchronized boolean tryAcquire() {
        refill();
        if (availableTokens > 0) {
            availableTokens--;
            return true;
        }
        return false;
    }
    
    private void refill() {
        long now = System.nanoTime();
        long elapsedNanos = now - lastRefillTime;
        long tokensToAdd = elapsedNanos * refillRate / 1_000_000_000;
        if (tokensToAdd > 0) {
            availableTokens = Math.min(maxTokens, availableTokens + tokensToAdd);
            lastRefillTime = now;
        }
    }
}
```

---

## 15. Best Practices and Pitfalls

### 15.1 Thread Safety Best Practices

```java
// 1. Prefer immutability
public final class ImmutablePerson {
    private final String name;
    private final int age;
    
    public ImmutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // Only getters, no setters
}

// 2. Use thread-safe collections
Map<String, String> map = new ConcurrentHashMap<>();
List<String> list = new CopyOnWriteArrayList<>();

// 3. Minimize lock scope
public void process() {
    // Do non-critical work outside lock
    prepareData();
    
    synchronized (lock) {
        // Only critical section
        criticalOperation();
    }
    
    // Do more non-critical work
    postProcess();
}

// 4. Always release locks in finally
Lock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}

// 5. Use higher-level concurrency utilities
// Instead of wait/notify, use:
BlockingQueue<Task> queue = new LinkedBlockingQueue<>();
CountDownLatch latch = new CountDownLatch(5);
Semaphore semaphore = new Semaphore(10);
```

### 15.2 Common Pitfalls

```java
// 1. WRONG: Swallowing InterruptedException
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Don't ignore!
}

// CORRECT: Restore interrupt status
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    // Handle interruption
}

// 2. WRONG: Double-checked locking without volatile
private static Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton(); // May see partially constructed
            }
        }
    }
    return instance;
}

// 3. WRONG: Synchronizing on non-final field
private Object lock = new Object();
public void method() {
    synchronized (lock) { // Lock reference can change!
        lock = new Object(); // BUG!
    }
}

// CORRECT: Use final lock
private final Object lock = new Object();

// 4. WRONG: Publishing this reference in constructor
public class Unsafe {
    public Unsafe() {
        EventSource.register(this); // Other threads see incomplete object
    }
}

// 5. WRONG: Synchronizing on String literal
synchronized ("lock") { // All classes share same String from pool
    // ...
}

// CORRECT: Use dedicated lock object
private final Object lock = new Object();

// 6. WRONG: Calling alien methods while holding lock
synchronized (lock) {
    listener.onEvent(event); // Listener might try to acquire same lock
}

// CORRECT: Copy and iterate outside lock
List<Listener> snapshot;
synchronized (lock) {
    snapshot = new ArrayList<>(listeners);
}
snapshot.forEach(l -> l.onEvent(event));
```

### 15.3 Performance Guidelines

| Scenario | Recommendation |
|----------|----------------|
| High read, low write | ReadWriteLock or StampedLock |
| High contention counter | LongAdder |
| Cache | ConcurrentHashMap |
| Event listeners | CopyOnWriteArrayList |
| Task queue | LinkedBlockingQueue |
| Thread pool for CPU tasks | Fixed pool, size = CPU cores |
| Thread pool for I/O tasks | Larger pool or Virtual Threads |
| Async composition | CompletableFuture |
| Parallel computation | Fork/Join or parallel streams |

### 15.4 Testing Concurrent Code

```java
// 1. Use CountDownLatch for coordination
@Test
void testConcurrentAccess() throws InterruptedException {
    int threadCount = 100;
    CountDownLatch startLatch = new CountDownLatch(1);
    CountDownLatch endLatch = new CountDownLatch(threadCount);
    
    for (int i = 0; i < threadCount; i++) {
        new Thread(() -> {
            try {
                startLatch.await(); // Wait for signal
                // Perform concurrent operation
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                endLatch.countDown();
            }
        }).start();
    }
    
    startLatch.countDown(); // Start all threads
    endLatch.await(); // Wait for completion
    
    // Verify results
}

// 2. Use CyclicBarrier for phased testing
@Test
void testWithBarrier() throws InterruptedException {
    CyclicBarrier barrier = new CyclicBarrier(10);
    // ... similar pattern
}

// 3. Consider using jcstress for stress testing
// https://github.com/openjdk/jcstress
```

---

## Quick Reference Card

### Thread States

| State | Caused By |
|-------|-----------|
| NEW | `new Thread()` |
| RUNNABLE | `start()` |
| BLOCKED | Waiting for monitor |
| WAITING | `wait()`, `join()`, `park()` |
| TIMED_WAITING | `sleep()`, `wait(t)`, `join(t)` |
| TERMINATED | `run()` completes |

### Lock Comparison

| Feature | synchronized | ReentrantLock | StampedLock |
|---------|-------------|---------------|-------------|
| Fairness | No | Optional | No |
| Interruptible | No | Yes | Yes |
| Try lock | No | Yes | Yes |
| Conditions | 1 | Multiple | No |
| Optimistic | No | No | Yes |
| Reentrant | Yes | Yes | No |

### ExecutorService Methods

| Method | Blocks | Returns |
|--------|--------|---------|
| `execute(Runnable)` | No | void |
| `submit(Runnable)` | No | Future<?> |
| `submit(Callable)` | No | Future<T> |
| `invokeAll(Collection)` | Yes | List<Future<T>> |
| `invokeAny(Collection)` | Yes | T |

### CompletableFuture Cheat Sheet

| Method | Input | Output | Async Version |
|--------|-------|--------|---------------|
| thenApply | T | U | thenApplyAsync |
| thenAccept | T | void | thenAcceptAsync |
| thenRun | - | void | thenRunAsync |
| thenCompose | T | CF<U> | thenComposeAsync |
| handle | T/Throwable | U | handleAsync |
| exceptionally | Throwable | T | - |

### Java Version Features

| Version | Key Concurrency Features |
|---------|--------------------------|
| Java 5 | Executor, Lock, Atomic, Concurrent collections |
| Java 7 | Fork/Join, Phaser |
| Java 8 | CompletableFuture, StampedLock, LongAdder, parallel streams |
| Java 9 | Flow API, CF.orTimeout/completeOnTimeout, Process API |
| Java 11 | HTTP Client async support |
| Java 16 | Records (immutable) |
| Java 19+ | Virtual Threads (Preview) |
| Java 21 | Virtual Threads (GA), Structured Concurrency |

---

*Last Updated: 2024 | Covers Java 8 - Java 17 (with Java 19+ previews)*
