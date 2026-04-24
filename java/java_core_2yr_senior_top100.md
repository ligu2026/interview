# Top 100 Java Core Interview Questions & Answers
### For 2-Year & Senior Programmers

---

## 🟡 INTERMEDIATE — 2 Years Experience (Q1–Q50)

### OOP & Design

**Q1. What is the difference between composition and inheritance? When do you prefer one over the other?**
- **Inheritance** — "is-a" relationship. Subclass reuses parent behavior but creates tight coupling. Changes to parent ripple to all children.
- **Composition** — "has-a" relationship. A class delegates behavior to contained objects. More flexible, easier to change.

**Prefer composition** when:
- Behavior needs to change at runtime
- You want to avoid tight coupling
- Multiple behaviors are needed from different sources

```java
// Inheritance — tight coupling
class ElectricCar extends Car { ... }

// Composition — flexible
class Car {
    private Engine engine; // Engine can be swapped
    Car(Engine engine) { this.engine = engine; }
}
```

---

**Q2. Explain SOLID principles with Java examples.**

- **S — Single Responsibility**: A class should do one thing.
- **O — Open/Closed**: Open for extension, closed for modification.
- **L — Liskov Substitution**: Subclasses must be substitutable for their parent.
- **I — Interface Segregation**: Prefer specific interfaces over fat general ones.
- **D — Dependency Inversion**: Depend on abstractions, not concrete implementations.

```java
// D — Dependency Inversion
interface NotificationService { void send(String msg); }
class EmailNotification implements NotificationService { ... }
class UserService {
    private final NotificationService notifier; // depends on abstraction
    UserService(NotificationService notifier) { this.notifier = notifier; }
}
```

---

**Q3. What is the difference between abstract class and interface? When do you use each?**
| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| State | Can have instance fields | Only constants |
| Constructor | Yes | No |
| Inheritance | Single | Multiple |
| Access modifiers | Any | `public` (default) |
| Use when | Shared base implementation | Capability contract |

**Use abstract class** for shared state/behavior among related classes.
**Use interface** for defining capabilities across unrelated classes.

---

**Q4. What is method hiding vs method overriding?**
- **Overriding** — instance methods; resolved at runtime (dynamic dispatch).
- **Hiding** — static methods; resolved at compile time based on reference type.

```java
class Parent { static void greet() { System.out.println("Parent"); } }
class Child extends Parent { static void greet() { System.out.println("Child"); } }

Parent p = new Child();
p.greet(); // prints "Parent" — static, resolved at compile time
```

---

**Q5. What is covariant return type?**
A subclass can override a method and return a more specific type than the parent's return type.

```java
class Animal { Animal create() { return new Animal(); } }
class Dog extends Animal {
    @Override
    Dog create() { return new Dog(); } // covariant — Dog is a subtype of Animal
}
```

---

