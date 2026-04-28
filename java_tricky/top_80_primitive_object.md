Below are the answers and examples for all 80 questions. Each includes a concise explanation and, where helpful, a code snippet.

---

### Primitives & Basic Types

1. **What is the size of a `boolean` in Java? Is it exactly 1 bit?**  
   *Answer*: The Java Language Specification does not define a precise size; it depends on the JVM. Typically, a `boolean` occupies 1 byte in arrays and 4 bytes on stack (like an `int`).  
   *Example*: `boolean[] flags = new boolean[10];` – each element may be a byte.

2. **Why does `int i = 10L;` fail to compile, but `long l = 10;` works?**  
   *Answer*: `10L` is a `long` literal, assigning to `int` loses precision (explicit cast required). `10` is an `int` literal, and widening to `long` is safe.  
   *Example*:  
   ```java
   long l = 10;     // OK, int widened to long
   int i = 10L;     // error: incompatible types
   int i2 = (int)10L; // OK
   ```

3. **What happens when you add a `byte` and a `short`? What is the resulting type?**  
   *Answer*: Binary numeric promotion converts both to `int`; result is `int`.  
   *Example*:  
   ```java
   byte b = 10;
   short s = 20;
   int result = b + s; // result is int
   ```

4. **Does `float f = 0.1;` compile? If not, why?**  
   *Answer*: No, because `0.1` is a double literal. Use `0.1f`.  
   *Example*: `float f = 0.1f;`

5. **What is the default value of a local `int` variable? Of an instance `int` variable?**  
   *Answer*: Local variables have no default; they must be initialized. Instance variables default to `0`.  
   *Example*:  
   ```java
   int x;          // compilation error if used
   class Demo { int y; }  // y = 0
   ```

6. **Why does `System.out.println(0.1 + 0.2);` not print `0.3` exactly?**  
   *Answer*: Floating-point arithmetic uses binary representation; `0.1` and `0.2` cannot be represented exactly, leading to small rounding errors.  
   *Example*: Prints `0.30000000000000004`.

7. **What is the difference between `++i` and `i++` when used inside a complex expression like `i = i++`?**  
   *Answer*: `i++` returns original value then increments; `++i` increments then returns new value. `i = i++` leaves `i` unchanged (temporary value).  
   *Example*:  
   ```java
   int i = 5;
   i = i++; // i remains 5
   ```

8. **Does Java support unsigned primitive types? If not, how can you work with unsigned values?**  
   *Answer*: Only `char` is unsigned (16-bit). For others, use wrapper methods like `Integer.toUnsignedString()` and `Integer.compareUnsigned()` (Java 8+).  
   *Example*: `int signed = -1; String unsigned = Integer.toUnsignedString(signed); // "4294967295"`

9. **What happens when you cast a `long` larger than `Integer.MAX_VALUE` to `int`?**  
   *Answer*: The higher 32 bits are discarded; the result is the lower 32 bits interpreted as `int` (may be negative).  
   *Example*: `long l = 2147483648L; int i = (int)l; // i = -2147483648`

10. **Why does `int i = (int) 10.9;` produce `10`? Is rounding performed?**  
    *Answer*: Truncation toward zero, not rounding.  
    *Example*: `(int)10.9` → `10`, `(int)-10.9` → `-10`.

11. **What is the result of `-1 >>> 1`? Why?**  
    *Answer*: `2147483647` (max int). `>>>` is unsigned right shift, shifts zeros into the most significant bit. `-1` is all ones; after shift, becomes `0111...111`.  
    *Example*: `System.out.println(-1 >>> 1);`

12. **Can a `boolean` be cast to `int`? Can an `int` be used in a conditional expression expecting `boolean`?**  
    *Answer*: No to both. Java does not treat numbers as booleans.  
    *Example*: `if (1) {}` // compilation error.

---

### Objects, References, Equality

13. **What is the difference between `==` and `equals()` for primitive wrappers like `Integer`?**  
    *Answer*: `==` compares reference identity (or value for primitives). `equals()` compares actual int value.  
    *Example*:  
    ```java
    Integer a = new Integer(100);
    Integer b = new Integer(100);
    System.out.println(a == b);     // false
    System.out.println(a.equals(b));// true
    ```

