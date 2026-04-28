# Top 100 Tricky Java Interview Questions (with Answers & Examples)
This list covers **Java fundamentals, OOP, collections, concurrency, JVM, tricky code outputs, and modern Java features**—all with clear answers and code examples to help you ace interviews.

---

## Part 1: Java Basics (20 Questions)
### 1. What is the difference between `==` and `equals()` in Java?
**Answer**:
- `==`: For **primitives**, compares values; for **objects**, compares memory addresses (reference equality).
- `equals()`: Default behavior (in `Object`) is same as `==`, but can be **overridden** to compare logical content (e.g., `String` compares character sequences).
**Example**:
```java
String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1 == s2);       // false (different objects)
System.out.println(s1.equals(s2));  // true (same content)
```

### 2. Can we override `private` methods?
**Answer**: No. `private` methods are not visible to subclasses, so they cannot be overridden (only hidden).

### 3. What is autoboxing and unboxing?
**Answer**:
- **Autoboxing**: Automatic conversion of **primitive → wrapper class** (e.g., `int → Integer`).
- **Unboxing**: Automatic conversion of **wrapper class → primitive** (e.g., `Integer → int`).
**Example**:
```java
Integer num = 10; // autoboxing (int → Integer)
int value = num;   // unboxing (Integer → int)
```

### 4. What is the output of `System.out.println(10 + 20 + "Java" + 30 + 40);`?
**Answer**: `30Java3040` (left-to-right evaluation: `10+20=30` → concatenate with `"Java"` → then append `30` and `40` as strings).

### 5. Is `main` a keyword in Java?
**Answer**: No. `main` is just a **method name**; you can have multiple `main` methods in a class (overloading).

### 6. Can we write `public void static main(String[] args)`?
**Answer**: No. Modifier order matters: **access modifier → static → return type** → method name. Correct: `public static void main(...)`.

### 7. What is the difference between `final`, `finally`, and `finalize`?
**Answer**:
- `final`: Keyword for **immutability** (final class = cannot be inherited; final method = cannot be overridden; final variable = constant).
- `finally`: Block in `try-catch`—**always executes** (used for resource cleanup).
- `finalize()`: Method in `Object`—called by GC before object destruction (deprecated in Java 9).

### 8. How many objects are created in `String s = "a" + "b" + "c";`?
**Answer**: **1 object** (compiler optimizes to `"abc"` and stores in the string pool).

### 9. What is the difference between `String`, `StringBuffer`, and `StringBuilder`?
**Answer**:
| Feature          | String               | StringBuffer         | StringBuilder        |
|------------------|----------------------|----------------------|----------------------|
| Mutability       | Immutable            | Mutable              | Mutable              |
| Thread Safety    | Safe (immutable)     | Safe (synchronized)  | Unsafe               |
| Performance      | Slow (new objects)   | Medium               | Fast (no sync)       |
**Example**:
```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // No new object created
```

### 10. What is a `null` in Java?
**Answer**: A special literal representing **no object reference**; `null` can be assigned to any reference type (throws `NullPointerException` if dereferenced).

### 11. Can we overload a method by changing the return type only?
**Answer**: No. Method overloading requires **different parameter lists** (number/type/order of parameters).

### 12. What is the output of the following code?
```java
public class Test {
    public static void main(String[] args) {
        int a = 10;
        System.out.println(a++);
        System.out.println(++a);
    }
}
```
**Answer**: `10` (post-increment: return first, then increment) → `12` (pre-increment: increment first, then return).

### 13. What is the difference between `break` and `continue`?
**Answer**:
- `break`: Exits the **entire loop/switch**.
- `continue`: Skips the **current iteration** and proceeds to the next.

### 14. Can a constructor be `final`?
**Answer**: No. Constructors are not inherited, so `final` is irrelevant.

### 15. What is the default value of an instance variable?
**Answer**:
- Primitives: `0` (int), `0.0` (float/double), `false` (boolean), `\u0000` (char).
- Reference types: `null`.