**Q6. What is the diamond problem and how does Java handle it?**
The diamond problem occurs when a class inherits from two classes that share a common ancestor. Java avoids this by not allowing multiple class inheritance. For interfaces with `default` methods, Java requires the implementing class to explicitly override the conflicting method.

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }
class C implements A, B {
    @Override
    public void hello() { A.super.hello(); } // must resolve explicitly
}
```

---

**Q7. What is the difference between early binding and late binding?**
- **Early binding (compile-time)** — method call resolved at compile time. Applies to `static`, `private`, `final` methods and overloaded methods.
- **Late binding (runtime)** — method call resolved at runtime via vtable lookup. Applies to overridden instance methods (polymorphism).

---

**Q8. What is object cloning? Difference between shallow and deep copy?**
`clone()` creates a copy of an object. Override it and implement `Cloneable`.

- **Shallow copy** — copies field values as-is; object references point to same objects.
- **Deep copy** — recursively copies all referenced objects.

```java
// Deep copy via copy constructor
class Address { String city; Address(Address a) { this.city = a.city; } }
class Person {
    String name; Address address;
    Person(Person p) { this.name = p.name; this.address = new Address(p.address); }
}
```

---

**Q9. What is an immutable class? How do you create one?**
An object whose state cannot be changed after creation.

Steps:
1. Declare class as `final`
2. Make all fields `private final`
3. No setters
4. Initialize all fields via constructor
5. Return copies of mutable fields in getters

```java
public final class Money {
    private final double amount;
    private final String currency;
    public Money(double amount, String currency) {
        this.amount = amount; this.currency = currency;
    }
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
}
```

---

**Q10. What is the `Comparable` vs `Comparator` interface? When to use each?**
- `Comparable` — natural ordering; implemented inside the class via `compareTo()`. One ordering only.
- `Comparator` — external/custom ordering; multiple comparators can exist for the same class.

```java
// Multiple orderings with Comparator
Comparator<Employee> byName = Comparator.comparing(Employee::getName);
Comparator<Employee> bySalary = Comparator.comparingDouble(Employee::getSalary).reversed();
employees.sort(byName.thenComparing(bySalary));
```

---

### Collections & Data Structures

**Q11. How does `HashMap` work internally? What happens during a collision?**
`HashMap` uses an array of `Node` buckets. The key's `hashCode()` is hashed and mapped to a bucket index. On collision (same bucket, different key), nodes are chained as a linked list. In Java 8+, when the chain length exceeds 8 and the map has ≥ 64 buckets, the list is converted to a **red-black tree** for O(log n) lookup.

---

**Q12. What is the load factor in `HashMap`? What is the default?**
Load factor determines when to resize. Default is `0.75`. When `size > capacity × loadFactor`, the map is resized (doubled) and all entries are rehashed. A lower load factor reduces collisions but wastes memory.

---

**Q13. What is the difference between `ConcurrentHashMap` and `Collections.synchronizedMap()`?**
| | `ConcurrentHashMap` | `synchronizedMap` |
|--|---------------------|-------------------|
| Locking | Per-bucket (node-level) | Entire map |
| Concurrent reads | Yes | No |
| `null` keys/values | Not allowed | Allowed |
| Performance | Much higher | Lower |
| Iteration | Weakly consistent | Throws `ConcurrentModificationException` |

---

**Q14. What is `LinkedHashMap` and when would you use it?**
A `HashMap` that maintains insertion order (or access order if configured). Use cases:
- LRU cache (access-order mode with `removeEldestEntry()` override)
- Preserving JSON field order
- Deterministic iteration

```java
// LRU Cache using LinkedHashMap
new LinkedHashMap<>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry<K,V> e) {
        return size() > MAX_CACHE_SIZE;
    }
};
```

---

**Q15. What is `TreeMap` and what are its guarantees?**
A `NavigableMap` backed by a red-black tree. Maintains keys in natural sorted order or by a provided `Comparator`. Guarantees O(log n) for `get`, `put`, `remove`. Useful for range queries: `subMap()`, `headMap()`, `tailMap()`, `floorKey()`, `ceilingKey()`.

---

**Q16. Explain `fail-fast` vs `fail-safe` iterators with examples.**
- **Fail-fast** (`ArrayList`, `HashMap`) — uses `modCount`; throws `ConcurrentModificationException` if modified during iteration.
- **Fail-safe** (`CopyOnWriteArrayList`, `ConcurrentHashMap`) — iterates over a snapshot; no exception but may not reflect latest state.

```java
// Fail-safe
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>(List.of("a","b","c"));
for (String s : list) {
    list.add("d"); // no exception
}
```

---

**Q17. What is `ArrayDeque` and why prefer it over `Stack`?**
`ArrayDeque` is a resizable array-based double-ended queue. It is faster than `Stack` (which extends `Vector` and is synchronized). Use `ArrayDeque` as both a stack (`push`/`pop`) and a queue (`offer`/`poll`).

---

**Q18. What is `PriorityQueue` and how does it work internally?**
A min-heap backed by an array. The smallest element (by natural order or comparator) is always at the head. `offer()` is O(log n). `poll()` is O(log n). `peek()` is O(1). Not thread-safe — use `PriorityBlockingQueue` for concurrency.

---

**Q19. What is `Collections.unmodifiableList()` vs `List.of()`?**
- `Collections.unmodifiableList()` — wraps a mutable list; the underlying list can still be modified through the original reference.
- `List.of()` (Java 9+) — truly immutable; throws `UnsupportedOperationException` on any mutation, including `set()`. Also does not allow `null` elements.

---

**Q20. What is the difference between `peek()`, `poll()`, and `remove()` in Queue?**
| Method | Empty queue behavior |
|--------|---------------------|
| `peek()` | Returns `null` |
| `poll()` | Returns `null` |
| `element()` | Throws `NoSuchElementException` |
| `remove()` | Throws `NoSuchElementException` |

---

### Generics & Functional Programming

**Q21. What is the PECS principle in generics?**
**Producer Extends, Consumer Super.**
- Use `? extends T` when the collection **produces** (you read from it).
- Use `? super T` when the collection **consumes** (you write to it).

```java
// Producer — read from source
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (T item : src) dest.add(item);
}
```

---

**Q22. What is type erasure and what are its implications?**
Generics are a compile-time feature — the compiler removes type parameters and replaces them with `Object` (or bounds). At runtime, `List<String>` and `List<Integer>` are the same `List`. This means:
- Cannot use `instanceof` with generic types: `obj instanceof List<String>` — illegal
- Cannot create generic arrays: `new T[]` — illegal
- Cannot create instances of type parameter: `new T()` — illegal

---

**Q23. What is a functional interface? Name built-in ones.**
An interface with exactly one abstract method. Key built-in ones in `java.util.function`:
| Interface | Method | Use |
|-----------|--------|-----|
| `Predicate<T>` | `test(T)` | Filter/condition |
| `Function<T,R>` | `apply(T)` | Transform |
| `Consumer<T>` | `accept(T)` | Side effect |
| `Supplier<T>` | `get()` | Factory/lazy value |
| `BiFunction<T,U,R>` | `apply(T,U)` | Two-input transform |
| `UnaryOperator<T>` | `apply(T)` | Same-type transform |

---

**Q24. What is the difference between `map()`, `flatMap()`, and `peek()` in streams?**
- `map(f)` — transforms each element (1-to-1), returns `Stream<R>`
- `flatMap(f)` — transforms each element to a stream, flattens (1-to-many)
- `peek(f)` — applies a `Consumer` for debugging without changing the stream; still lazy

```java
// flatMap
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));
List<Integer> flat = nested.stream().flatMap(Collection::stream).toList();
// [1, 2, 3, 4]
```

---

**Q25. What is `Collectors.groupingBy()` and `partitioningBy()`?**
- `groupingBy(classifier)` — groups elements into a `Map<K, List<V>>`
- `partitioningBy(predicate)` — splits into `Map<Boolean, List<T>>` (true/false groups)

```java
Map<String, List<Employee>> byDept =
    employees.stream().collect(Collectors.groupingBy(Employee::getDepartment));