14. **Why does `new Integer(100) == new Integer(100)` sometimes return `false`?**  
    *Answer*: Always false because `new` creates distinct objects; reference equality compares memory addresses.

15. **What is the value of `0.0 / 0.0`? What about `1.0 / 0.0`?**  
    *Answer*: `NaN` (Not a Number) for `0.0/0.0`; `Infinity` for positive division by zero; `-Infinity` for negative.  
    *Example*: `Double.isNaN(0.0/0.0)` → true.

16. **What is the difference between `String` literal and `new String("...")` regarding object creation and reference equality?**  
    *Answer*: Literal uses string pool (interned). `new String` always creates a new heap object.  
    *Example*:  
    ```java
    String s1 = "hello";
    String s2 = "hello";
    String s3 = new String("hello");
    System.out.println(s1 == s2); // true
    System.out.println(s1 == s3); // false
    ```

17. **If two objects have the same `hashCode()`, does that mean they are equal?**  
    *Answer*: No. Different objects can have same hash code (collision).  
    *Example*: `"Aa"` and `"BB"` both have hash code 2112 in some JDK versions.

18. **What happens if you override `equals()` but not `hashCode()`?**  
    *Answer*: Breaks hash-based collections (e.g., `HashSet`, `HashMap`). Objects that are equal may be stored in different buckets.  
    *Example*: `map.put(key1, val); map.get(key2);` returns null even if `key1.equals(key2)`.

19. **Why is `String` immutable? What pitfalls arise from immutability when passing to methods?**  
    *Answer*: Immutability enables caching, security, thread-safety. Pitfall: Changing a string inside a method does not affect caller's reference (can't "modify in place").  
    *Example*:  
    ```java
    void appendWorld(String s) { s += " world"; } // caller's string unchanged
    ```

20. **What is a “reachable” object? When is an object eligible for garbage collection even if referenced?**  
    *Answer*: Reachable = accessible from GC roots (e.g., local variables, static fields). An object can be GC’d if only weakly reachable (weak/phantom references).  
    *Example*: `WeakReference<Object> wr = new WeakReference<>(obj);` after no strong refs, obj may be collected.

21. **What are strong, soft, weak, and phantom references? Which one allows an object to be GC’d as soon as possible?**  
    *Answer*: Weak reference.  
    - Strong: normal reference  
    - Soft: GC may collect under memory pressure  
    - Weak: collected eagerly  
    - Phantom: after finalization, used for cleanup

22. **What is a reference queue? How is it used with weak references?**  
    *Answer*: A queue that receives references when their referent is GC’d. Used to perform cleanup.  
    *Example*:  
    ```java
    ReferenceQueue<Object> q = new ReferenceQueue<>();
    WeakReference<Object> wr = new WeakReference<>(obj, q);
    // Later, poll q to know when obj is gone
    ```

23. **Can you resurrect an object in `finalize()`? Why is this dangerous?**  
    *Answer*: Yes, by assigning `this` to a static field. Dangerous because finalize is called at most once; resurrection can lead to inconsistent state and unpredictable GC behavior.  
    *Example*: `static Object saved; void finalize() { saved = this; }`

24. **Why is `finalize()` deprecated? What should you use instead?**  
    *Answer*: Unpredictable, performance issues, deadlock-prone. Use `Cleaner` (Java 9+) or try-with-resources (`AutoCloseable`).

---

### Object Pool & Creation

25. **What is the object pool pattern? What are the typical pitfalls (memory leaks, race conditions, stale objects)?**  
    *Answer*: Reuses objects to reduce allocation cost. Pitfalls: forgetting to reset state (stale objects), concurrency issues, memory leak if borrowed objects are never returned, hidden overhead.

26. **In a JDBC connection pool, why must you close `Connection`, `Statement`, and `ResultSet` even if you return the connection to the pool?**  
    *Answer*: Closing returns connection to pool, but `Statement`/`ResultSet` hold database resources (cursors, memory). Failure to close them leads to resource leaks.

27. **How does creating an object via `new` differ from `Class.forName(...).newInstance()`?**  
    *Answer*: `new` is compile-time type-safe; `Class.forName().newInstance()` uses reflection, requires no-arg constructor, and throws checked exceptions.  
    *Example*:  
    ```java
    Class<?> clazz = Class.forName("MyClass");
    Object obj = clazz.newInstance(); // deprecated, now use clazz.getDeclaredConstructor().newInstance()
    ```