### 16. What is the difference between `this` and `super`?
**Answer**:
- `this`: Refers to the **current object instance** (accesses current class members, invokes current constructor).
- `super`: Refers to the **parent class** (accesses parent members, invokes parent constructor).

### 17. Can we have multiple `public` classes in one `.java` file?
**Answer**: No. Only **one public class** is allowed per file, and its name must match the filename.

### 18. What is the output of `System.out.println(0.1 + 0.2 == 0.3);`?
**Answer**: `false` (floating-point precision error in binary representation).

### 19. What is a `static` block?
**Answer**: A block executed **once when the class is loaded** (used for static variable initialization).
**Example**:
```java
class Demo {
    static {
        System.out.println("Class loaded");
    }
}
```

### 20. Can we declare a `static` method in an interface (pre-Java 8)?
**Answer**: No. Pre-Java 8 interfaces only have **abstract methods**; Java 8+ allows `static` and `default` methods.

---

## Part 2: OOP Concepts (20 Questions)
### 21. What are the 4 pillars of OOP?
**Answer**: **Encapsulation, Inheritance, Polymorphism, Abstraction**.

### 22. What is encapsulation?
**Answer**: Wrapping data (variables) and methods into a single unit (class), with **private variables** and public getter/setter methods to control access.
**Example**:
```java
class Student {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

### 23. What is inheritance?
**Answer**: A mechanism where a **subclass** inherits properties and methods from a **superclass** (supports code reuse). Java uses `extends` for single inheritance.

### 24. What is polymorphism?
**Answer**: "One interface, multiple implementations". Two types:
- **Compile-time (static)**: Method overloading.
- **Runtime (dynamic)**: Method overriding (determined by object type at runtime).

### 25. What is abstraction?
**Answer**: Hiding implementation details and showing only essential features. Achieved via **abstract classes** and **interfaces**.

### 26. Difference between abstract class and interface (Java 8+)?
**Answer**:
| Feature          | Abstract Class       | Interface            |
|------------------|----------------------|----------------------|
| Inheritance      | `extends` (single)   | `implements` (multiple) |
| Methods          | Can have abstract + concrete | Abstract (default) + static + concrete |
| Variables        | Any type (instance/static) | Only `public static final` (constants) |
| Constructor      | Yes                  | No                   |

### 27. Can an abstract class have a `main` method?
**Answer**: Yes. Abstract classes can have `main` and be executed (but cannot be instantiated).

### 28. What is method overriding?
**Answer**: A subclass provides a **specific implementation** of a method already defined in its superclass. Rules: same method name, parameters, return type (or covariant).

### 29. What is the `@Override` annotation?
**Answer**: Ensures the method **correctly overrides** a superclass method (compiler error if not).

### 30. Can we override `static` methods?
**Answer**: No. Static methods are **class-level**, not instance-level—subclass static methods **hide** (not override) parent static methods.

### 31. What is a covariant return type?
**Answer**: In overriding, a subclass can return a **subtype** of the parent method’s return type (Java 5+).
**Example**:
```java
class Parent {
    Object get() { return new Object(); }
}
class Child extends Parent {
    String get() { return "Java"; } // Covariant return
}
```

### 32. What is a `sealed` class (Java 17)?
**Answer**: Restricts which classes can extend it (uses `permits` to list allowed subclasses).
**Example**:
```java
public sealed class Shape permits Circle, Square {}
final class Circle extends Shape {}
final class Square extends Shape {}
```

### 33. What is a `record` (Java 16)?
**Answer**: Immutable data carrier class (automatically generates `equals()`, `hashCode()`, `toString()`, and accessors).
**Example**:
```java
record User(String name, int age) {}
User user = new User("Alice", 30);
```

### 34. Can a class be both `final` and `abstract`?
**Answer**: No. `final` = cannot be inherited; `abstract` = must be inherited—contradictory.

### 35. What is the diamond problem in inheritance?
**Answer**: A conflict when a class inherits from two classes with the same method (Java avoids it via **single inheritance**; resolved in interfaces with `default` methods via explicit override).

### 36. What is a constructor?
**Answer**: A special method to **initialize objects**—same name as class, no return type. If no constructor is defined, Java provides a **default no-arg constructor**.

### 37. Can a constructor call another constructor?
**Answer**: Yes. Use `this()` (current class) or `super()` (parent class)—must be the **first statement** in the constructor.

### 38. What is a copy constructor?
**Answer**: A constructor that creates a new object by copying an existing object’s state.
**Example**:
```java
class Car {
    String model;
    Car(Car car) { this.model = car.model; }
}
```

### 39. What is the difference between shallow and deep copy?
**Answer**:
- **Shallow copy**: Copies object references (original and copy share nested objects).
- **Deep copy**: Copies all nested objects (original and copy are fully independent).

### 40. What is a `static` nested class vs inner class?
**Answer**:
- **Static nested class**: Belongs to the outer **class** (no outer class instance needed).
- **Inner class**: Belongs to the outer **instance** (requires an outer class object to create).

---

## Part 3: Collections Framework (20 Questions)
### 41. What is the Collection Framework?
**Answer**: A set of interfaces and classes (e.g., `List`, `Set`, `Map`) to store and manipulate groups of objects—replaces legacy classes like `Vector`, `Hashtable`.

### 42. Difference between `ArrayList` and `LinkedList`?
**Answer**:
| Feature          | ArrayList            | LinkedList           |
|------------------|----------------------|----------------------|
| Internal Structure| Dynamic array        | Doubly linked list   |
| Access Time      | O(1) (random access) | O(n) (sequential)    |
| Insert/Delete    | O(n) (shift elements)| O(1) (no shift)      |
| Memory Overhead  | Low                  | High (node pointers) |
**Use Case**: `ArrayList` for read-heavy; `LinkedList` for write-heavy.

### 43. Difference between `HashSet` and `TreeSet`?
**Answer**:
| Feature          | HashSet              | TreeSet              |
|------------------|----------------------|----------------------|
| Order            | Unordered            | Sorted (natural order) |
| Underlying       | HashMap              | TreeMap (Red-Black Tree) |
| Null Values      | Allows one null      | No null (throws NPE) |
| Performance      | O(1) (add/contains)  | O(log n)             |

### 44. How does `HashMap` work (JDK 8+)?
**Answer**:
- Stores `key-value` pairs in an **array of buckets** (Node table).
- Uses `key.hashCode()` → hash → index to find the bucket.
- Resolves collisions via **chaining (linked list)**; if list length ≥8 and array size ≥64 → converts to **Red-Black Tree** (O(log n) lookup).
- Default capacity: 16; load factor: 0.75;扩容 to **2x** when threshold is reached.

### 45. Why does `HashMap` allow one `null` key and multiple `null` values?
**Answer**: `null` key is stored at index 0; `null` values have no hash conflict issues.

### 46. Difference between `HashMap` and `Hashtable`?
**Answer**:
| Feature          | HashMap              | Hashtable            |
|------------------|----------------------|----------------------|
| Thread Safety    | Unsafe               | Safe (synchronized)  |
| Null Keys/Values | 1 null key, many nulls | No nulls (throws NPE) |
| Performance      | Faster               | Slower               |
| Legacy           | No                   | Yes (replaced by ConcurrentHashMap) |

### 47. What is `ConcurrentHashMap`?
**Answer**: A thread-safe `Map` with **segment locking (JDK7)** or **CAS + synchronized (JDK8)**—better performance than `Hashtable`.

### 48. Difference between `Iterator` and `ListIterator`?
**Answer**:
- `Iterator`: Traverse **any Collection** (forward only; `remove()`).
- `ListIterator`: Traverse **List only** (forward/backward; `add()`, `set()`, `previous()`).

### 49. What is fail-fast vs fail-safe iterators?
**Answer**:
- **Fail-fast**: Throws `ConcurrentModificationException` if collection is modified during iteration (e.g., `ArrayList`, `HashMap`).
- **Fail-safe**: Works on a **copy** of the collection (no exception; e.g., `CopyOnWriteArrayList`, `ConcurrentHashMap`).

### 50. What is `Comparable` vs `Comparator`?
**Answer**:
- `Comparable`: Interface with `compareTo()`—defines **natural ordering** of a class (implemented by the class itself).
- `Comparator`: Interface with `compare()`—defines **custom ordering** (external class, used for sorting).
**Example**:
```java
// Comparable
class Student implements Comparable<Student> {
    int age;
    public int compareTo(Student s) { return this.age - s.age; }
}
// Comparator
Comparator<Student> byName = Comparator.comparing(Student::getName);
```

### 51. What is `Arrays.asList()`?
**Answer**: Converts an array to a **fixed-size List** (backed by the original array; cannot add/remove elements).
**Example**:
```java
List<String> list = Arrays.asList("A", "B");
list.add("C"); // UnsupportedOperationException
```

### 52. What is `LinkedHashMap`?
**Answer**: A `HashMap` that **preserves insertion order** (uses a doubly linked list to maintain order).

### 53. What is `IdentityHashMap`?
**Answer**: Uses **reference equality (`==`)** instead of `equals()`/`hashCode()` for key comparison.

### 54. What is `WeakHashMap`?
**Answer**: Entries with **weakly referenced keys** are GC’d when no other references to the key exist (used for caching).

### 55. How to sort a `List`?
**Answer**: Use `Collections.sort()` (for `Comparable`) or `list.sort(Comparator)` (Java 8+).
**Example**:
```java
List<Integer> list = Arrays.asList(3,1,2);
Collections.sort(list); // Natural order
list.sort(Comparator.reverseOrder()); // Reverse
```

### 56. What is the difference between `Collection` and `Collections`?
**Answer**:
- `Collection`: **Root interface** of the Collection Framework (e.g., `List`, `Set`).
- `Collections`: **Utility class** with static methods (e.g., `sort()`, `reverse()`, `unmodifiableList()`).

### 57. Can a `List` contain duplicate elements?
**Answer**: Yes (e.g., `ArrayList`, `LinkedList`). `Set` does not allow duplicates.

### 58. What is `EnumSet`?
**Answer**: A high-performance `Set` for **enum types** (bit-vector implementation; faster than `HashSet`).

### 59. What is `PriorityQueue`?
**Answer**: A queue ordered by **natural ordering or a comparator** (head is the smallest element).

### 60. How to make a collection unmodifiable?
**Answer**: Use `Collections.unmodifiableXXX()` (e.g., `unmodifiableList()`) or Java 9+ `List.of()`, `Set.of()`.

---

## Part 4: Concurrency & Multithreading (20 Questions)
### 61. How to create a thread in Java?
**Answer**:
1. Extend `Thread` class and override `run()`.
2. Implement `Runnable` interface and pass to `Thread` constructor.
3. Implement `Callable` (returns result) and use `FutureTask` + `Thread`.
**Example**:
```java
// Runnable
class MyRunnable implements Runnable {
    public void run() { System.out.println("Thread"); }
}
Thread t = new Thread(new MyRunnable());
t.start(); // Not run()!
```

### 62. Difference between `start()` and `run()`?
**Answer**:
- `start()`: Starts a **new thread** and invokes `run()` in that thread.
- `run()`: Executes `run()` in the **current thread** (no new thread created).

### 63. What is the `Thread` lifecycle?
**Answer**: `New → Runnable → Running → Blocked/Waiting/Timed Waiting → Terminated`.

### 64. What is `synchronized`?
**Answer**: A keyword for **thread synchronization**—ensures only one thread executes a block/method at a time (prevents race conditions). Can be used on:
- Instance methods (locks on `this`).
- Static methods (locks on class object).
- Code blocks (locks on a specified object).

### 65. What is a race condition?
**Answer**: A bug where multiple threads access shared data **concurrently**—result depends on thread scheduling (non-deterministic). Solved via synchronization.

### 66. What is `wait()`, `notify()`, `notifyAll()`?
**Answer**: Methods in `Object` for **inter-thread communication** (must be called inside a `synchronized` block):
- `wait()`: Releases the lock and makes the thread wait.
- `notify()`: Wakes up **one** waiting thread.
- `notifyAll()`: Wakes up **all** waiting threads.

### 67. Difference between `sleep()` and `wait()`?
**Answer**:
| Feature          | sleep()               | wait()                |
|------------------|-----------------------|-----------------------|
| Belongs To       | `Thread` class        | `Object` class        |
| Lock Behavior    | Does **not** release lock | Releases lock        |
| Wakes Up On      | Timeout or interrupt   | `notify()`/`notifyAll()` |
| Usage            | Any context           | Only in `synchronized` |

### 68. What is `join()`?
**Answer**: Makes the current thread **wait** until the target thread terminates.
**Example**:
```java
Thread t1 = new Thread(() -> {});
t1.start();
t1.join(); // Main thread waits for t1 to finish
```

### 69. What is `volatile`?
**Answer**: A keyword ensuring **visibility** (changes to a `volatile` variable are immediately visible to other threads) and **prevents instruction reordering** (but not atomicity).

### 70. What is `Atomic` classes (e.g., `AtomicInteger`)?
**Answer**: Classes for **lock-free, thread-safe operations** on single variables (uses CAS—Compare-and-Swap).

### 71. What is `Callable` vs `Runnable`?
**Answer**:
- `Runnable`: `run()` returns `void`, cannot throw checked exceptions.
- `Callable`: `call()` returns a value, can throw exceptions. Used with `ExecutorService` and `Future`.

### 72. What is `ExecutorService`?
**Answer**: A framework for **managing thread pools** (reuses threads, avoids overhead of creating new threads).
**Example**:
```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<Integer> future = executor.submit(() -> 10);
executor.shutdown();
```

### 73. What is `Future`?
**Answer**: Represents the **result of an asynchronous computation** (use `get()` to retrieve the result, blocks until completion).

### 74. What is `CountDownLatch`?
**Answer**: A synchronization aid that allows one or more threads to **wait** until a set of operations completes.
**Example**:
```java
CountDownLatch latch = new CountDownLatch(2);
// Threads call latch.countDown();
latch.await(); // Main thread waits until count=0
```

### 75. What is `CyclicBarrier`?
**Answer**: A synchronization aid that makes a set of threads **wait for each other** to reach a common barrier point.

### 76. What is `Semaphore`?
**Answer**: Controls access to a resource with a **fixed number of permits** (used for limiting concurrent access).

### 77. What is `ReentrantLock`?
**Answer**: An explicit lock with more features than `synchronized`:
- `tryLock()`, `lockInterruptibly()`, `fair/unfair` modes.
- Must call `unlock()` in `finally`.
**Example**:
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try { /* critical section */ }
finally { lock.unlock(); }
```