Map<Boolean, List<Employee>> seniorSplit =
    employees.stream().collect(Collectors.partitioningBy(e -> e.getYears() >= 5));
```

---

**Q26. What is a method reference and its four types?**
A shorthand for a lambda that delegates to an existing method.
1. Static: `Integer::parseInt`
2. Instance on specific object: `str::startsWith`
3. Instance on arbitrary object: `String::toLowerCase`
4. Constructor: `ArrayList::new`

---

**Q27. What is `Optional`? How do you use it correctly?**
A container for a potentially absent value. Avoids `null` and makes absence explicit.

```java
// Good usage
Optional.ofNullable(user)
    .map(User::getEmail)
    .filter(e -> e.contains("@"))
    .orElse("no-email@default.com");

// Anti-patterns to avoid
opt.get();                          // throws if empty
if (opt.isPresent()) opt.get();    // just use ifPresent() or map()
Optional<Optional<T>> nested;      // never nest Optionals
```

---

**Q28. What is `Stream.reduce()` and when would you use it?**
A terminal operation that combines all elements into one result via a binary operator. Use when `collect()` doesn't apply — e.g., summing, multiplying, finding max.

```java
int product = IntStream.rangeClosed(1, 5).reduce(1, (a, b) -> a * b); // 120
Optional<Integer> max = Stream.of(3,1,4,1,5).reduce(Integer::max);
```

---

**Q29. What is `Predicate.and()`, `or()`, `negate()`?**
Combinators for composing predicates.

```java
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> combined = notEmpty.and(startsWithA);

list.stream().filter(combined).toList();
```

---

**Q30. What is `Comparator.comparing()` chaining?**
A fluent API for building multi-field comparators.

```java
list.sort(Comparator.comparing(Employee::getDept)
          .thenComparing(Employee::getName)
          .thenComparingDouble(Employee::getSalary).reversed());
```

---

### Exceptions & I/O

**Q31. What is the exception hierarchy in Java?**
```
Throwable
├── Error (JVM-level, unrecoverable: OutOfMemoryError, StackOverflowError)
└── Exception
    ├── Checked (IOException, SQLException, ClassNotFoundException)
    └── RuntimeException (Unchecked: NPE, IllegalArgument, ArrayIndexOutOfBounds)
```

---

**Q32. What is `try-with-resources`? Can you have multiple resources?**
Automatically closes `AutoCloseable` resources. Multiple resources are closed in **reverse** declaration order.

```java
try (
    InputStream in = new FileInputStream("a.txt");
    OutputStream out = new FileOutputStream("b.txt")
) {
    // both closed automatically; out closed first, then in
}
```

---

**Q33. What is exception chaining?**
Wrapping a lower-level exception in a higher-level one to preserve the original cause.

```java
try {
    loadConfig();
} catch (IOException e) {
    throw new ApplicationException("Config load failed", e); // chain
}
// getCause() returns the original IOException
```

---

**Q34. What is the difference between `final`, `finally`, and `finalize()`?**
- `final` — modifier: prevents reassignment, overriding, or subclassing
- `finally` — block: always executes after try/catch
- `finalize()` — deprecated method called by GC before object is collected; unreliable, avoid

---

**Q35. When should you create a custom exception?**
- When you need domain-specific error meaning
- When callers need to catch and handle your exception specifically
- To add contextual data (error codes, field names)
- Prefer unchecked (`RuntimeException`) for programming errors, checked for recoverable conditions

---

### String & Math

**Q36. How does `String.intern()` work?**
Moves a string to the String Pool and returns the pooled reference. Useful to reduce memory when many duplicate strings exist. Two interned equal strings share the same reference.

```java
String a = new String("hello").intern();
String b = "hello";
a == b; // true — both point to pool
```

---

**Q37. What is `String.format()` vs `MessageFormat` vs text blocks?**
- `String.format()` — printf-style formatting with `%s`, `%d`, `%f`
- `MessageFormat` — pattern-based with `{0}`, `{1}`; supports locale, plurals
- **Text blocks** (Java 15+) — multi-line strings without escape sequences

```java
String json = """
    {
        "name": "%s",
        "age": %d
    }
    """.formatted("Alice", 30);
