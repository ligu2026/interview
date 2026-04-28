# Top 80 Tricky & Pitfall Java Interview Questions
## Collections, Generics, Concurrency & Thread Safety (with Answers & Examples)

---

### Collections – General & Common Pitfalls

1. **What happens if you add an element to a `List` while iterating over it with a `for-each` loop?**  
   *Answer*: Throws `ConcurrentModificationException` because the iterator detects structural modification.  
   *Example*:  
   ```java
   List<String> list = new ArrayList<>(List.of("A", "B"));
   for (String s : list) {
       if (s.equals("A")) list.remove(s); // throws ConcurrentModificationException
   }
   ```

2. **Is `ArrayList` thread-safe? What is the correct way to make it thread-safe?**  
   *Answer*: No. Use `Collections.synchronizedList(new ArrayList<>())` or `CopyOnWriteArrayList`.  
   *Example*:  
   ```java
   List<String> syncList = Collections.synchronizedList(new ArrayList<>());
   ```

3. **What is the difference between `Iterator` and `ListIterator`?**  
   *Answer*: `ListIterator` allows bidirectional traversal, element replacement, and adding elements; `Iterator` only forward and remove.

4. **Why does `Set` not allow duplicates? How does it check equality?**  
   *Answer*: Uses `hashCode()` and `equals()`. If you override only `equals()`, duplicates may appear.

5. **What is the initial capacity of `HashMap`? When does it resize?**  
   *Answer*: Default initial capacity 16, load factor 0.75. Resizes when size > capacity * load factor (threshold).

6. **What is the problem with `HashMap` in Java 7 and earlier when multiple threads put elements?**  
   *Answer*: In Java 7, resizing could cause an infinite loop due to circular linked list in the bucket. Fixed in Java 8.

7. **What is the difference between `IdentityHashMap` and `HashMap`?**  
   *Answer*: `IdentityHashMap` uses reference equality (`==`) instead of `equals()` for keys.

8. **Why does `TreeSet` throw `ClassCastException` for elements not implementing `Comparable`?**  
   *Answer*: `TreeSet` stores elements in sorted order; requires natural ordering or a `Comparator`.  
   *Example*:  
   ```java
   Set<Object> set = new TreeSet<>();
   set.add(1);    // OK
   set.add("a");  // ClassCastException: Integer vs String
   ```

9. **What is the performance of `ArrayList` vs `LinkedList` for removing an element in the middle?**  
   *Answer*: `ArrayList` – O(n) due to shifting; `LinkedList` – O(n) due to search, but removal is O(1) once found.

10. **Can you store `null` keys in a `HashMap`? In a `Hashtable`?**  
    *Answer*: `HashMap` allows one `null` key; `Hashtable` does not allow any `null` key or value (throws NPE).

11. **What is the difference between `fail-fast` and `fail-safe` iterators?**  
    *Answer*: Fail-fast throw `ConcurrentModificationException` (e.g., `ArrayList` iterator). Fail-safe work on a clone (e.g., `CopyOnWriteArrayList` iterator).

12. **Why does `LinkedHashMap` maintain insertion order? How to make it maintain access order?**  
    *Answer*: Uses a doubly linked list. Access order via constructor `new LinkedHashMap<>(initialCapacity, loadFactor, true)`.

13. **What happens when you call `remove()` on a `Set` that is backed by a `Map` (e.g., `HashSet`)?**  
    *Answer*: It removes the mapping from the underlying map. Works fine.

14. **What is the difference between `poll()` and `remove()` in `Queue`?**  
    *Answer*: `poll()` returns `null` if empty; `remove()` throws `NoSuchElementException`.

15. **Why is it bad to use `LinkedList` with `get(index)` in a loop?**  
    *Answer*: `LinkedList.get(index)` is O(n). Loop becomes O(n²).  
    *Example*:  
    ```java
    LinkedList<Integer> list = new LinkedList<>();
    for (int i = 0; i < list.size(); i++) { int x = list.get(i); } // O(n²)
    ```

---

### Generics – Type Safety & Pitfalls

16. **What is type erasure? How does it affect method overriding?**  
    *Answer*: Generic type parameters are removed at compile time. You cannot overload a method based on generic type parameters alone.  
    *Example*:  
    ```java
    void print(List<String> list) {} // error: same erasure as List<Integer>
    void print(List<Integer> list) {}
    ```

