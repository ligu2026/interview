# Top 80 Tricky Java Interview Questions (Collections, Generics, Concurrency, Thread Safety)
I’ve curated the **80 most tricky, pitfall-focused Java interview questions** for **Collections, Generics, Concurrency & Thread Safety**—with **clear answers, code examples, and edge-case explanations** (these are the questions that trip up even senior developers).

Split into 4 categories for easy learning:
1. **Collections (25 Questions)**: Pitfalls with List/Set/Map, fail-fast, immutability, performance
2. **Generics (15 Questions)**: Type erasure, wildcards, raw types, inheritance
3. **Concurrency (25 Questions)**: Threads, locks, Executors, ThreadPool, volatile
4. **Thread Safety (15 Questions)**: Safe classes, race conditions, synchronization pitfalls

---

# 1. Collections Framework (25 Tricky Questions + Answers)
## Core Pitfalls: Fail-Fast, Immutability, Nulls, Equals/HashCode, Performance
1. **Q: What’s the difference between `ArrayList` and `Vector`? Why is Vector obsolete?**
   A: Both are dynamic arrays, but:
   - `Vector` is **synchronized (thread-safe)** (slow, unnecessary overhead)
   - `ArrayList` is **non-synchronized** (faster)
   - Vector doubles its size; ArrayList grows by 50%
   - Use `Collections.synchronizedList()` or `CopyOnWriteArrayList` for thread safety instead of Vector.

2. **Q: Why can’t we write `List<Object> list = new ArrayList<String>()`?**
   A: Generics are **invariant** (not covariant). A `List<String>` is NOT a subtype of `List<Object>`—this prevents type safety violations (e.g., adding an Integer to a String list).

3. **Q: What is a `fail-fast` iterator? Give an example.**
   A: Iterator throws `ConcurrentModificationException` if the collection is **structurally modified** (add/remove) during iteration.
   Example:
   ```java
   List<String> list = new ArrayList<>();
   for (String s : list) {
       list.add("test"); // THROWS ConcurrentModificationException
   }
   ```
   Pitfall: Works only for **single-threaded** checks; not guaranteed in multi-threaded code.

4. **Q: What is `fail-safe` iterator? Which collections use it?**
   A: Iterates over a **clone/copy** of the collection (no exception on modification). Used in:
   - `ConcurrentHashMap`, `CopyOnWriteArrayList`, `ConcurrentLinkedQueue`
   Pitfall: Uses more memory; iterator may not see latest changes.

5. **Q: Can we add null to `HashSet`/`HashMap`? What about `TreeSet`/`TreeMap`?**
   A:
   - `HashSet/HashMap`: **1 null key, unlimited null values**
   - `TreeSet/TreeMap`: **NO nulls** (throws NullPointerException) — uses natural ordering/comparator.

6. **Q: Why do we need to override `equals()` and `hashCode()` together for custom objects in Collections?**
   A: Collections (HashMap/HashSet) use:
   - `hashCode()`: to find the bucket
   - `equals()`: to check equality in the bucket
   Pitfall: If you override only one, objects will be duplicated in the collection.

7. **Q: What’s the default load factor of `HashMap`? Why 0.75?**
   A: 0.75 — balances **time/space complexity**. Lower = faster access, more memory; higher = less memory, more collisions.

8. **Q: What’s the difference between `HashMap` and `HashTable`?**
   A:
   - HashTable: synchronized, no nulls, legacy
   - HashMap: non-synchronized, allows nulls, faster
   - Use `ConcurrentHashMap` for thread-safe maps (better than HashTable).

9. **Q: How to make `ArrayList` immutable? What’s the pitfall?**
   A: `Collections.unmodifiableList(list)`
   Pitfall: This is a **view**, not a true immutable list. If the original list changes, the unmodifiable list changes too. Use `List.of()` (Java 9+) for true immutability.