```

---

**Q38. What are common `String` methods every developer must know?**
`charAt()`, `substring()`, `indexOf()`, `lastIndexOf()`, `contains()`, `startsWith()`, `endsWith()`, `replace()`, `replaceAll()`, `split()`, `trim()`, `strip()`, `toUpperCase()`, `toLowerCase()`, `isEmpty()`, `isBlank()`, `join()`, `formatted()`, `matches()`, `toCharArray()`, `valueOf()`

---

### Concurrency Basics

**Q39. What is the difference between `sleep()` and `wait()`?**
| | `Thread.sleep()` | `Object.wait()` |
|--|-----------------|-----------------|
| Lock released | No | Yes |
| Called from | Anywhere | `synchronized` block only |
| Woken by | Timeout | `notify()` / `notifyAll()` / timeout |
| Purpose | Pause execution | Inter-thread communication |

---

**Q40. What is a `ThreadLocal`?**
Provides a separate copy of a variable for each thread. Useful for per-thread state like database connections, user sessions, or formatters.

```java
ThreadLocal<SimpleDateFormat> formatter =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
// Each thread gets its own SimpleDateFormat instance
```

---

**Q41. What is `synchronized` block vs method?**
- `synchronized` method — locks on `this` (instance) or the Class object (static)
- `synchronized` block — locks on a specified object; more granular, reduces contention

```java
private final Object lock = new Object();
public void update() {
    synchronized(lock) { // finer-grained than synchronized method
        // critical section
    }
}
```

---

**Q42. What is `volatile` and when is it sufficient?**
Guarantees **visibility** — all threads see the latest written value. Does NOT guarantee **atomicity**. Sufficient for single-writer, multiple-reader flags. NOT sufficient for `count++` (read-modify-write) — use `AtomicInteger` instead.

---

**Q43. What is `Callable` and `Future`?**
`Callable<V>` is a task that returns a result and may throw checked exceptions. `Future<V>` represents the pending result of an async computation.

```java
ExecutorService es = Executors.newFixedThreadPool(2);
Future<Integer> future = es.submit(() -> expensiveComputation());
// Do other work...
int result = future.get(); // blocks until done
```

---

**Q44. What is `CountDownLatch` vs `CyclicBarrier`?**
| | `CountDownLatch` | `CyclicBarrier` |
|--|-----------------|-----------------|
| Reusable | No | Yes |
| Direction | One-time countdown | Parties wait for each other |
| Use case | Wait for N events | Synchronize N threads at a point |

---

**Q45. What is `Semaphore` in Java concurrency?**
Controls access to a resource with a limited number of permits. Useful for rate limiting or connection pools.

```java
Semaphore semaphore = new Semaphore(3); // max 3 concurrent
semaphore.acquire();
try { accessResource(); }
finally { semaphore.release(); }
```

---

### JVM & Memory

**Q46. What is the JVM memory structure?**
- **Heap** — objects and arrays; divided into Young Gen (Eden + Survivor) and Old Gen
- **Stack** — one per thread; stores frames (local vars, operand stack, method references)
- **Method Area / Metaspace** — class metadata, static fields, constants
- **PC Register** — current instruction pointer per thread
- **Native Method Stack** — for native (JNI) method calls

---

**Q47. What is the Young Generation and Old Generation in GC?**
- **Young Gen (Eden + S0 + S1)** — new objects allocated here; collected by Minor GC (fast)
- **Old Gen (Tenured)** — long-lived objects promoted from Young Gen; collected by Major/Full GC (slow)
- **GC Roots** — stack references, static fields, JNI references; live objects reachable from GC roots survive

---

**Q48. What causes `StackOverflowError` vs `OutOfMemoryError`?**
- `StackOverflowError` — call stack exceeds limit (e.g., infinite recursion)
- `OutOfMemoryError: Java heap space` — heap exhausted; too many live objects
- `OutOfMemoryError: Metaspace` — too many loaded classes; class loader leak

---

**Q49. What is the difference between `String` in heap vs pool?**
- String literals (`"hello"`) go to the String Pool in the heap and are interned automatically.
- `new String("hello")` creates a new heap object outside the pool.
- `==` comparing pooled literals returns `true`; comparing `new String` instances returns `false`.

---

**Q50. What are strong, soft, weak, and phantom references?**
| Type | GC Behavior | Use Case |
|------|------------|----------|
| Strong | Never collected while referenced | Normal references |
| `SoftReference` | Collected when memory low | Memory-sensitive caches |
| `WeakReference` | Collected on next GC | `WeakHashMap`, canonicalization |
| `PhantomReference` | Collected after finalization | Post-GC cleanup actions |

---

## 🔴 SENIOR LEVEL (Q51–Q100)

### Advanced Concurrency

**Q51. What is `CompletableFuture` and how does it differ from `Future`?**
`CompletableFuture` extends `Future` with a non-blocking, composable API.

```java
CompletableFuture.supplyAsync(() -> fetchUser(id))
    .thenApplyAsync(user -> enrichWithOrders(user))
    .thenCombine(fetchPreferences(id), (user, prefs) -> merge(user, prefs))
    .exceptionally(ex -> defaultUser())
    .thenAccept(result -> sendResponse(result));