### 78. What is `ReadWriteLock`?
**Answer**: A lock that allows **multiple readers** or **one writer** (improves concurrency for read-heavy data).

### 79. What is a deadlock?
**Answer**: A situation where two or more threads are **blocked forever**, each waiting for a resource held by the other. Conditions: mutual exclusion, hold and wait, no preemption, circular wait.

### 80. How to avoid deadlock?
**Answer**:
- Use **ordered locking** (acquire locks in a fixed order).
- Use `tryLock()` with timeout.
- Avoid nested locks.

---

## Part 5: JVM & Memory Management (10 Questions)
### 81. What is JVM?
**Answer**: Java Virtual Machine—executes Java bytecode (.class files) and provides platform independence.

### 82. JVM memory structure?
**Answer**:
1. **Heap**: Stores objects (GC’d; split into Young/Old Gen).
2. **Stack**: Stores method frames (local variables, return addresses; thread-safe).
3. **Method Area**: Stores class metadata, static variables, constants (PermGen pre-Java 8; Metaspace Java 8+).
4. **PC Register**: Tracks current instruction address.
5. **Native Method Stack**: For native (C/C++) methods.

### 83. What is garbage collection (GC)?
**Answer**: Automatic process to **reclaim memory** from unreachable objects (prevents memory leaks).