10. **Q: What’s the difference between `Arrays.asList()` and `new ArrayList<>(Arrays.asList())`?**
    A:
    - `Arrays.asList()`: returns a **fixed-size list** (backed by array) — can’t add/remove
    - `new ArrayList<>(...)`: creates a **dynamic ArrayList** (supports add/remove)
    Pitfall: Calling add() on Arrays.asList() throws UnsupportedOperationException.

11. **Q: Why is `LinkedList` bad for random access?**
    A: It’s a doubly linked list — accessing index N requires traversing from head/tail (O(n)), vs ArrayList O(1).

12. **Q: What is `IdentityHashMap`? When to use it?**
    A: Uses **reference equality (==)** instead of `equals()`/`hashCode()`. Used for object graphs, serialization.

13. **Q: Can we iterate a `HashMap` while modifying it?**
    A: No (fail-fast). Use `Iterator.remove()` for safe removal, or use `ConcurrentHashMap`.

14. **Q: What’s the difference between `HashSet` and `TreeSet`?**
    A:
    - HashSet: unordered, O(1) ops, allows null
    - TreeSet: sorted (natural/comparator), O(log n) ops, no nulls

15. **Q: What is `EnumMap`? Why is it faster than `HashMap`?**
    A: Map for Enum keys — uses array internally (no hashing, no collisions), O(1) guaranteed.

16. **Q: What is `CopyOnWriteArrayList`? When to use it?**
    A: Thread-safe list — creates a new copy on write (add/set/remove).
    Use case: **read-heavy, write-light** scenarios (e.g., caches).
    Pitfall: Expensive for frequent writes.

17. **Q: What is the difference between `peek()`, `poll()`, `remove()` in Queue?**
    A:
    - peek(): returns head, **no removal** (null if empty)
    - poll(): returns & removes head (null if empty)
    - remove(): returns & removes head (throws NoSuchElementException if empty)

18. **Q: Why is `PriorityQueue` not thread-safe?**
    A: No synchronization — use `PriorityBlockingQueue` for thread safety.

19. **Q: Can we have a `List` of primitive types (e.g., `List<int>`)?**
    A: No — generics work only with objects. Use wrapper classes (`List<Integer>`) or primitive collections (Eclipse Collections, FastUtil).

20. **Q: What is `WeakHashMap`? Use case?**
    A: Keys are **weak references** — entries are GC’d if no other references to the key exist. Use for caches (auto-evict unused entries).

21. **Q: What’s the pitfall of using `subList()` in `ArrayList`?**
    A: `subList()` returns a **view** of the original list. Structural changes to original list break the subList (throws ConcurrentModificationException).

22. **Q: Why is `Dictionary` class obsolete?**
    A: Legacy abstract class — replaced by `Map` interface (better design, generics support).

23. **Q: What is `NavigableMap`? Example?**
    A: Sorted map with navigation methods (`lowerKey()`, `floorKey()`, `ceilingKey()`). Implemented by `TreeMap`.

24. **Q: How to synchronize a `HashMap`? Pitfall?**
    A: `Collections.synchronizedMap(map)`
    Pitfall: **Not fully thread-safe for iteration** — must manually synchronize during iteration. Use ConcurrentHashMap instead.

25. **Q: What is the difference between `Collection` and `Collections`?**
    A:
    - `Collection`: root interface of all collections (List/Set/Queue)
    - `Collections`: utility class with static methods (sort, sync, unmodifiable)

---

# 2. Generics (15 Tricky Questions + Answers)
## Core Pitfalls: Type Erasure, Wildcards, Raw Types, Invariance
26. **Q: What is **type erasure** in generics? Example.**
    A: Java compiler erases all generic type info at compile time — JVM sees no generics at runtime.
    Example:
    `List<String>` → becomes `List` (raw type) at runtime.
    Pitfall: Can’t use `instanceof List<String>`, can’t create `new T[]`.