28. **What is the difference between `clone()` and a copy constructor? Which one is safer?**  
    *Answer*: `clone()` is protected, requires `Cloneable`, does shallow copy by default, prone to misuse. Copy constructor is explicit, safe, and customizable.  
    *Example*:  
    ```java
    public class MyClass {
        public MyClass(MyClass other) { this.x = other.x; } // copy constructor
    }
    ```

29. **Does `Arrays.asList(…)` create a new array object or a view?**  
    *Answer*: It returns a fixed-size list **backed by** the original array. Modifying the list modifies the array, and structural changes (add/remove) throw `UnsupportedOperationException`.  
    *Example*: `List<String> list = Arrays.asList("a","b"); list.set(0,"c"); // array changed`

30. **What is the “diamond operator” (`<>`) and how does it affect type inference during object creation?**  
    *Answer*: Diamond operator allows the compiler to infer generic type arguments from the context, avoiding redundancy.  
    *Example*: `List<String> list = new ArrayList<>();` (instead of `new ArrayList<String>()`).

31. **What happens when you assign a subclass object to a superclass reference? What methods can be called?**  
    *Answer*: You can only call methods defined in the superclass (or overridden versions). Subclass-specific methods are not accessible without downcasting.  
    *Example*:  
    ```java
    Number n = new Integer(5);
    n.intValue(); // works
    n.longValue(); // works (defined in Number)
    // n.byteValue()? No, but Number has byteValue()? Actually Number has byteValue()
    // Better: subclass-specific: ((Integer)n).compareTo(...) not allowed without cast.
    ```

---

### Autoboxing & Unboxing

32. **Why does `Integer a = 127; Integer b = 127; System.out.println(a == b);` print `true`, but for `128` it prints `false`?**  
    *Answer*: `Integer` cache for values -128..127 (JLS). Within this range, `valueOf()` returns cached instances; outside, new objects.  
    *Example*: use `a.equals(b)` instead.

33. **What is autoboxing? What is the hidden cost of mixing primitives and wrappers in collections?**  
    *Answer*: Automatic conversion between primitive and wrapper. Hidden cost: temporary wrapper objects created, causing memory overhead and GC pressure.  
    *Example*: `list.add(42);` creates an `Integer`.

34. **What is unboxing? When does it throw `NullPointerException`?**  
    *Answer*: Converting wrapper to primitive. Throws NPE if the wrapper reference is `null`.  
    *Example*: `Integer i = null; int j = i;` // NPE.

35. **Why does `Integer i = null; int j = i;` throw NPE? How to avoid?**  
    *Answer*: Because autounboxing tries to call `i.intValue()` on null. Avoid by checking null before unboxing.

36. **What happens when you autobox a primitive to a wrapper and then compare with `==`?**  
    *Answer*: If one operand is primitive, unboxing occurs; actual values compared. If both are wrapper objects, identity comparison.  
    *Example*:  
    ```java
    Integer a = 100;
    int b = 100;
    System.out.println(a == b); // true (a unboxed)
    ```

37. **Does autoboxing work with method overloading? Which method is called for `void m(int i)` and `void m(Integer i)` when passing `5`?**  
    *Answer*: The most specific method that avoids boxing/unboxing wins. `m(int)` is called because widening is preferred over boxing.  
    *Example*:  
    ```java
    void m(int i) {}   // called for m(5)
    void m(Integer i) {}
    ```

38. **Why does `Long l = 5L;` compile but `Long l2 = 5;` fails?**  
    *Answer*: `5` is an `int` literal. Widening from `int` to `long` then boxing to `Long` is allowed. `5` alone is `int`, cannot box directly to `Long` (requires `long` first). Actually `Long l2 = 5;` fails because `5` is int, and Java requires a `long` for boxing to `Long`. `Long l = 5L;` works because `5L` is long literal.  
    Wait – careful: `Long l = 5;` fails because int cannot be converted to Long. `Long l = 5L;` works.  
    *Correction*: `Long l = 5L;` (long literal) OK. `Long l = 5;` fails because 5 is int → can't convert to Long without explicit cast.  
    But `Long l = 5L;` is fine.

39. **What is the result of `new Integer(5) == 5`? Is it autounboxing or something else?**  
    *Answer*: `true` because the `Integer` is unboxed to `int` and then compared.  
    *Example*: `new Integer(5) == 5` → true.