### 84. Common GC algorithms?
**Answer**:
- **Mark-Sweep**: Marks live objects, sweeps dead ones (fragmentation).
- **Copy**: Copies live objects to a new space (no fragmentation; uses 2x space).
- **Mark-Compact**: Marks live objects, compacts them to one end (reduces fragmentation).
- **Generational GC**: Divides heap into Young (Eden/Survivor) and Old Gen (optimizes for short-lived objects).

### 85. What is `OutOfMemoryError`?
**Answer**: Thrown when JVM runs out of heap/metaspace memory (e.g., infinite object creation).

### 86. What is `StackOverflowError`?
**Answer**: Thrown when the **call stack exceeds its limit** (e.g., infinite recursion).

### 87. What is class loading?
**Answer**: Process of loading `.class` files into JVM. Three steps: **Loading → Linking (Verify/Prepare/Resolve) → Initialization**.

### 88. Class loaders in Java?
**Answer**:
1. **Bootstrap ClassLoader**: Loads core classes (rt.jar).
2. **Extension ClassLoader**: Loads ext classes (jre/lib/ext).
3. **Application ClassLoader**: Loads user classes (CLASSPATH).
Follows **parent delegation** model.

### 89. What is `finalize()`?
**Answer**: Method called by GC before object destruction (deprecated in Java 9; use `Cleaner` instead).

