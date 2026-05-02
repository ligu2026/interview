I've put together 100 essential Java concurrency and multithreading questions to help you prepare, divided into clear categories. Each question comes with a straightforward answer, and where possible, a practical code example. Let's dive in.

### 📌 Question Index: Your High-Level Roadmap

*   **Q1-Q5: Fundamentals & Thread Basics** – Processes, threads, and thread states.
*   **Q6-Q20: Concurrency, Synchronization & Locks** – Shared resources, deadlocks, and lock mechanisms.
*   **Q21-Q26: Visibility & Atomicity** – `volatile`, CAS, and atomic classes.
*   **Q27-Q37: High-Level Concurrency with `java.util.concurrent`** – `ExecutorService`, `Future`, and thread pools.
*   **Q38-Q46: Thread Coordination & Data Structures** – `Condition`, `Semaphore`, and concurrent collections.
*   **Q47-Q52: Advanced Topics & Memory Management** – `ThreadLocal`, `ForkJoinPool`, and JMM.
*   **Q53-Q58: Producer-Consumer Problem** – The classic problem with various solutions.
*   **Q59-Q63: Performance & Testing** – Tools and techniques for identifying bottlenecks.
*   **Q64-Q67: Patterns & Best Practices** – Safe singletons and avoiding thread leaks.
*   **Q68-Q72: Comparing Concurrency Mechanisms** – Synchronized vs. locks, `Runnable` vs. `Callable`.
*   **Q73-Q80: Background & Utility Threads** – Daemon threads and thread-local storage.
*   **Q81-Q84: Framework & Environment Integration** – Concurrency in Spring and Tomcat.
*   **Q85-Q90: Advanced `java.util.concurrent` Utilities** – `CompletableFuture`, `Phaser`, `Exchanger`.
*   **Q91-Q100: In-Depth API & Usage Questions** – Fine-grained API behavior and usage scenarios.

---

### 🏆 Fundamentals & Thread Basics (Q1–Q5)

**1. What are the differences between a process and a thread? How do they relate to each other?**

A **process** is an independent program in execution with its own memory space, while a **thread** is a lightweight unit of execution within a process, sharing the process's memory and resources. This shared nature allows for efficient communication but introduces challenges for thread safety.

**2. What is multithreading, and what are its primary benefits in Java?**

Multithreading is a programming technique that enables concurrent execution of multiple threads, allowing a program to handle several tasks simultaneously. Its primary benefits include improved resource utilization, better application responsiveness, and the ability to leverage multi-core processors for parallel computation.

**3. What is a daemon thread in Java? Provide an example.**

A **daemon thread** is a low-priority background thread that performs tasks like garbage collection or monitoring. The JVM terminates when only daemon threads remain. In contrast, user threads execute application-specific work.

```java
// Creating a daemon thread
Thread daemonThread = new Thread(() -> {
    while (true) {
        // Background task
    }
});
daemonThread.setDaemon(true);
daemonThread.start();
```

**4. What are the possible states in a thread's lifecycle?**

*   **NEW**: The thread is created but not started.
*   **RUNNABLE**: The thread is executing or ready to execute.
*   **BLOCKED**: The thread is waiting for a monitor lock.
*   **WAITING**: The thread waits indefinitely for another thread's action.
*   **TIMED_WAITING**: The thread waits for a specified time.
*   **TERMINATED**: The thread has completed execution.

**5. What is a `ThreadLocal` variable? Can it cause a memory leak?**

A `ThreadLocal` provides each thread with its own copy of a variable, which is useful for thread-safe operations without explicit synchronization. However, **memory leaks can occur** in web applications because containers often maintain thread pools, and if you forget to remove `ThreadLocal` entries after use, they can accumulate and cause memory leaks.

### 🔒 Concurrency, Synchronization & Locks (Q6–Q20)

**6. What is thread safety and what are the ways to achieve it in Java?**

A class is **thread-safe** when it behaves correctly in a multithreaded environment without requiring external synchronization. Strategies include using synchronized blocks/methods, `volatile` variables, atomic classes, concurrent collections (like `ConcurrentHashMap`), and immutable objects.

**7. How does `synchronized` work in Java?**

The `synchronized` keyword provides built-in monitor locks. When a thread enters a synchronized block, it acquires the lock for the object or class. Other threads are blocked until the lock is released.