40. **What is the Java Virtual Method Invocation (dynamic dispatch)? How does it affect method calls on superclass references?**  
    *Answer*: At runtime, the actual object’s method is called, not the reference type's method (for overridden methods).  
    *Example*:  
    ```java
    class Parent { void m() { System.out.println("Parent"); } }
    class Child extends Parent { void m() { System.out.println("Child"); } }
    Parent p = new Child(); p.m(); // prints "Child"
    ```

---

### Virtual, Overriding, Constructors

41. **Why can’t you override a `private` method? Why can’t you override a `static` method?**  
    *Answer*: Private methods are not visible to subclasses. Static methods are class-level, not instance-level; they are hidden, not overridden.

42. **What is “method hiding” vs “method overriding”? Give an example with static methods.**  
    *Answer*: Hiding: static methods in subclass mask those in superclass. Overriding: instance methods.  
    *Example*:  
    ```java
    class Parent { static void foo() {} void bar() {} }
    class Child extends Parent { static void foo() {} // hides; void bar() {} // overrides }
    ```

43. **What is the behavior of calling an overridden method inside a constructor? Why is this a pitfall?**  
    *Answer*: The subclass’s version of the method is called before the subclass constructor has run, possibly using uninitialized fields.  
    *Example*:  
    ```java
    class Parent {
        Parent() { print(); }
        void print() { System.out.println("Parent"); }
    }
    class Child extends Parent {
        private int x = 5;
        void print() { System.out.println(x); } // prints 0, not 5
    }
    ```

44. **Does `super.someMethod()` always call the parent version? What if the child overrides it?**  
    *Answer*: Yes, `super` explicitly calls the parent’s version, bypassing the override.

45. **What happens when a subclass declares a field with the same name as a parent field? How is it accessed?**  
    *Answer*: Field is hidden, not overridden. Access depends on reference type. Use `super.field` or cast.  
    *Example*:  
    ```java
    class A { int x = 1; }
    class B extends A { int x = 2; }
    B b = new B(); b.x → 2; ((A)b).x → 1;
    ```

46. **What is a constructor? Can a constructor be `private`? What is the use?**  
    *Answer*: A constructor initializes an object. Private constructor prevents instantiation from outside, used in singletons, static utility classes, or factory methods.  
    *Example*: `public class Singleton { private Singleton() {} }`

47. **What is the default constructor? When is it not generated?**  
    *Answer*: No-argument constructor provided by compiler if no constructor is defined. Not generated if any constructor is explicitly defined.

48. **What is constructor chaining? What happens if a constructor calls `this()` and `super()` both?**  
    *Answer*: Chaining: calling one constructor from another via `this()` or `super()`. `this()` and `super()` cannot both appear in same constructor because each must be first statement.

49. **Why must `this()` or `super()` be the first statement in a constructor?**  
    *Answer*: To ensure the superclass is fully initialized before subclass fields are used or methods called.

50. **What is an initialization block? What is the order of execution among static blocks, instance blocks, and constructors?**  
    *Answer*: Static blocks run once at class loading. Instance blocks run before constructor body. Order: static init → instance init → constructor.  
    *Example*:  
    ```java
    class Demo {
        static { System.out.println("static"); }
        { System.out.println("instance"); }
        Demo() { System.out.println("constructor"); }
    }
    // new Demo() prints static, instance, constructor
    ```

51. **Can you call a non-final method from a constructor? What risk does that pose?**  
    *Answer*: Yes, but the overridden version may run before the subclass’s fields are initialized (see question 43).

52. **What is the difference between a copy constructor and the `clone()` method regarding handling of `final` fields?**  
    *Answer*: `clone()` cannot set final fields because it doesn’t call a constructor; copy constructor can set them in its constructor.  
    *Example*:  
    ```java
    class Point {
        final int x;
        Point(Point other) { this.x = other.x; } // ok
        // clone would be problematic
    }
    ```

---

### Shallow & Deep Copy

53. **What is shallow copy? How does `Object.clone()` behave by default?**  
    *Answer*: Copies only the object’s fields, not the objects they reference. The default `clone()` makes a shallow copy.  
    *Example*:  
    ```java
    class Address { String city; }
    class Person { Address addr; }
    Person p1 = new Person(); p1.addr = new Address();
    Person p2 = (Person) p1.clone(); // p2.addr == p1.addr (same object)
    ```