```
Key advantage: no blocking `get()` required; chains execute on thread pool asynchronously.

---

**Q52. What is the ForkJoin framework?**
A framework for parallel divide-and-conquer tasks. `ForkJoinPool` manages a pool of worker threads with **work-stealing** — idle threads steal tasks from busy threads' queues. Parallel streams use the common ForkJoinPool internally.

```java
class SumTask extends RecursiveTask<Long> {
    protected Long compute() {
        if (size <= THRESHOLD) return computeDirectly();
        SumTask left = new SumTask(...); left.fork();
        SumTask right = new SumTask(...);
        return right.compute() + left.join();
    }
}
```

---

**Q53. What is `ReentrantLock` and how does it differ from `synchronized`?**
| Feature | `synchronized` | `ReentrantLock` |
|---------|---------------|-----------------|
| Interruptible | No | Yes (`lockInterruptibly()`) |
| Try lock | No | Yes (`tryLock()`) |
| Fairness | No | Optional (`new ReentrantLock(true)`) |
| Condition variables | One (`wait/notify`) | Multiple (`newCondition()`) |
| Explicit unlock | No | Required (use `finally`) |

---

**Q54. What is the `happens-before` relationship in the Java Memory Model?**
A guarantee that memory writes by one action are visible to another. Key rules:
- An unlock `happens-before` a subsequent lock on the same monitor
- A `volatile` write `happens-before` a subsequent `volatile` read of the same variable
- `Thread.start()` `happens-before` any action in the started thread
- All actions in a thread `happens-before` `Thread.join()` returns

---

**Q55. What is `StampedLock` and when to use it?**
A lock with three modes: **write**, **read**, and **optimistic read**. Optimistic read has no blocking — validate the stamp afterward. Use when reads vastly outnumber writes and you want maximum throughput.

```java
StampedLock lock = new StampedLock();
long stamp = lock.tryOptimisticRead();
double x = this.x, y = this.y;
if (!lock.validate(stamp)) {
    stamp = lock.readLock();
    try { x = this.x; y = this.y; }
    finally { lock.unlockRead(stamp); }
}
```

---

**Q56. What is a `Phaser`?**
A flexible synchronization barrier that supports multiple phases, dynamic registration/deregistration of parties, and hierarchical phasers. More powerful than `CyclicBarrier`.

---

**Q57. What is `BlockingQueue` and its implementations?**
A thread-safe queue that blocks on `put()` when full and on `take()` when empty. Implementations:
- `ArrayBlockingQueue` — bounded, array-backed
- `LinkedBlockingQueue` — optionally bounded, linked-node
- `PriorityBlockingQueue` — priority-ordered, unbounded
- `SynchronousQueue` — no capacity; direct hand-off between threads

---

**Q58. What is the actor model vs shared-memory concurrency?**
- **Shared-memory** (Java default) — threads communicate via shared state; requires locks/synchronization; prone to races and deadlocks.
- **Actor model** — entities communicate by message passing; no shared state; used in Akka, Vert.x, virtual threads (Project Loom).

---

**Q59. What are virtual threads (Project Loom, Java 21)?**
Lightweight threads managed by the JVM rather than the OS. Millions can exist concurrently. Blocking a virtual thread does not block the OS thread — it unmounts and the carrier thread is reused.

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> handleRequest()); // creates a virtual thread
}
```

---

**Q60. What is `StructuredTaskScope` (Java 21 preview)?**
A structured concurrency API ensuring subtasks do not outlive their scope, making concurrent code easier to reason about and cancellation automatic on failure.

---

### JVM Internals & Performance

**Q61. What is JIT compilation and how does it work?**
The JVM initially interprets bytecode. The JIT (Just-In-Time) compiler monitors hot methods (via profiling) and compiles them to native machine code. It applies optimizations: inlining, dead code elimination, loop unrolling, escape analysis.

---

**Q62. What is escape analysis?**
A JIT optimization that determines if an object's reference escapes the method. If not, the object can be:
- **Stack-allocated** instead of heap-allocated (no GC pressure)
- **Lock eliminated** if only one thread accesses it