**8. What is the difference between a synchronized method and a synchronized block?**

A `synchronized` method locks the entire method, while a block allows you to lock a specific section. **Synchronized blocks** are generally more efficient because they lock only when necessary, reducing contention.

```java
public class Counter {
    private int count = 0;
    private Object lock = new Object();

    // Method lock
    public synchronized void methodLockIncrement() {
        count++;
    }

    // Block lock.
    public void blockLockIncrement() {
        synchronized(lock) {
            count++;
        }
    }
}
```

**9. What is a deadlock? How can you prevent it?**

A **deadlock** occurs when two or more threads are blocked forever, each waiting for the other to release a lock. Prevention strategies include:
- Lock ordering
- Using `tryLock()` with timeouts
- Avoiding nested locks

```java
// Example of potential deadlock
public void method1() {
    synchronized(lockA) {
        synchronized(lockB) {
            // This can deadlock if method2 acquires locks in opposite order
        }
    }
}
```

**10. What is the difference between `ReentrantLock` and `synchronized`?**

`ReentrantLock` offers more flexibility than `synchronized`:
- It provides fairness policies (fair lock can be requested)
- It includes methods like `tryLock()` to attempt lock acquisition without indefinite blocking
- It requires manual unlock in a `finally` block

**11. Explain the concept of lock striping in Java with an example.**

Lock striping divides a data structure into segments, each protected by its own lock. It improves concurrency because threads can operate on different segments simultaneously. This technique is used in `ConcurrentHashMap` (pre-Java 8) which used segment-level locking.

**12. What is a read-write lock, and when would you use it?**

A **ReadWriteLock** maintains a pair of locks: one for read operations and another for writes. Multiple threads can hold the read lock simultaneously, while the write lock is exclusive. It's ideal for data structures where reads occur more frequently than writes, boosting concurrency.

**13. What is the `Condition` interface, and how does it improve thread coordination?**

The `Condition` interface, used with `ReentrantLock`, provides advanced thread coordination beyond the built-in `wait()`/`notify()` methods. A single lock can have multiple `Condition` objects, each managing a separate queue, which enables more precise control. For example, you can have "not full" and "not empty" conditions to wait and signal on each independently.

**14. What is the difference between `Lock` and `synchronized`?**

`Lock` is an interface with implementations like `ReentrantLock`. Compared to `synchronized`, it offers more features (like `tryLock()`), but requires explicit lock and unlock operations, increasing the risk of forgetting to unlock.

**15. What is the `ReentrantLock` fairness policy? How does it work?**

A **fair lock** in `ReentrantLock` processes lock requests in thread arrival order, preventing starvation. Fair locks can impact performance, so they are not the default. In a fair lock, threads acquire the lock in the order they requested it, while unfair locks allow thread "barging."

**16. What is lock contention? How can you reduce it?**

Lock contention occurs when multiple threads compete for the same lock, reducing parallelism. You can reduce it by minimizing synchronized block scope, using lock striping, employing concurrent data structures, or considering optimistic locking.

**17. Explain the difference between `wait()`, `sleep()`, and `yield()` methods.**

- `wait()`: Releases the lock and suspends the thread until notified
- `sleep()`: Suspends the thread without releasing any locks
- `yield()`: Hints the scheduler that the current thread is willing to give up its CPU time

**18. What is a livelock, and how is it different from a deadlock?**

A **livelock** occurs when threads continuously change states in response to each other but make no progress. This differs from a deadlock where threads are simply waiting.

**19. What is a race condition, and how can you identify it?**

A **race condition** occurs when multiple threads access shared data simultaneously, and the program's correctness depends on the unpredictable timing of thread execution. They're notoriously difficult to debug because issues may only manifest under specific timing conditions.

**20. What is thread starvation, and how can you prevent it?**

Starvation happens when a thread is perpetually denied access to resources it needs to proceed. You can prevent it by using fair locks, adjusting thread priorities appropriately, or using `tryLock()` with timeouts to allow threads to back off.

### 🧠 Visibility & Atomicity (Q21–Q26)

**21. What is the `volatile` keyword, and when should you use it?**