54. **Why does `clone()` require implementing `Cloneable`? What happens if you call `clone()` on a class that does not implement `Cloneable`?**  
    *Answer*: `Cloneable` is a marker interface. Without it, `clone()` throws `CloneNotSupportedException`.

55. **How do you implement deep copy without using `clone()`? What are the trade-offs of using serialization?**  
    *Answer*: Copy constructor, factory method, or serialization. Serialization is easy but slower, may cause security issues, and requires `Serializable`.  
    *Deep copy via serialization*:  
    ```java
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bos);
    oos.writeObject(original);
    ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
    return (MyClass) ois.readObject();
    ```

56. **What happens if you shallow-copy an object that contains mutable fields? Give an example where changes affect the original.**  
    *Answer*: Modifying the mutable field in the copy changes the original because they share the same reference.  
    *Example*:  
    ```java
    Person p1 = new Person("John");
    Person p2 = shallowCopy(p1);
    p2.name.append(" Doe"); // p1.name also changed
    ```

57. **Is an array a primitive type or an object in Java? How is it copied (shallow/deep)?**  
    *Answer*: Array is an object. `clone()` on array returns shallow copy. For primitive arrays, copy is deep in values but shallow for object arrays (references copied).  
    *Example*:  
    ```java
    int[] a = {1,2};
    int[] b = a.clone(); // b is new array with values
    String[] sa = {"a"};
    String[] sb = sa.clone(); // sb[0] == sa[0]
    ```

58. **What is the effect of `System.arraycopy()` on reference types? Does it create new objects?**  
    *Answer*: It copies references, not objects. Shallow copy.

59. **What is the difference between `Integer.valueOf(100)` and `new Integer(100)` regarding caching and object creation?**  
    *Answer*: `valueOf()` uses cache for -128..127, returns possibly cached instance. `new` always creates new object. (Note: `new Integer` deprecated since Java 9.)

60. **Why does `Integer.valueOf(127) == Integer.valueOf(127)` return `true` but `Integer.valueOf(128) == Integer.valueOf(128)` return `false`? How to reliably compare?**  
    *Answer*: Cache range. Use `equals()` or `intValue()`.

61. **What happens when autoboxing a `char` to `Character` and comparing with `==`?**  
    *Answer*: Similar to `Integer`, `Character` caches values 0-127. For chars outside that, `==` returns false.  
    *Example*: `Character c1 = 'a', c2 = 'a'; c1 == c2 -> true`; `'δ'` (not cached) -> false.

62. **What are the memory and performance implications of excessive autoboxing in loops?**  
    *Answer*: Creates many temporary wrapper objects, increasing GC pressure and slowing execution.  
    *Example*: `for(int i=0; i<1000000; i++) { sum += list.get(i); }` where list is `List<Integer>` – each addition unboxes, fine, but if you do `Integer sum = 0;` and sum in loop, many `Integer` objects created.

63. **Can you have a `null` reference to a primitive wrapper? What unboxing operations cause NPE?**  
    *Answer*: Yes, e.g., `Integer i = null;`. Any arithmetic, comparison, or assignment to primitive causes NPE: `int x = i;`, `i + 5`, `i == 5`.

64. **What is object creation overhead? How does object pooling reduce it? When is pooling actually harmful?**  
    *Answer*: Overhead includes memory allocation, constructor calls, GC. Pooling reuses objects to reduce allocation. Harmful when objects are cheap to create, pooling adds synchronization overhead, or when objects become stale/leak memory.

65. **Why does `String s = new String("abc")` create two objects? Explain the string pool interaction.**  
    *Answer*: One object in heap (`new String`), another in string pool (if not already present) from the literal `"abc"`. The literal is interned automatically.

66. **What is the difference between `String intern()` and normal string creation?**  
    *Answer*: `intern()` returns a canonical representation from the string pool; subsequent `intern()` on equal strings returns same reference. Normal creation (literal or `new`) does not guarantee pooling for `new`.