---

**Q63. What is G1 GC and how does it work?**
Garbage First (G1) divides the heap into equal-sized regions (Eden, Survivor, Old, Humongous). It prioritizes collecting regions with the most garbage first. Predictable pause times via configurable targets (`-XX:MaxGCPauseMillis`). Default GC since Java 9.

---

**Q64. What is ZGC and when should you use it?**
A low-latency GC (Java 11+) with sub-millisecond pause times regardless of heap size (up to terabytes). Uses load barriers and colored pointers. Ideal for latency-sensitive systems. Trade-off: slightly higher CPU usage than G1.

---

**Q65. How do you tune JVM memory settings?**
```bash
-Xms512m             # initial heap size
-Xmx4g               # max heap size
-XX:NewRatio=3        # Old:Young ratio
-XX:SurvivorRatio=8   # Eden:Survivor ratio
-XX:MaxMetaspaceSize=256m
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

---

**Q66. What tools do you use for JVM profiling and diagnostics?**
- `jstack` — thread dump (deadlock detection)
- `jmap` — heap dump
- `jstat` — GC statistics
- `jconsole` / `VisualVM` — real-time monitoring
- **JFR (Java Flight Recorder)** — low-overhead production profiling
- **Async-profiler** — CPU/allocation/lock profiling

---

**Q67. What is a memory leak in Java? How do you detect and fix it?**
Memory leaks occur when objects are no longer needed but still referenced. Common causes:
- Static collections holding references
- Listeners/callbacks not deregistered
- ThreadLocal not removed
- Unclosed streams/connections

Detection: heap dumps + `jmap`, then analyze with Eclipse MAT or VisualVM.
Fix: `removeEventListener()`, `ThreadLocal.remove()`, try-with-resources.

---

**Q68. What is the difference between `finalize()` and `Cleaner` (Java 9+)?**
`finalize()` is deprecated — unpredictable timing, performance overhead, can resurrect objects. `java.lang.ref.Cleaner` provides a reliable, explicit cleanup mechanism using phantom references.

```java
Cleaner cleaner = Cleaner.create();
cleaner.register(resource, () -> releaseNativeMemory());
```

---

### Design Patterns & Architecture

**Q69. What is the difference between Factory Method and Abstract Factory?**
- **Factory Method** — one method in a subclass decides which object to create. One product family.
- **Abstract Factory** — an interface for creating families of related objects without specifying concrete classes. Multiple product families.

---

**Q70. What is the Decorator pattern and how is it used in Java?**
Dynamically adds behavior to an object by wrapping it. Java I/O streams use this extensively.

```java
InputStream in =
    new BufferedInputStream(
        new GZIPInputStream(
            new FileInputStream("data.gz")));
```

---

**Q71. What is the Strategy pattern?**
Defines a family of algorithms, encapsulates each, and makes them interchangeable. The client selects the algorithm at runtime.

```java
interface SortStrategy { void sort(int[] arr); }
class QuickSort implements SortStrategy { ... }
class MergeSort implements SortStrategy { ... }
class Sorter {
    private SortStrategy strategy;
    void setStrategy(SortStrategy s) { this.strategy = s; }
    void sort(int[] arr) { strategy.sort(arr); }
}
```

---

**Q72. What is the Observer pattern? How is it implemented in Java?**
Defines a one-to-many dependency: when one object changes state, all dependents are notified. Used in event systems, MVC, reactive streams.

```java
interface Observer { void update(Event e); }
class EventBus {
    private List<Observer> observers = new ArrayList<>();
    void subscribe(Observer o) { observers.add(o); }
    void publish(Event e) { observers.forEach(o -> o.update(e)); }
}
```

---

**Q73. What is the Proxy pattern? Types?**
Provides a surrogate for another object to control access.
- **Virtual proxy** — lazy initialization
- **Protection proxy** — access control
- **Remote proxy** — local representative of remote object (RMI)
- **Caching proxy** — caches results
- Java's `java.lang.reflect.Proxy` creates dynamic proxies; Spring AOP uses proxies for `@Transactional`, `@Cacheable`.

---

**Q74. What is the Command pattern?**
Encapsulates a request as an object, allowing parameterization, queuing, logging, and undo operations.

```java
interface Command { void execute(); void undo(); }
class DeleteCommand implements Command {
    void execute() { db.delete(entity); }
    void undo() { db.save(entity); }
}
```

---

**Q75. What is the difference between Singleton and static class?**
| | Singleton | Static class |
|--|-----------|--------------|
| Instance | One | No instance |
| Inheritance | Can extend/implement | Cannot |
| Lazy init | Yes | Class loading |
| Testable | Yes (mock via interface) | Harder to mock |
| State | Instance state | Only static state |

---

### Modern Java Features

**Q76. What are sealed classes and how do they work with pattern matching?**
Sealed classes restrict which classes can extend them. Combined with `switch` pattern matching, they enable exhaustive, type-safe dispatch.

```java
sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double w, double h) implements Shape {}