17. **Why can’t you create a generic array like `T[] array = new T[10]`?**  
    *Answer*: Because arrays are reified (type known at runtime), generics are erased. Leads to `ClassCastException` risk. Use `List<T>` instead.

18. **What is the difference between `List<? extends Number>` and `List<Number>`?**  
    *Answer*: `List<? extends Number>` can hold any subtype of `Number` but you cannot add elements (except `null`). `List<Number>` can hold any `Number` and you can add.

19. **What is the PECS rule (Producer Extends, Consumer Super)?**  
    *Answer*: Use `? extends T` when you **produce** (read) items; use `? super T` when you **consume** (write) items.  
    *Example*:  
    ```java
    void copy(List<? extends T> src, List<? super T> dest) { ... }
    ```

20. **Why does `List<Object>` not accept `List<String>` even though `String` is subtype of `Object`?**  
    *Answer*: Generics are **invariant**. `List<String>` is not a subtype of `List<Object>`. Use wildcards: `List<? extends Object>`.

21. **What is a raw type? What happens when you mix raw types with generics?**  
    *Answer*: A raw type is a generic class used without type arguments. Causes loss of type safety, may lead to `ClassCastException` at runtime.  
    *Example*:  
    ```java
    List list = new ArrayList<String>(); // raw
    list.add(1); // no compile error, but later cast fails
    ```

22. **Why can’t you use `instanceof` with generic types like `list instanceof List<String>`?**  
    *Answer*: Type erasure removes `<String>`, so only `list instanceof List` is allowed.  
    *Workaround*:  
    ```java
    if (list instanceof List<?>) { ... }
    ```

23. **What is a generic method? How does type inference work?**  
    *Answer*: A method that declares its own type parameters before return type. Example:  
    ```java
    public static <T> T getFirst(List<T> list) { return list.get(0); }
    ```

24. **Why does `static <T> T[] toArray(T[] a)` in `Collection` not always work?**  
    *Answer*: If the passed array is too small, it creates a new array of the same component type via reflection; may fail at runtime if the runtime type isn't `T[]`.

25. **What is the difference between `List<Object>` and `List<?>`?**  
    *Answer*: `List<Object>` can hold any object and you can add any object. `List<?>` is an unknown type – you cannot add anything except `null`, only read elements as `Object`.

26. **Can you catch a generic exception? `catch (T e)`?**  
    *Answer*: No. Generic types are not allowed in catch clauses because they are erased.

27. **What is a bounded type parameter? Give example of multiple bounds.**  
    *Answer*: `T extends Number & Comparable<T>` – `T` must be subtype of `Number` and implement `Comparable`.  
    *Example*:  
    ```java
    public static <T extends Number & Comparable<T>> T max(T a, T b) { ... }
    ```

28. **Why does `new ArrayList<String>().getClass() == new ArrayList<Integer>().getClass()` return `true`?**  
    *Answer*: Type erasure – both return `ArrayList.class` at runtime.

29. **What is the `SuppressWarnings("unchecked")` annotation used for? When is it safe to use?**  
    *Answer*: To suppress unchecked cast warnings. Safe when you have type safety via logic (e.g., casting from a `List<?>` to `List<String>` after checking).  
    *Example*:  
    ```java
    @SuppressWarnings("unchecked")
    List<String> list = (List<String>) someList;
    ```

30. **Why can’t you have a generic exception class?**  
    *Answer*: Because the JVM cannot distinguish between `MyException<String>` and `MyException<Integer>` at runtime due to erasure, making catch blocks ambiguous.

---

### Concurrency – Thread Safety Basics

31. **What does the `volatile` keyword guarantee? What does it *not* guarantee?**  
    *Answer*: Visibility (writes are seen by other threads immediately). Does **not** guarantee atomicity (e.g., `volatile int i; i++` is not atomic).  
    *Example*:  
    ```java
    volatile boolean flag = false;
    // thread1: flag = true;
    // thread2: while (!flag) {} // will see flag change
    ```

32. **What is the difference between `synchronized` on a method vs a static method?**  
    *Answer*: Instance method locks on `this`. Static method locks on `Class` object.  
    *Example*:  
    ```java
    public synchronized void instanceMethod() {} // lock = this
    public static synchronized void staticMethod() {} // lock = MyClass.class
    ```

33. **What is a deadlock? Write a simple example.**  
    *Answer*: Two or more threads each waiting for a lock held by the other.  
    *Example*:  
    ```java
    Object lock1 = new Object(), lock2 = new Object();
    // Thread1: synchronized(lock1) { Thread.sleep(100); synchronized(lock2) {...}}
    // Thread2: synchronized(lock2) { Thread.sleep(100); synchronized(lock1) {...}}
    ```