### 90. How to force GC?
**Answer**: Call `System.gc()`—**only a request** (JVM may ignore it).

---

## Part 6: Tricky Code Output Questions (10 Questions)
### 91. Output?
```java
public class Test {
    public void print(Integer i) { System.out.println("Integer"); }
    public void print(int i) { System.out.println("int"); }
    public static void main(String[] args) {
        Test t = new Test();
        t.print(10);
    }
}
```
**Answer**: `int` (prefer primitive over wrapper in overloading).

### 92. Output?
```java
String s1 = "Hello";
String s2 = new String("Hello");
String s3 = s2.intern();
System.out.println(s1 == s2);
System.out.println(s1 == s3);
```
**Answer**: `false` → `true` (`intern()` adds to string pool and returns pool reference).

### 93. Output?
```java
public class Base {
    public Base() { foo(); }
    public void foo() { System.out.println("Base"); }
}
class Derived extends Base {
    private int x = 10;
    public void foo() { System.out.println("Derived: " + x); }
}
public class Main {
    public static void main(String[] args) { new Derived(); }
}
```
**Answer**: `Derived: 0` (base constructor calls overridden `foo()` before `Derived` initializes `x`).

### 94. Output?
```java
Integer a = 128;
Integer b = 128;
System.out.println(a == b);
Integer c = 127;
Integer d = 127;
System.out.println(c == d);
```
**Answer**: `false` → `true` (Integer cache for `-128 to 127`).