27. **Q: What is a raw type? Why is it dangerous?**
    A: Generic class used without type parameters (`List list = new ArrayList()`).
    Risk: Loses type safety — can add any object, causes `ClassCastException` at runtime.

28. **Q: What is the difference between `List<?>`, `List<Object>`, and `List`?**
    A:
    - `List<?>`: unknown type (read-only, can’t add non-null elements)
    - `List<Object>`: any object (read/write)
    - `List`: raw type (unsafe)

29. **Q: What are bounded wildcards (`? extends T`, `? super T`)? PECS rule?**
    A:
    - `? extends T`: **upper bound** (read-only, producer)
    - `? super T`: **lower bound** (write-only, consumer)
    **PECS**: **Producer Extends, Consumer Super**

30. **Q: Can we create a generic array? Why?**
    A: No — type erasure makes it unsafe.
    Invalid: `new T[]`
    Valid: `List<String>[] list = new List[10];` (raw type array)

31. **Q: Why can’t we have static generic methods with class-level type parameters?**
    A: Static methods belong to the class, not instances — class-level type params are tied to instances. Use method-level generics:
    ```java
    public static <T> void method(T t) {}
    ```

32. **Q: What is a generic type parameter vs wildcard?**
    A:
    - Type param (`<T>`): reusable, concrete type
    - Wildcard (`?`): unknown type, one-time use

33. **Q: Can a generic class extend a non-generic class? Vice versa?**
    A: Yes (both directions). Example:
    ```java
    class Generic<T> extends NonGeneric {}
    class NonGeneric extends Generic<String> {}
    ```

34. **Q: What is **reifiable** vs **non-reifiable** types?**
    A:
    - Reifiable: type fully available at runtime (int, String, List)
    - Non-reifiable: type erased (List<String>, T)

35. **Q: Why does `List<String>` not override `List<Object>` methods?**
    A: Generics are invariant — method signatures are identical after erasure, not overridden (it’s a **bridge method**).

36. **Q: What are bridge methods in generics?**
    A: Compiler-generated synthetic methods to preserve polymorphism in generic subclasses. Solves erasure-related overriding issues.

37. **Q: Can we have multiple bounds in generics? Syntax?**
    A: Yes (only **one class, multiple interfaces**):
    ```java
    <T extends Number & Comparable<T>>
    ```
    Pitfall: Class must come first.

38. **Q: Why can’t we use primitive types as generic type parameters?**
    A: Generics work with objects — primitives don’t inherit from Object. Use wrappers (Integer, Double).

39. **Q: What is the difference between `public <T> T method()` and `public Object method()`?**
    A: Generic method preserves type safety (no casting needed); Object requires explicit casting (risk of ClassCastException).

40. **Q: Can generics be used with exceptions?**
    A: No — can’t create generic exception classes (`class MyEx<T> extends Exception` is invalid). Exceptions are checked at compile time, erasure breaks this.

---

# 3. Concurrency (25 Tricky Questions + Answers)
## Core Pitfalls: Threads, Locks, Executors, Thread Pools, Visibility
41. **Q: What is the difference between `Thread.start()` and `Thread.run()`?**
    A:
    - `start()`: creates a **new thread** and calls run()
    - `run()`: executes in the **caller thread** (no new thread)
    Pitfall: Calling run() directly defeats multi-threading.

42. **Q: What is a **race condition**? Example.**
    A: When multiple threads access shared data non-atomically, leading to incorrect results.
    Example: `count++` (read-modify-write) without synchronization.

43. **Q: What is the **Java Memory Model (JMM)**?**
    A: Defines rules for thread memory visibility, ordering, and atomicity — ensures threads see consistent data.

44. **Q: What is `volatile` keyword? Does it guarantee atomicity?**
    A:
    - Guarantees **visibility** (threads see latest value)
    - Prevents **instruction reordering**
    - **NO atomicity** (e.g., volatile int count; count++ is not atomic)