The `volatile` keyword ensures visibility of changes to variables across threads by preventing their caching locally. It doesn't guarantee atomicity, but it's suitable for flags, status indicators, or when only one thread writes and others read.

**22. What is the AtomicInteger class, and how does it differ from `volatile`?**

`AtomicInteger` provides atomic operations like `incrementAndGet()`, which are not possible with `volatile` alone. While `volatile` ensures visibility, `AtomicInteger` provides both visibility and atomicity.

**23. What is a Compare-And-Swap (CAS) operation? What are its three problems?**

CAS is an atomic instruction that compares a variable's current value with an expected value, and if they match, it replaces the variable with a new value. Its three main problems are:
- ABA problem (a value changes from A to B and back to A, tricking CAS)
- Long loop times in high contention scenarios
- It can only handle a single shared variable atomically

**24. What is the ABA problem in the context of CAS, and how can it be solved?**

The ABA problem occurs when a value changes from A to B and back to A, making the CAS operation incorrectly believe nothing has changed. Solutions include using version numbers (like `AtomicStampedReference` or `AtomicMarkableReference`) to detect changes.

**25. How do you handle long-running CAS loops in high-contention scenarios?**

Options include using backoff strategies, switching to locks for high contention, or using alternative atomic classes like `LongAdder` for certain workloads.

**26. Can CAS be used to atomically update multiple variables?**

No, CAS cannot atomically update multiple variables. You can combine them into an object and use `AtomicReference` to update the entire object atomically, or use locks for complex updates.

### 🛠️ High-Level Concurrency with `java.util.concurrent` (Q27–Q37)

**27. What is an `ExecutorService`, and why use it over raw threads?**

An `ExecutorService` manages thread pools, simplifies task submission, and provides lifecycle management. It's better than raw threads because it reuses threads, reduces creation overhead, and offers methods to track and coordinate tasks.

**28. What is a `Future`, and how does it relate to `ExecutorService`?**

A `Future` represents the result of an asynchronous computation submitted to an `ExecutorService`. It provides methods to check if the task is complete, cancel it, wait for completion, and retrieve the result.

**29. What is the difference between `submit()` and `execute()` methods?**

`submit()` returns a `Future` that can be used to track the task's progress and retrieve results, while `execute()` doesn't return any result.

**30. How do you gracefully shut down an `ExecutorService`?**

Proper shutdown involves calling `shutdown()` to prevent new task submissions, then waiting for existing tasks to complete using `awaitTermination()`. You can call `shutdownNow()` as an immediate but potentially unsafe alternative.

```java
public void shutdownExecutor(ExecutorService executor) {
    executor.shutdown(); // Disable new tasks
    try {
        if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
            executor.shutdownNow(); // Cancel currently executing tasks
        }
    } catch (InterruptedException e) {
        executor.shutdownNow();
    }
}
```

**31. What are the different types of thread pools provided by `Executors`?**

The `Executors` utility class provides several factory methods:
- `newFixedThreadPool(int n)`: Returns a pool with a fixed number of threads
- `newCachedThreadPool()`: Creates threads as needed, reuses available ones
- `newSingleThreadExecutor()`: Uses a single worker thread
- `newScheduledThreadPool(int corePoolSize)`: For scheduling commands with delays or periodic execution

**32. Explain the core parameters of `ThreadPoolExecutor`.**

The core parameters are:
- `corePoolSize`: Number of threads to keep in the pool
- `maximumPoolSize`: Maximum number of threads allowed
- `keepAliveTime`: Time excess threads wait for new tasks before terminating
- `unit`: Time unit for `keepAliveTime`
- `workQueue`: Queue to hold tasks before execution
- `threadFactory`: Factory for creating new threads
- `handler`: Saturation policy when task can't be accepted

**33. How does `ThreadPoolExecutor` scale threads based on the number of submitted tasks?**

It starts with `corePoolSize` threads. When all are busy and the queue fills, it creates new threads up to `maximumPoolSize`. Once the load decreases, it terminates idle threads after `keepAliveTime`.

**34. What is the difference between `execute()` and `submit()` in the Executor framework?**

`execute()` accepts a `Runnable` task with no return value, while `submit()` accepts either a `Runnable` or `Callable` and returns a `Future`. `submit()` is the only way to execute a `Callable`.

**35. What are the task saturation policies for `ThreadPoolExecutor`?**