67. **How does Java handle method parameters: pass-by-value or pass-by-reference? Show with primitives vs objects.**  
    *Answer*: Always pass-by-value. For objects, the reference is passed by value (copy of reference).  
    *Example*:  
    ```java
    void change(int x) { x = 5; } // caller unaffected
    void change(Person p) { p.name = "new"; } // affects caller's object
    void changeRef(Person p) { p = new Person(); } // caller's reference unchanged
    ```

68. **Why can’t you swap two objects inside a method using their references?**  
    *Answer*: Because references are passed by value; swapping local copies does not affect caller’s variables.  
    *Example*:  
    ```java
    void swap(Object a, Object b) { Object temp = a; a = b; b = temp; } // futile
    ```

69. **What is the difference between `final`, `finally`, and `finalize()`?**  
    *Answer*:  
    - `final`: modifier for classes, methods, variables.  
    - `finally`: block after try-catch for cleanup.  
    - `finalize()`: deprecated method called by GC.

70. **What happens if you declare a variable `final` and then try to modify its state (if it’s a mutable object)?**  
    *Answer*: The reference cannot be reassigned, but the object’s internal state can be changed.  
    *Example*: `final List<String> list = new ArrayList<>(); list.add("hello");` // allowed.

71. **Can a constructor throw an exception? What happens to the partially constructed object?**  
    *Answer*: Yes. The object may be partially initialized; it becomes eligible for GC (no reference returned).

72. **What is a static factory method? How does it differ from a public constructor regarding object creation and caching (e.g., `Boolean.valueOf`)?**  
    *Answer*: Static factory returns an instance, can cache objects, return subtypes, or control instance count. Constructor always returns a new object.  
    *Example*: `Boolean.valueOf(true)` returns static `Boolean.TRUE`.

73. **What is the “virtual method invocation in constructor” problem? Write a failing example.**  
    *Answer*: Calling overridden method in constructor leads to subclass method being called before subclass fields are initialized (question 43 gives example, prints 0 instead of 5).

74. **Why does `double d = 10.0; int i = d;` produce a compilation error but `int i = (int)10.0;` works?**  
    *Answer*: Narrowing conversion (double to int) may lose data; requires explicit cast. Literal is known at compile time.

75. **What is the difference between bitwise `&` and logical `&&`? How do they affect boolean expressions?**  
    *Answer*: `&` always evaluates both operands; `&&` short-circuits (stops if left false). Also, `&` can be used with integers.  
    *Example*: `if (x != null & x.equals(""))` may throw NPE; `&&` prevents it.

76. **What is the result of `short s = 32767; s = s + 1;`? What about `s++`?**  
    *Answer*: `s = s + 1;` fails to compile because `s+1` produces `int`. `s++;` works because compound assignment includes implicit cast.  
    *Result of `s++` after execution: `-32768` (overflow).

77. **What is the meaning of `this` reference inside a constructor? Can it be passed to another method before the object is fully initialized?**  
    *Answer*: `this` refers to the object being constructed, but it is not yet fully initialized (superclass constructor may not have finished). Passing `this` to another method is allowed but dangerous; that method may see inconsistent state.

78. **Why is it bad practice to create an object pool for immutable objects (e.g., `String`)? When is pooling beneficial?**  
    *Answer*: Immutable objects are safe to reuse, but pooling adds overhead. String has its own pool (intern). Pool beneficial for heavy-weight objects (e.g., database connections, threads). Immutable objects like `Integer` have built-in small cache.

79. **What is the difference between shallow copy of a `Collection` (e.g., `new ArrayList<>(originalList)`) and deep copy?**  
    *Answer*: Shallow copy: new collection, same elements (references). Deep copy: new collection and new element objects (clone/copy each element).  
    *Example*:  
    ```java
    List<Person> shallow = new ArrayList<>(original); // Person objects shared
    List<Person> deep = original.stream().map(Person::new).collect(Collectors.toList());
    ```

80. **How does Java’s `record` (since Java 14) affect shallow/deep copy and equals comparisons? Do records automatically provide deep copy?**  
    *Answer*: Records provide automatic shallow copy via canonical constructor and `equals` based on all components. No automatic deep copy; you must manually deep copy components if they are mutable.  
    *Example*:  
    ```java
    record R(List<String> list) {}
    R r1 = new R(new ArrayList<>());
    R r2 = new R(r1.list()); // shallow copy; both share same list
    ```

---

These answers and examples cover the most common pitfalls and nuances for Java interviews. Good luck!