double area = switch (shape) {
    case Circle c    -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.w() * r.h();
    case Triangle t  -> 0.5 * t.base() * t.height();
};
```

---

**Q77. What are records and their limitations?**
Records are immutable data carriers (Java 16+). Auto-generated: constructor, accessors, `equals()`, `hashCode()`, `toString()`.

Limitations:
- Cannot extend another class (implicitly extends `Record`)
- All fields are `final`
- Cannot declare instance fields beyond the record components
- Cannot be `abstract`

---

**Q78. What is the difference between `Stream.toList()` (Java 16) and `Collectors.toList()`?**
- `Collectors.toList()` — returns a mutable `ArrayList`
- `Stream.toList()` — returns an **unmodifiable** list; more concise; null elements allowed

---

**Q79. What is `instanceof` pattern matching (Java 16+)?**
Combines type check and casting in one step, eliminating explicit cast.

```java
// Before
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
// After
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

---

**Q80. What are text blocks (Java 15+) and how do indentation and escape sequences work?**
Text blocks use `"""` delimiters. The incidental leading whitespace is determined by the position of the closing `"""`. `\` suppresses a newline; `\s` adds a trailing space.

```java
String html = """
        <html>
            <body>Hello</body>
        </html>
        """; // indentation stripped up to 8 spaces
```

---

**Q81. What is `switch` expression (Java 14+) vs `switch` statement?**
Switch expressions return a value, use `->` syntax, and are exhaustive (compiler enforces all cases). No fall-through.

```java
int numDays = switch (month) {
    case JANUARY, MARCH, MAY, JULY, AUGUST, OCTOBER, DECEMBER -> 31;
    case APRIL, JUNE, SEPTEMBER, NOVEMBER -> 30;
    case FEBRUARY -> year % 4 == 0 ? 29 : 28;
};
```

---

**Q82. What is `HttpClient` (Java 11+)?**
A modern, built-in HTTP client supporting HTTP/1.1, HTTP/2, WebSocket, sync/async requests, and reactive streams.

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Accept", "application/json")
    .GET().build();
HttpResponse<String> response =
    client.send(request, HttpResponse.BodyHandlers.ofString());
```

---

**Q83. What is `var` and what are its limitations?**
`var` infers local variable types at compile time. Cannot be used for:
- Method parameters
- Return types
- Fields
- `null` initializer — `var x = null` fails (no type to infer)
- Lambda parameters (except in Java 11 with explicit `(var x) -> ...`)

---

### Testing & Code Quality

**Q84. What is the difference between unit, integration, and end-to-end tests?**
| | Unit | Integration | E2E |
|--|------|-------------|-----|
| Scope | Single class/method | Multiple components | Full system |
| Speed | Milliseconds | Seconds | Minutes |
| Dependencies | Mocked | Real (DB, services) | Real (browser, API) |
| Tools | JUnit + Mockito | Spring Test, Testcontainers | Selenium, Playwright |

---

**Q85. What is Mockito and how does it work?**
A mocking framework that creates fake implementations of dependencies for unit testing.

```java
@Mock UserRepository repo;
@InjectMocks UserService service;

when(repo.findById(1L)).thenReturn(Optional.of(new User("Alice")));
User user = service.getUser(1L);
assertEquals("Alice", user.getName());
verify(repo, times(1)).findById(1L);
```

---

**Q86. What is the difference between `@Mock` and `@Spy`?**
- `@Mock` — creates a full mock; all methods return defaults unless stubbed.
- `@Spy` — wraps a real object; real methods called unless stubbed. Use to partially mock.

---

**Q87. What is TDD (Test-Driven Development)?**
Write failing tests first, then minimal code to pass, then refactor. Cycle: **Red → Green → Refactor**. Benefits: better design, fewer regressions, living documentation.

---

**Q88. What is code coverage and what percentage should you target?**
Measures which lines/branches are executed by tests. Tools: JaCoCo, IntelliJ. 80% is a common industry target, but coverage alone is not a quality metric — test quality matters more. Aim for 100% on critical business logic.

---

### Reflection, Annotations & Serialization

**Q89. What is reflection and what are its performance implications?**
Reflection allows runtime inspection and invocation of classes, methods, and fields. It bypasses compile-time checks and is slower than direct calls due to:
- Security checks each invocation
- Boxing of primitives
- JIT cannot inline reflective calls

Prefer reflection only in frameworks; avoid in hot paths.

---

**Q90. How do you create a custom annotation in Java?**
```java
@Retention(RetentionPolicy.RUNTIME)  // available at runtime
@Target(ElementType.METHOD)           // applicable to methods
public @interface Retry {
    int times() default 3;
    Class<? extends Exception>[] on() default {Exception.class};
}
```
Process at runtime via `method.getAnnotation(Retry.class)` and reflection.