When the thread pool is saturated, the `RejectedExecutionHandler` defines the policy:
- `AbortPolicy`: Throws `RejectedExecutionException` (default)
- `CallerRunsPolicy`: Executes the task in the caller's thread
- `DiscardPolicy`: Silently discards the task
- `DiscardOldestPolicy`: Discards the oldest queued task and retries

**36. How does `ScheduledExecutorService` work?**

`ScheduledExecutorService` is a subinterface of `ExecutorService` that provides methods to schedule tasks with delays or at fixed rates. It's more flexible than `Timer` because it uses a thread pool and can handle multiple scheduled tasks.

**37. What is the difference between `scheduleAtFixedRate` and `scheduleWithFixedDelay`?**

- `scheduleAtFixedRate`: Runs a task at a fixed interval regardless of execution time
- `scheduleWithFixedDelay`: Waits a fixed amount of time after each task completes before starting the next

### 🤝 Thread Coordination & Data Structures (Q38–Q46)

**38. What is a blocking queue, and how does it simplify multithreading?**

`BlockingQueue` is a thread-safe queue that blocks when trying to add to a full queue (`put()`) or remove from an empty queue (`take()`). This simplifies the classic producer-consumer problem by eliminating explicit synchronization.

**39. How does `ConcurrentHashMap` achieve thread-safety and high concurrency?**

Modern `ConcurrentHashMap` uses fine-grained locking and CAS operations. It's broken into bins, and each bin can be locked separately, allowing multiple threads to access different bins simultaneously. This provides better concurrency and scalability than a fully synchronized map.

**40. When would you use `CopyOnWriteArrayList`?**

`CopyOnWriteArrayList` creates a new copy of the underlying array for every modification, making it ideal for read-heavy scenarios where writes are rare. It's thread-safe without explicit synchronization and provides fail-safe iterators.

**41. Which concurrency collection would you use for a scenario with very frequent modifications and moderate reads?**

`ConcurrentHashMap` or `ConcurrentLinkedQueue` would be good choices depending on the access pattern. If modifications are frequent, avoid `CopyOnWriteArrayList` due to its copying cost for each write.

**42. Explain the fail-fast vs. fail-safe iterators in Java collections.**

- **Fail-fast**: Iterators that throw `ConcurrentModificationException` if the collection is modified during iteration (e.g., `ArrayList`)
- **Fail-safe**: Iterators that operate on a snapshot of the data (e.g., `CopyOnWriteArrayList`), allowing modifications during iteration without exceptions

**43. How does `PriorityBlockingQueue` work?**

`PriorityBlockingQueue` is an unbounded blocking queue that orders elements according to their natural order or a provided comparator.

**44. What is a `Semaphore`, and how does it work?**

A `Semaphore` maintains a set of permits. `acquire()` consumes a permit (blocking if none available), and `release()` adds a permit back. It's useful for controlling access to a limited pool of resources.