34. **What is `ThreadLocal`? When would you use it?**  
    *Answer*: Provides thread-local variables, each thread gets its own copy. Used for `SimpleDateFormat` (not thread-safe) or transaction context.

35. **What is `happens-before` relationship in Java? Give an example.**  
    *Answer*: Guarantees ordering and visibility. Examples: unlock happens-before subsequent lock, `volatile` write happens-before read, thread start happens-before actions in that thread.

36. **What is the difference between `sleep()` and `wait()`?**  
    *Answer*: `sleep()` keeps locks, does not release; `wait()` releases the lock on the object, must be inside `synchronized`.  
    *Example*:  
    ```java
    synchronized (obj) { obj.wait(); } // releases lock
    Thread.sleep(1000); // holds lock
    ```

37. **Why is `Thread.stop()` deprecated?**  
    *Answer*: Unlocks all monitors, potentially leaving objects in inconsistent state. Use interruption instead.

38. **What is the difference between `notify()` and `notifyAll()`?**  
    *Answer*: `notify()` wakes one waiting thread (unknown which); `notifyAll()` wakes all. `notify()` can cause thread starvation if not used carefully.

39. **What is a `CountDownLatch`? Provide a use case.**  
    *Answer*: Allows one or more threads to await until a count reaches zero. Use case: wait for multiple services to start.  
    *Example*:  
    ```java
    CountDownLatch latch = new CountDownLatch(3);
    // each service calls latch.countDown()
    latch.await(); // wait for all 3
    ```

40. **What is the difference between `ConcurrentHashMap` and `Collections.synchronizedMap()`?**  
    *Answer*: `ConcurrentHashMap` allows concurrent reads without locking and locks only bucket segments (or bins) for writes; higher concurrency. `synchronizedMap` locks entire map on every operation.

---

### Concurrent Collections & Utilities

41. **What is `CopyOnWriteArrayList`? When is it useful?**  
    *Answer*: Thread-safe variant where all mutative operations create a new copy. Best for read-heavy scenarios with few writes.  
    *Pitfall*: multiple writes cause high overhead.

42. **Why does `ConcurrentHashMap` not allow `null` keys or values?**  
    *Answer*: Ambiguity – `null` could mean “key not present” or actual null value, but concurrency makes it impossible to distinguish reliably.  
    *Example*: `concurrentHashMap.put(null, "value")` → NPE.

43. **What is the difference between `ConcurrentHashMap` and `Hashtable`?**  
    *Answer*: `Hashtable` synchronizes whole map; `ConcurrentHashMap` uses lock striping (Java 7) or CAS + synchronized (Java 8+), higher concurrency.

44. **What is a `BlockingQueue`? Give an example of its methods.**  
    *Answer*: Queue that waits for the queue to become non-empty when retrieving (`take()`) or non-full when inserting (`put()`). Implementations: `ArrayBlockingQueue`, `LinkedBlockingQueue`.

45. **What is the difference between `ArrayBlockingQueue` and `LinkedBlockingQueue`?**  
    *Answer*: `ArrayBlockingQueue` bounded and fixed capacity; uses single lock. `LinkedBlockingQueue` optionally bounded, uses separate locks for put/take (higher throughput).

46. **What is `ThreadPoolExecutor`? Explain its core parameters.**  
    *Answer*: Executor service with configurable thread pool. Parameters: `corePoolSize`, `maximumPoolSize`, `keepAliveTime`, `workQueue`, `threadFactory`, `handler`.  
    *Pitfall*: unbounded queue + max threads = core size never exceeded? Actually tasks queue.

47. **What is the difference between `submit()` and `execute()` in `ExecutorService`?**  
    *Answer*: `execute()` returns void; `submit()` returns `Future<T>` to get result or catch exceptions.  
    *Example*:  
    ```java
    Future<?> future = executor.submit(() -> { throw new RuntimeException(); });
    future.get(); // throws ExecutionException
    ```

48. **What is a `ForkJoinPool`? How does work-stealing work?**  
    *Answer*: Designed for divide-and-conquer tasks. Each thread has a deque; idle threads steal tasks from others' deque tails.  
    *Example*: `RecursiveTask` for parallel sum.