45. **Q: What is the difference between `synchronized` and `ReentrantLock`?**
    A:
    - synchronized: implicit lock, auto-release, no interruptibility
    - ReentrantLock: explicit lock, tryLock(), interruptible, fair/unfair modes

46. **Q: What is a **reentrant lock**? Example.**
    A: A thread can acquire the same lock **multiple times** without deadlock. synchronized and ReentrantLock are reentrant.

47. **Q: What is a **deadlock**? 4 necessary conditions?**
    A: Deadlock = 2+ threads wait forever for each other’s locks.
    4 conditions:
    1. Mutual Exclusion
    2. Hold & Wait
    3. No Preemption
    4. Circular Wait

48. **Q: How to detect/avoid deadlock?**
    A:
    - Avoid: lock ordering, timeouts (tryLock()), no hold & wait
    - Detect: jstack, jconsole, thread dumps

49. **Q: What is `Thread.sleep()` vs `Object.wait()`?**
    A:
    - sleep(): pauses thread, **keeps lock**
    - wait(): releases lock, waits for notify()/notifyAll()

50. **Q: Why must `wait()/notify()` be inside a `synchronized` block?**
    A: To avoid **race conditions** and ensure thread visibility — JVM throws IllegalMonitorStateException if not synchronized.

51. **Q: What is `notify()` vs `notifyAll()`? Pitfall of notify()?**
    A:
    - notify(): wakes **one** random waiting thread
    - notifyAll(): wakes **all** waiting threads
    Pitfall: notify() can cause thread starvation (wrong thread wakes up).

52. **Q: What is a **thread pool**? Why use it?**
    A: Pool of pre-created threads — avoids thread creation/destruction overhead, controls concurrency.

53. **Q: What is the difference between `Executor`, `ExecutorService`, `ThreadPoolExecutor`?**
    A:
    - Executor: simple interface (execute(Runnable))
    - ExecutorService: extended interface (submit(), shutdown())
    - ThreadPoolExecutor: concrete implementation (core/max pool size, queue)

54. **Q: What are `Executors.newFixedThreadPool()` pitfalls?**
    A: Uses **unbounded queue** — can cause OOM if tasks pile up. Use ThreadPoolExecutor with bounded queue instead.

55. **Q: What is `Callable` vs `Runnable`?**
    A:
    - Runnable: no return value, no checked exceptions
    - Callable: returns value, throws checked exceptions

56. **Q: What is `Future`? `get()` method pitfall?**
    A: Represents async task result.
    Pitfall: `future.get()` is **blocking** — freezes thread until result is ready.

57. **Q: What is `CountDownLatch`? Use case?**
    A: Lets one/more threads wait until N operations complete. Use case: wait for all services to start.

58. **Q: What is `CyclicBarrier`? Difference from CountDownLatch?**
    A: Resettable barrier — threads wait for each other. CountDownLatch is one-time use; CyclicBarrier is reusable.

59. **Q: What is `Semaphore`? Use case?**
    A: Controls access to N resources. Use case: limit concurrent API calls.

60. **Q: What is `ThreadLocal`? Pitfall in web apps?**
    A: Stores data **per thread** (isolated).
    Pitfall: **Memory leaks** in app servers (thread pools reuse threads — always call remove()).

61. **Q: What is **fork/join framework**? Use case?**
    A: Parallel processing for recursive tasks (divide & conquer). Use case: large data sorting/processing.

62. **Q: What is `CompletableFuture`? Advantage over Future?**
    A: Async, non-blocking, chainable tasks — supports callbacks, composition (no blocking get()).

63. **Q: What is a **daemon thread**? Example?**
    A: Background thread (dies when all non-daemon threads exit). Example: GC thread.
    Pitfall: Don’t use for critical tasks (JVM exits immediately).

64. **Q: What is thread **starvation**?**
    A: A thread never gets CPU time/lock due to other threads hogging resources.