**45. How would you implement a simple thread-safe counter using `AtomicInteger`?**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadSafeCounter {
    private final AtomicInteger counter = new AtomicInteger(0);
    
    public int increment() {
        return counter.incrementAndGet(); // Thread-safe increment
    }
    
    public int get() {
        return counter.get();
    }
}
```

**46. In what scenario is `ConcurrentSkipListMap` superior to `ConcurrentHashMap`?**

`ConcurrentSkipListMap` maintains a navigable sorted order, making it superior when you need sorted traversal, range queries, or approximate matches. `ConcurrentHashMap` offers better average performance for simple lookups.

### 🧩 Advanced Topics & Memory Management (Q47–Q52)

**47. What is the `happens-before` relationship in the Java Memory Model?**

The `happens-before` relationship guarantees that memory writes from one thread are visible to another thread under certain conditions (e.g., unlocking a monitor, starting a thread, or writing to a volatile variable). This formal definition replaces the vague "before" notion with precise ordering guarantees.

**48. What's the difference between `synchronized` and `ReentrantLock` in terms of performance?**

Generally, `synchronized` has been optimized over the years and performs similarly to `ReentrantLock` in low-contention scenarios. `ReentrantLock` may have better performance under high contention due to advanced features like `tryLock()` and fairness policies.

**49. What is the `ForkJoinPool`, and what is work-stealing?**

`ForkJoinPool` is designed for divide-and-conquer algorithms where tasks can be recursively split. **Work-stealing** allows idle worker threads to take tasks from busy threads' queues, improving CPU utilization and throughput for parallel tasks.

**50. What is false sharing, and how can you avoid it in Java?**

False sharing occurs when threads on different cores modify variables that reside on the same CPU cache line, causing unnecessary cache invalidation. You can avoid it by padding data structures to ensure hot fields occupy separate cache lines or using the `@Contended` annotation.

**51. How does the JVM implement `synchronized` under the hood?**

Java's `synchronized` uses a **monitor** associated with each object. The JVM applies **biased locking**, **lightweight locking**, and **heavyweight locking** as optimizations. Locks can be upgraded (never downgraded) based on contention levels.

**52. Explain the concept of reentrancy in locks.**

A lock is **reentrant** if a thread that already holds the lock can acquire it again without blocking. This prevents deadlocks when synchronized methods call other synchronized methods on the same object. Both `synchronized` blocks and `ReentrantLock` are reentrant.

### 🏭 Producer-Consumer Problem Deep Dive (Q53–Q58)

**53. What is the classic Producer-Consumer problem?**

It's a scenario where producer threads generate data and consumer threads process that data, sharing a common buffer. Producers should block when the buffer is full, and consumers block when it's empty. The challenge is coordinating access without race conditions or deadlocks.

**54. How can you solve the Producer-Consumer problem with `BlockingQueue`?**

`BlockingQueue` is the simplest solution because `put()` blocks when full and `take()` blocks when empty.

```java
public class ProducerConsumerExample {
    private static final int CAPACITY = 10;
    private static final BlockingQueue<Integer> QUEUE = new ArrayBlockingQueue<>(CAPACITY);
    