49. **What is `CompletableFuture`? How does it handle exceptions?**  
    *Answer*: Asynchronous programming with chaining. Exception handling via `exceptionally()`, `handle()`.  
    *Example*:  
    ```java
    CompletableFuture.supplyAsync(() -> "Hello")
        .thenApply(String::toUpperCase)
        .exceptionally(ex -> "Fallback");
    ```

50. **What is `ReadWriteLock`? When would you use it?**  
    *Answer*: Separates read and write locks; multiple readers allowed, exclusive writer. Use when reads greatly outnumber writes.

---

### Thread Safety Pitfalls & Advanced Concurrency

51. **What is the double-checked locking problem? How is it fixed in Java?**  
    *Answer*: Incorrect lazy initialization without `volatile` could result in seeing partially constructed object. Fixed with `volatile` or static holder pattern.  
    *Example (correct)*:  
    ```java
    private static volatile MySingleton instance;
    public static MySingleton getInstance() {
        if (instance == null) {
            synchronized (MySingleton.class) {
                if (instance == null) instance = new MySingleton();
            }
        }
        return instance;
    }
    ```

52. **What is a `StampedLock`? How is it better than `ReadWriteLock`?**  
    *Answer*: Supports optimistic read locking. Allows non-blocking reads; if conflict occurs, upgrade to read lock.  
    *Example*:  
    ```java
    long stamp = lock.tryOptimisticRead();
    // read data
    if (!lock.validate(stamp)) { // conflict
        stamp = lock.readLock();
        // re-read
    }
    ```

53. **What is the problem with using `String` as a lock object?**  
    *Answer*: String interning (same literal may be same object across different contexts) can cause unexpected lock contention. Use `new Object()` as lock.

54. **What is a `Phaser`? How is it different from `CyclicBarrier`?**  
    *Answer*: `Phaser` allows dynamic registration of parties, can be reused, and has phases. `CyclicBarrier` has fixed party count.

55. **What are `Atomic` classes (e.g., `AtomicInteger`)? How do they achieve thread safety?**  
    *Answer*: Use CAS (Compare-And-Swap) via `Unsafe` class. Provide atomic operations like `incrementAndGet()` without locks.  
    *Example*:  
    ```java
    AtomicInteger counter = new AtomicInteger(0);
    counter.incrementAndGet(); // thread-safe
    ```

56. **What is the ABA problem in CAS? How does `AtomicStampedReference` solve it?**  
    *Answer*: ABA: value changes A→B→A, CAS sees A and incorrectly succeeds. `AtomicStampedReference` adds a stamp/version to detect changes.

57. **What is a `Semaphore`? Give a practical use.**  
    *Answer*: Controls number of concurrent accesses. Use for connection pool limiting to 10 connections.  
    *Example*:  
    ```java
    Semaphore semaphore = new Semaphore(10);
    semaphore.acquire(); // get permit
    // use resource
    semaphore.release();
    ```

58. **What is the difference between `locks` in `ReentrantLock` and `synchronized`?**  
    *Answer*: `ReentrantLock` offers tryLock with timeout, interrupts, fairness policy; `synchronized` is simpler but less flexible.

59. **What is thread starvation? How can it occur?**  
    *Answer*: Low-priority threads never get CPU because high-priority threads keep running. Can be mitigated using fair locks or proper priority design.

60. **How does `Thread.interrupt()` work? How should you handle `InterruptedException`?**  
    *Answer*: Sets interrupt flag. Blocking methods like `sleep()` throw `InterruptedException` and clear flag. You should either re-interrupt or propagate.  
    *Correct handling*:  
    ```java
    try { Thread.sleep(1000); } catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // restore flag
        return;
    }
    ```

---

### Concurrency – Java Memory Model & Visibility

61. **What is the JMM guarantee for `final` fields?**  
    *Answer*: Properly constructed object's `final` fields are visible to all threads after constructor finishes, without synchronization.

62. **What is the difference between `synchronized` and `volatile` for visibility?**  
    *Answer*: Both provide visibility, but `synchronized` also provides mutual exclusion and atomicity.

63. **What is the “out-of-thin-air” safety problem?**  
    *Answer*: A compile-time reordering could make a non-volatile `long` or `double` read appear as intermediate value; JVM guarantees atomic reads for 32-bit but not 64-bit without `volatile`.

64. **What is a memory barrier? When does the JVM insert them?**  
    *Answer*: CPU instruction that prevents reordering. JVM inserts them around `volatile` reads/writes, `synchronized` enter/exit.