---

**Q91. What is the difference between `RetentionPolicy.RUNTIME`, `CLASS`, and `SOURCE`?**
- `SOURCE` — available only in source code; discarded by compiler (e.g., `@Override`)
- `CLASS` — in `.class` file but not available at runtime (e.g., `@NonNull` for static analysis)
- `RUNTIME` — available at runtime via reflection (e.g., `@Transactional`, `@RequestMapping`)

---

**Q92. What is serialization? What are `serialVersionUID` and `transient`?**
Serialization converts an object to bytes (`ObjectOutputStream`). `serialVersionUID` is a version identifier — mismatch between serialized and current class throws `InvalidClassException`. `transient` marks fields to skip during serialization.

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private transient String password; // not serialized
}
```

---

### Architecture & Best Practices

**Q93. What is defensive programming in Java?**
Writing code that anticipates and handles incorrect inputs, unexpected states, and failures.

```java
public void transfer(Account from, Account to, BigDecimal amount) {
    Objects.requireNonNull(from, "source account required");
    Objects.requireNonNull(to, "destination account required");
    if (amount.compareTo(BigDecimal.ZERO) <= 0)
        throw new IllegalArgumentException("Amount must be positive");
    if (from.getBalance().compareTo(amount) < 0)
        throw new InsufficientFundsException("Insufficient balance");
    // proceed
}
```

---

**Q94. What is the difference between `BigDecimal` and `double` for monetary calculations?**
`double` uses binary floating-point — cannot represent 0.1 exactly, leading to rounding errors. **Always use `BigDecimal` for money.**

```java
double bad = 0.1 + 0.2; // 0.30000000000000004
BigDecimal good = new BigDecimal("0.1").add(new BigDecimal("0.2")); // 0.3

// Use string constructor, NOT double constructor
new BigDecimal(0.1); // WRONG — inherits float imprecision
new BigDecimal("0.1"); // CORRECT
```

---

**Q95. What is the principle of least surprise in API design?**
APIs should behave in a way that developers expect based on their name, signature, and documentation. Surprises cause bugs.
- Method named `getUser()` should not modify state
- `add()` on a collection should not silently ignore duplicates without documentation
- Return types should be consistent; never return `null` when you could return `Optional` or empty collection

---

**Q96. What is `Objects` utility class and why use it?**
`java.util.Objects` provides null-safe utility methods:

```java
Objects.requireNonNull(value, "value must not be null");
Objects.requireNonNullElse(value, defaultValue);
Objects.toString(obj, "null");
Objects.equals(a, b);     // null-safe equality
Objects.hash(field1, field2); // convenience hashCode
Objects.checkIndex(index, length); // bounds check
```

---

**Q97. How do you implement `equals()` and `hashCode()` correctly?**
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User user)) return false;
    return age == user.age && Objects.equals(name, user.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```
Rules: reflexive, symmetric, transitive, consistent, `x.equals(null)` → false.

---

**Q98. What are common Java anti-patterns to avoid?**
- Returning `null` from methods — use `Optional` or empty collections
- Catching `Exception` or `Throwable` — too broad
- Empty catch blocks — silently swallowing errors
- String concatenation in loops — use `StringBuilder`
- Mutable public fields — breaks encapsulation
- Overusing `instanceof` — prefer polymorphism
- Creating objects inside loops unnecessarily
- Using `==` to compare strings

---

**Q99. What is the difference between `System.currentTimeMillis()` and `System.nanoTime()`?**
- `currentTimeMillis()` — wall-clock time since epoch; affected by system clock changes; use for timestamps.
- `nanoTime()` — high-resolution monotonic timer; not affected by clock changes; use for measuring elapsed time/performance.

```java
long start = System.nanoTime();
process();
long elapsed = System.nanoTime() - start;
System.out.printf("Took %.2f ms%n", elapsed / 1_000_000.0);
```

---

**Q100. How do you approach performance optimization in Java?**
1. **Measure first** — profile before optimizing (JFR, async-profiler); don't guess
2. **Algorithmic complexity** — fix O(n²) before micro-optimizing
3. **Reduce allocations** — reuse objects, prefer primitives, use object pools
4. **Minimize synchronization** — use lock-free structures, reduce scope
5. **Cache effectively** — `@Cacheable`, `LinkedHashMap` LRU, Caffeine
6. **Database** — fix N+1 queries, add indexes, batch operations
7. **I/O** — buffer reads/writes, use async I/O, connection pools
8. **GC tuning** — choose right GC, tune heap sizes, reduce object churn
9. **JIT-friendly code** — avoid megamorphic dispatch, keep methods small for inlining
10. **Benchmark properly** — use JMH (Java Microbenchmark Harness) for micro-benchmarks

---

*End of Top 100 Java Core Interview Q&A — 2 Years & Senior Level*