    static class Producer implements Runnable {
        @Override
        public void run() {
            try {
                for (int i = 0; i < 100; i++) {
                    QUEUE.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    static class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                while (true) {
                    Integer value = QUEUE.take();
                    System.out.println("Consumed: " + value);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(4);
        executor.submit(new Producer());
        executor.submit(new Consumer());
        executor.shutdown();
    }
}
```

**55. How do you solve the Producer-Consumer problem with `wait()` and `notify()`?**

The classic `wait()`/`notify()` solution uses a synchronized buffer.

```java
public class SharedBuffer {
    private final int[] buffer;
    private int count = 0, in = 0, out = 0;
    
    public SharedBuffer(int size) {
        buffer = new int[size];
    }
    
    public synchronized void produce(int value) throws InterruptedException {
        while (count == buffer.length) {
            wait(); // Buffer is full, wait for consumer
        }
        buffer[in] = value;
        in = (in + 1) % buffer.length;
        count++;
        System.out.println("Produced: " + value);
        notifyAll(); // Notify waiting consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        while (count == 0) {
            wait(); // Buffer is empty, wait for producer
        }
        int value = buffer[out];
        out = (out + 1) % buffer.length;
        count--;
        System.out.println("Consumed: " + value);
        notifyAll(); // Notify waiting producers
        return value;
    }
}
```

**56. What is the "spurious wakeup" problem in the context of `wait()`/`notify()`?**

A **spurious wakeup** is when a thread wakes up from `wait()` without being notified, interrupted, or timed out. This is why `wait()` should always be called in a loop, re-checking the condition.

**57. How can semaphores be used to solve the Producer-Consumer problem?**

Binary semaphores or counting semaphores can control access to the buffer.

```java
import java.util.concurrent.Semaphore;

public class SemaphoreBuffer {
    private final int[] buffer;
    private int in = 0, out = 0;
    private final Semaphore emptySlots; // Counts empty slots
    private final Semaphore fullSlots;  // Counts filled slots
    private final Semaphore mutex;       // Protects buffer indices
    
    public SemaphoreBuffer(int size) {
        buffer = new int[size];
        emptySlots = new Semaphore(size);
        fullSlots = new Semaphore(0);
        mutex = new Semaphore(1);
    }
    
    public void produce(int value) throws InterruptedException {
        emptySlots.acquire(); // Wait for empty slot
        mutex.acquire();      // Enter critical section
        buffer[in] = value;
        in = (in + 1) % buffer.length;
        System.out.println("Produced: " + value);
        mutex.release();      // Exit critical section
        fullSlots.release();  // Signal filled slot
    }
    
    public int consume() throws InterruptedException {
        fullSlots.acquire();  // Wait for filled slot
        mutex.acquire();      // Enter critical section
        int value = buffer[out];
        out = (out + 1) % buffer.length;
        System.out.println("Consumed: " + value);
        mutex.release();      // Exit critical section
        emptySlots.release(); // Signal empty slot
        return value;
    }
}
```

**58. What's the difference between using `BlockingQueue` vs manual `wait()`/`notify()` for the Producer-Consumer?**

| Aspect | `BlockingQueue` | Manual `wait()`/`notify()` |
|--------|---------------|----------------------------|
| **Ease of use** | Very simple, built-in solution | Complex, error-prone |
| **Risk of bugs** | Low, well-tested implementation | High risk of deadlocks, spurious wakeups |
| **Features** | Queue-specific features | Can be customized for special cases |
| **Recommended for** | Most use cases | Edge cases with specific requirements |

### 📊 Performance & Testing (Q59–Q63)

**59. What causes concurrency performance bottlenecks?**

Bottlenecks include high lock contention, excessive context switching, false sharing, and poor memory locality. Profile your code to identify these issues rather than assuming.

**60. How do you measure and monitor thread contention in Java?**

Use tools like JConsole, VisualVM, or Java Flight Recorder. Look for threads in `BLOCKED` state, high lock wait times, or high CPU utilization from context switching.

**61. What is context switching overhead, and how does it affect performance?**

Context switching involves saving a thread's state and loading another. This overhead increases with more active threads than CPU cores. Minimizing synchronization and using appropriate thread pool sizes reduces needless switching.

**62. How can you test multithreaded code effectively?**

Use tools like JCStress for concurrency testing, along with unit tests running many iterations. Incorporate stress tests and runtime analysis to detect race conditions.

**63. What is the `Thread.holdsLock()` method useful for?**

`Thread.holdsLock()` returns `true` if the current thread holds the monitor lock on a given object, which is useful for assertions and debugging.

### 🎨 Patterns & Best Practices (Q64–Q67)

**64. Why is `SimpleDateFormat` not thread-safe, and how do you handle it?**

`SimpleDateFormat` stores state in instance variables, causing data corruption when multiple threads access the same instance. Use `ThreadLocal`, create new instances for each operation (`DateTimeFormatter` in Java 8+ is immutable and thread-safe), or synchronize access.

**65. How do you implement thread-safe lazy initialization in Java?**

Use the **Initialization-on-demand holder idiom**:

```java
public class Singleton {
    private Singleton() {}
    private static class LazyHolder {
        static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE; // Safe with JVM guarantees
    }
}
```

**66. How can you ensure that a class is immutable and thread-safe?**

Make all fields `final`, initialize them in a constructor, and avoid providing setter methods. Ensure mutable fields are defensively copied or returned as unmodifiable views.

**67. What strategies can you use to avoid deadlocks in your code?**

- Consistent lock ordering across all code paths
- Use `tryLock()` with timeouts
- Avoid nested locks when possible
- Use higher-level concurrency constructs
- Apply deadlock detection tools during development

### 🔍 Comparing Concurrency Mechanisms (Q68–Q72)

**68. What are the differences between `synchronized`, `ReentrantLock`, and `Semaphore`?**

| Feature | `synchronized` | `ReentrantLock` | `Semaphore` |
|---------|--------------|----------------|-------------|
| **Acquisition** | Implicit | `lock()`/`tryLock()` | `acquire()` |
| **Release** | Automatic at block end | Manual `unlock()` | `release()` |
| **Fairness** | Not supported | Configurable | Configurable |
| **Interruptible** | Not directly | Yes (lockInterruptibly) | Yes |
| **Timeout** | No | Yes (tryLock timeout) | No, but tryAcquire |

**69. `Runnable` vs `Callable` – which to use and when?**

**Use `Runnable`** when your task doesn't need to return a result. **Use `Callable`** when you need a return value or want to throw checked exceptions.

```java
Runnable task = () -> System.out.println("No return value");
Callable<Integer> callable = () -> { return 42; };
```

**70. What are the differences between `Future`, `CompletableFuture`, and `ListenableFuture`?**

- `Future`: Basic, with simple `get()` blocking
- `CompletableFuture`: Rich functional composition without blocking
- `ListenableFuture` (Guava): Allows adding callbacks

**71. What is the difference between `Thread.sleep()`, `TimeUnit.sleep()`, and `LockSupport.parkNanos()`?**

- `Thread.sleep()`: Static method on `Thread`
- `TimeUnit.sleep()`: A more readable alternative
- `LockSupport.parkNanos()`: More flexible, used by many concurrency classes

**72. What's the difference between `CopyOnWriteArrayList` and `Collections.synchronizedList(new ArrayList<>())`?**

| Aspect | `CopyOnWriteArrayList` | `synchronizedList` |
|--------|------------------------|---------------------|
| **Read performance** | Very good (no locking) | Moderate (requires lock) |
| **Write performance** | Poor (copies array) | Moderate (acquires lock) |
| **Iterators** | Fail-safe | Fail-fast |
| **Best for** | Read-heavy scenarios | Balanced scenarios |

### ⚙️ Background & Utility Threads (Q73–Q80)

**73. What is the difference between a `Thread` and `run()` vs `start()`?**

`start()` creates a new system thread that executes the `run()` method. Calling `run()` directly executes the method in the current thread without creating a new thread.

**74. How do you handle uncaught exceptions in a thread?**

Set an `UncaughtExceptionHandler`:

```java
Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    System.err.println("Thread " + t.getName() + " threw: " + e);
});
```

**75. What is the `ThreadGroup` class used for?**

`ThreadGroup` organizes threads into groups for easier management (e.g., interrupting all threads in a group with a single call).

**76. When would you extend `Thread` vs implement `Runnable`?**

Implement `Runnable` is preferred because Java doesn't support multiple inheritance. Extend `Thread` if you need to override `Thread` methods. Use `Runnable` for better design and flexibility.

**77. What is a `Timer` and `TimerTask`? What are their limitations?**

`Timer` and `TimerTask` are legacy scheduling tools. They have limitations like single-threaded execution, which can cause delays, and lack of exception handling. Use `ScheduledExecutorService` instead.

**78. What is the `Phaser` class used for?**

`Phaser` is a more flexible synchronization barrier that supports dynamic registration of parties and phased tasks.

**79. What is the `Exchanger` class, and when would you use it?**

`Exchanger` allows two threads to swap data at a meeting point, useful for pipeline-style designs or producer-consumer with paired exchanges.

**80. What is the `ThreadLocalRandom` class, and why is it better than `Random` for multithreading?**

`ThreadLocalRandom` avoids performance degradation from contention on a single `Random` instance by giving each thread its own random generator.

### 🏗️ Framework & Environment Integration (Q81–Q84)

**81. How do you manage thread pools in a Spring application?**

Use Spring's `ThreadPoolTaskExecutor`, which wraps `ThreadPoolExecutor` and integrates well with Spring's lifecycle.

**82. How does Tomcat handle concurrent requests using its thread pool?**

Tomcat maintains a thread pool that assigns incoming requests to available threads. It's configured with parameters like `maxThreads` and `minSpareThreads` to manage concurrency.

**83. How do you handle concurrency in a web application context?**

Use request-scoped beans, `ThreadLocal` carefully, and ensure your controllers are stateless. For shared state, use concurrent collections or proper synchronization.

**84. How does the Java Memory Model (JMM) affect visibility in a distributed system?**

JMM only guarantees visibility within a single JVM. For distributed systems, you need additional coordination like message queues or shared databases.

### 🧬 Advanced `java.util.concurrent` Utilities (Q85–Q90)

**85. How does `CompletableFuture` provide better asynchronous programming?**

`CompletableFuture` composes multiple asynchronous operations without blocking, using methods like `thenApply`, `thenCompose`, and `thenCombine`, which enable complex asynchronous pipelines.

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenAccept(System.out::println);
```

**86. What is the difference between `CountDownLatch` and `CyclicBarrier`?**

- `CountDownLatch`: Used when threads must wait for others to complete a set of operations
- `CyclicBarrier`: Used when threads must wait for each other to reach a common point before proceeding; it's reusable

**87. How do `ThreadLocal` and `InheritableThreadLocal` differ?**

`InheritableThreadLocal` automatically propagates values to child threads created from a parent thread, while `ThreadLocal` doesn't.

**88. What is the `StampedLock` and how does it improve upon `ReentrantReadWriteLock`?**

`StampedLock` provides optimistic reading, which can improve performance when reads frequently don't conflict with writes. It also supports converting read to write locks.

**89. How does `LongAdder` perform better than `AtomicLong` under high contention?**

`LongAdder` uses a distributed approach where increments happen on local cells and are summed on-demand, reducing contention on a single atomic variable.

**90. What is the `ForkJoinPool.commonPool()`?**

`ForkJoinPool.commonPool()` returns a shared pool used by `CompletableFuture` and parallel streams when no custom pool is provided.

### 📚 In-Depth API & Usage Questions (Q91–Q100)

**91. What are the different implementations of `BlockingQueue` and their ideal use cases?**

- **ArrayBlockingQueue**: Bounded, efficient but fixed capacity
- **LinkedBlockingQueue**: Optionally bounded, better with high throughput
- **PriorityBlockingQueue**: Unbounded, elements ordered by priority
- **SynchronousQueue**: Handoff queue where each insert must wait for a take

**92. How do you implement a blocking queue using `ReentrantLock` and `Condition`?**

```java
public class CustomBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    
    public CustomBlockingQueue(int capacity) {
        this.capacity = capacity;
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.add(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.remove();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**93. What is the difference between a fair and unfair lock in `ReentrantLock`?**

A fair lock grants access to the longest-waiting thread, preventing starvation but potentially reducing throughput. Unfair locks can "barge," allowing newer threads to acquire the lock quickly, improving performance at the risk of starvation.

**94. How do you implement a thread-safe bounded buffer using `Semaphore`?**

See the `SemaphoreBuffer` example in Q57, which uses three semaphores: `emptySlots`, `fullSlots`, and `mutex`.

**95. What happens to a thread when it calls `wait()` on a monitor it doesn't own?**

An `IllegalMonitorStateException` is thrown because `wait()`, `notify()`, and `notifyAll()` can only be called from within a synchronized context on the monitor object.

**96. How does `Thread.yield()` work, and when is it useful?**

`Thread.yield()` hints to the scheduler that the current thread is willing to give up its CPU time. It's useful for debugging or improving responsiveness in cooperative multitasking designs.

**97. What is the `atomic` package, and what classes does it contain?**

`java.util.concurrent.atomic` provides atomic versions of primitive types (`AtomicInteger`, `AtomicBoolean`), arrays, references (`AtomicReference`), field updaters, and accumulators (`LongAdder`).

**98. How do you create a thread-safe singleton using an enum?**

```java
public enum Singleton {
    INSTANCE; // JVM guarantees single instance
    public void doSomething() { ... }
}
```

**99. What are the possible ways to handle checked exceptions in a `Runnable`?**

Wrap the checked exception in an unchecked exception like `RuntimeException`, or use a `Callable` instead, which can throw checked exceptions directly.

**100. How would you implement a simple thread pool without using `java.util.concurrent`?**

Here's a simplified hand-rolled thread pool:

```java
public class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue;
    private final List<Worker> workers = new ArrayList<>();
    private volatile boolean running = true;
    
    public SimpleThreadPool(int poolSize, int queueSize) {
        taskQueue = new LinkedBlockingQueue<>(queueSize);
        for (int i = 0; i < poolSize; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            worker.start();
        }
    }
    
    public void submit(Runnable task) {
        if (!running) throw new IllegalStateException("Pool is shutting down");
        try {
            taskQueue.put(task);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void shutdown() {
        running = false;
        workers.forEach(Thread::interrupt);
    }
    
    private class Worker extends Thread {
        @Override
        public void run() {
            while (running) {
                try {
                    Runnable task = taskQueue.take();
                    task.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
```

### 💎 Final Thoughts

Concurrency is a core skill for professional Java developers, and this list covers the topics most frequently encountered in both daily engineering work and technical interviews. The best way to ensure these concepts are truly second nature is by writing small sample programs that test them in action. If there's any topic you'd like me to elaborate on further—perhaps with more detailed code or an alternative explanation—please feel free to ask.