### 95. Output?
```java
public class Test {
    static {
        System.out.println("Static Block");
    }
    {
        System.out.println("Instance Block");
    }
    public Test() {
        System.out.println("Constructor");
    }
    public static void main(String[] args) {
        new Test();
    }
}
```
**Answer**: `Static Block` → `Instance Block` → `Constructor` (static block first, then instance block, then constructor).

### 96. Output?
```java
int[] arr = new int[5];
System.out.println(arr[0]);
```
**Answer**: `0` (array elements are initialized to default values).

### 97. Output?
```java
String str = null;
System.out.println(str.length());
```
**Answer**: `NullPointerException` (dereferencing `null`).

### 98. Output?
```java
public class Test {
    public static void main(String[] args) {
        try {
            int a = 5/0;
        } catch (ArithmeticException e) {
            System.out.println("Catch");
            return;
        } finally {
            System.out.println("Finally");
        }
    }
}
```
**Answer**: `Catch` → `Finally` (finally always executes before return).

### 99. Output?
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
for (String s : list) {
    list.remove(s); // ConcurrentModificationException
}:1

```
**Answer**: Throws `ConcurrentModificationException` (fail-fast iterator).

### 100. Output?
```java
public class Test {
    public static void main(String[] args) {
        String s1 = "abc";
        String s2 = "ab" + "c";
        System.out.println(s1 == s2);
    }
}
```
**Answer**: `true` (compiler optimizes to same string pool object).

---

Would you like me to convert this entire list into a **one-page cheat sheet (PDF/Markdown)** for quick interview review?