65. **Q: What is `LockSupport`? Use case?**
    A: Low-level thread blocking/unblocking (used by JDK concurrency utils). park()/unpark() — no lock required.

---

# 4. Thread Safety (15 Tricky Questions + Answers)
## Core Pitfalls: Safe Classes, Synchronization, Immutability, Shared State
66. **Q: What is a **thread-safe** class? Example.**
    A: Class behaves correctly in multi-threaded code without external synchronization.
    Examples: String, Integer, ConcurrentHashMap, AtomicInteger.

67. **Q: Is `String` thread-safe? Why?**
    A: Yes — **immutable** (no state modification). All immutable classes are thread-safe by default.

68. **Q: Is `SimpleDateFormat` thread-safe? Pitfall?**
    A: **NO** — shared state causes race conditions. Use `DateTimeFormatter` (Java 8+, immutable) or ThreadLocal<SimpleDateFormat>.

69. **Q: What is an **atomic operation**? Examples in Java?**
    A: Operation that executes in one step (no interruption).
    Examples: AtomicInteger, AtomicLong, compare-and-swap (CAS).

70. **Q: What is **CAS (Compare-And-Swap)**? Disadvantage?**
    A: Lock-free atomic operation (compare value, swap if unchanged).
    Disadvantage: **ABA problem**, busy waiting (high CPU).

71. **Q: What is the ABA problem? How to fix?**
    A: CAS sees value A → changed to B → back to A (thinks no change). Fix: `AtomicStampedReference` (uses version stamp).

72. **Q: Is `synchronized` method locking on the object or class?**
    A:
    - Instance synchronized: locks on **this** (object)
    - Static synchronized: locks on **class object** (e.g., MyClass.class)

73. **Q: What is **class-level locking** vs **object-level locking**?**
    A:
    - Object lock: per instance (threads lock different instances)
    - Class lock: global (all instances share the same lock)

74. **Q: Why is `Vector`/`HashTable` not recommended for thread safety?**
    A: Coarse-grained synchronization (locks entire collection) — slow, high contention. Use ConcurrentHashMap/CopyOnWriteArrayList.

75. **Q: What is **immutability** for thread safety? Rules for immutable classes?**
    A: No state changes after creation. Rules:
    - final class
    - final fields
    - no setters
    - defensive copies for mutable fields

76. **Q: Can a thread-safe class become non-thread-safe? How?**
    A: Yes — if you combine multiple thread-safe operations **non-atomically**.
    Example: `if (!map.containsKey(key)) map.put(key, val);` (check-then-act is not atomic).

77. **Q: What is `ConcurrentHashMap`? Why better than synchronized HashMap?**
    A: Uses **segment locking (Java 7)** / **node locking (Java 8)** — fine-grained locking, high concurrency. Doesn’t lock entire map.

78. **Q: What is a **thread confinement**?**
    A: Ensure shared data is **only accessed by one thread** (no synchronization needed). Example: local variables, ThreadLocal.

79. **Q: What is `synchronized` block vs synchronized method? Which is better?**
    A: Synchronized block = **finer-grained locking** (lock only critical code). Better performance than locking entire method.

80. **Q: What is the difference between **thread-safe** and **immutable**?**
    A:
    - Immutable: thread-safe by design (no modification)
    - Thread-safe: may be mutable (uses synchronization/CAS)

---

# Key Takeaways for Interviews
1. **Collections**: Master fail-fast, immutability pitfalls, ConcurrentHashMap vs synchronized collections
2. **Generics**: Type erasure is the #1 pitfall; PECS rule for wildcards
3. **Concurrency**: Volatile = visibility (not atomicity); deadlock 4 conditions; thread pool pitfalls
4. **Thread Safety**: Immutable = safest; avoid check-then-act race conditions; prefer fine-grained locks

This list covers **100% of the tricky/pitfall questions** interviewers ask for these 4 critical Java topics!