65. **Why can a non-volatile variable’s update be unseen by another thread even after a write?**  
    *Answer*: Java allows caching and reordering; without happens-before, threads may see stale values.

---

### Collections & Concurrency Mixed Pitfalls

66. **What happens if you use `HashMap` in a multi-threaded environment without synchronization?**  
    *Answer*: May cause infinite loop (Java 7), lost updates, or `ConcurrentModificationException`. Use `ConcurrentHashMap`.

67. **Is `Vector` fully thread-safe? What is its downside?**  
    *Answer*: Methods are synchronized, but compound actions (iterate + remove) still need external sync. Downside: performance overhead even when not needed.

68. **What is the difference between `Collections.synchronizedList` and `Vector`?**  
    *Answer*: Both synchronize methods, but `Vector` is legacy; `synchronizedList` allows you to wrap any `List`. Both need external sync for iteration.

69. **Why does `ConcurrentLinkedQueue` not have `size()` O(1)?**  
    *Answer*: It's lock-free; maintaining exact size would require costly synchronization. `size()` is O(n) and not accurate under concurrency.

70. **What is a `TransferQueue`? How does it differ from `BlockingQueue`?**  
    *Answer*: `TransferQueue` allows producer to wait for consumer to receive element (blocking transfer). Implementation: `LinkedTransferQueue`.

71. **What is the problem with using `HashSet` with multiple threads?**  
    *Answer*: Not thread-safe. Concurrent modifications may cause corrupted internal structure, infinite loops, or lost elements. Use `ConcurrentHashMap.newKeySet()`.

72. **What is the difference between `ConcurrentSkipListMap` and `ConcurrentHashMap`?**  
    *Answer*: `ConcurrentSkipListMap` maintains sorted order (like `TreeMap`), implements `NavigableMap`. `ConcurrentHashMap` does not sort.

73. **What is `Exchanger`? Provide a scenario.**  
    *Answer*: Allows two threads to exchange objects at a rendezvous point. Example: producer-consumer with direct handoff.

74. **Can you use `ArrayList` with `Collections.synchronizedList` and still get `ConcurrentModificationException`?**  
    *Answer*: Yes, if you iterate without manual synchronization over the synchronized list.  
    *Example*:  
    ```java
    List<String> list = Collections.synchronizedList(new ArrayList<>());
    // Must manually synchronize during iteration:
    synchronized(list) { for (String s : list) {...} }
    ```

75. **What is `LongAdder` vs `AtomicLong`? Which one should you use for high contention?**  
    *Answer*: `LongAdder` maintains multiple internal cells to reduce contention; better for high-write, low-read scenarios. `AtomicLong` is simpler.

---

### Generics & Collections – Advanced Pitfalls

76. **Why does `Arrays.asList(int[])` return a list of size 1?**  
    *Answer*: Because `int[]` is treated as a single `Object`; the array's elements are not auto-boxed. Use `Integer[]` or `IntStream.of(...).boxed().collect(...)`.

77. **What is the difference between `Collection.emptyList()` and `Collections.EMPTY_LIST`?**  
    *Answer*: `emptyList()` is type-safe (returns `List<T>`); `EMPTY_LIST` is raw type (unchecked assignment warning). Prefer `emptyList()`.

78. **What is a `Spliterator`? How is it used in parallel streams?**  
    *Answer*: Splittable iterator for parallel traversal. Used by streams to partition data.  
    *Pitfall*: Custom spliterator must be thread-safe or immutable.

79. **Why does `TreeMap` with comparator that is inconsistent with `equals` cause weird behavior?**  
    *Answer*: `TreeMap` uses comparator for ordering and equality. If `compare(a,b)==0` but `!a.equals(b)`, the map will treat them as same key.  
    *Example*:  
    ```java
    TreeMap<String, String> map = new TreeMap<>((a,b) -> 0); // all keys equal
    map.put("a", "1");
    map.put("b", "2"); // overwrites "a"
    System.out.println(map); // {a=2}
    ```

80. **What is the difference between `removeIf` on `ArrayList` and `CopyOnWriteArrayList` in a concurrent environment?**  
    *Answer*: `ArrayList.removeIf` may throw `ConcurrentModificationException` if modified concurrently. `CopyOnWriteArrayList.removeIf` uses a snapshot, no exception but may miss concurrently added elements.

---

These 80 questions cover the most common traps in collections, generics, concurrency, and thread safety. Use them to prepare for interviews or to deep-dive into Java’s tricky areas.