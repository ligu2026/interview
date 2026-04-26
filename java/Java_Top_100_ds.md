# Top 100 Java Interview Questions: Object, Class, Core & Autoboxing

## 1. What is a class in Java?
A class is a blueprint or template that defines the properties (fields) and behaviors (methods) of objects.  
**Example:**
```java
public class Car {
    String model;
    void drive() { System.out.println("Driving"); }
}
```

## 2. What is an object in Java?
An object is an instance of a class, created using the `new` keyword. It holds actual data.  
**Example:**
```java
Car myCar = new Car();
myCar.model = "Tesla";
myCar.drive();
```

## 3. Difference between class and object?
A class is a logical template; an object is a physical instance with memory allocated.

## 4. What is autoboxing?
Autoboxing is the automatic conversion of a primitive type to its corresponding wrapper class object.  
**Example:**
```java
Integer i = 10;  // int 10 autoboxed to Integer
```

## 5. What is unboxing?
Unboxing is automatic conversion from a wrapper object to its primitive type.  
**Example:**
```java
Integer i = 10;
int num = i;  // unboxing
```

## 6. List all primitive wrapper classes in Java.
Boolean, Byte, Character, Short, Integer, Long, Float, Double.

## 7. What is the difference between `==` and `equals()` for objects?
`==` compares references (memory address). `equals()` compares content (if overridden).  
**Example:**
```java
String s1 = new String("Hi");
String s2 = new String("Hi");
System.out.println(s1 == s2);      // false
System.out.println(s1.equals(s2)); // true
```

## 8. What is the `hashCode()` contract with `equals()`?
If two objects are equal according to `equals()`, they must have the same `hashCode()`. Reverse not required.

## 9. Can we override `equals()` without overriding `hashCode()`?
Yes, but it breaks the contract, especially for hash-based collections (HashMap, HashSet).

## 10. What is method overloading?
Multiple methods in the same class with same name but different parameter lists.  
**Example:**
```java
void add(int a, int b) {}
void add(int a, int b, int c) {}
```

## 11. What is method overriding?
Subclass redefines a superclass method (same name, parameters, return type covariant).  
**Example:**
```java
class Animal { void sound() { System.out.println("Generic"); } }
class Dog extends Animal { void sound() { System.out.println("Bark"); } }
```

## 12. Difference between overloading and overriding?
Overloading = compile-time (static), same class, different params. Overriding = runtime (dynamic), different class, same signature.

## 13. What is the `static` keyword?
Belongs to the class, not instances. Shared across all objects.  
**Example:**
```java
class Counter { static int count = 0; }
```

## 14. Can a static method access non-static members?
No, because static context cannot refer to instance variables/methods directly.

## 15. What is the `final` keyword?
- `final` variable = constant (cannot reassign).  
- `final` method = cannot be overridden.  
- `final` class = cannot be subclassed.

## 16. What is an abstract class?
A class that cannot be instantiated, may contain abstract (unimplemented) methods.  
**Example:**
```java
abstract class Shape { abstract void draw(); }
```

## 17. What is an interface?
A contract that declares methods (by default public abstract) that implementing classes must define. Java 8+ allows default/static methods.

## 18. Difference between abstract class and interface? (Java 8+)
Abstract class: state (fields), constructors, single inheritance. Interface: multiple inheritance, no instance fields (except static final), default methods.

## 19. What is inheritance?
Mechanism where one class acquires fields/methods of another (extends).  
**Example:**
```java
class Vehicle { void move() {} }
class Bike extends Vehicle {}
```

## 20. What is multiple inheritance? Does Java support it?
Multiple inheritance of classes is not allowed (diamond problem). Interfaces allow multiple inheritance of type.

## 21. What is polymorphism?
Ability of an object to take many forms – compile-time (overloading) and runtime (overriding).

## 22. What is encapsulation?
Bundling data and methods into a single unit (class), hiding internal state via `private` fields and public getters/setters.

## 23. What is a constructor?
Special method that initializes an object when created. Same name as class, no return type.

## 24. Can a constructor be `private`?
Yes – used in singleton pattern to prevent external instantiation.

## 25. What is a singleton class?
Class that allows only one instance. Private constructor + static factory method.  
**Example:**
```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }
}
```

## 26. Difference between `String`, `StringBuilder`, `StringBuffer`?
- String: immutable, thread-safe (inefficient for changes).
- StringBuilder: mutable, not thread-safe, fastest.
- StringBuffer: mutable, thread-safe (synchronized), slower.

## 27. How to create an immutable class?
- Declare class `final`.
- Make all fields `private final`.
- No setter methods.
- Initialize fields via constructor.
- Return defensive copies for mutable fields.

## 28. What is the difference between checked and unchecked exceptions?
Checked: checked at compile-time (e.g., IOException). Unchecked: runtime exceptions (e.g., NullPointerException).

## 29. What is try-with-resources?
Automatically closes resources (AutoCloseable) after try block.  
**Example:**
```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    // use br
} catch (IOException e) {}
```

## 30. What is garbage collection?
Automatic memory management that deletes unreachable objects. `System.gc()` is a hint.

## 31. What are access modifiers?
- `private`: class only
- `default` (package-private): package only
- `protected`: package + subclasses
- `public`: everywhere

## 32. Can we override a static method?
No – it's method hiding. Subclass defines a static method with same signature.

## 33. What is the difference between `this` and `super`?
`this` refers to current object. `super` refers to parent class object.

## 34. What does the `instanceof` operator do?
Tests if an object is an instance of a specific class/interface.  
**Example:**
```java
if (obj instanceof String) {}
```

## 35. What is reflection?
API to inspect/modify classes, methods, fields at runtime, bypassing access checks.

## 36. What are annotations?
Metadata providing information about code (e.g., `@Override`, `@Deprecated`).

## 37. What is an enum?
A special class representing a fixed set of constants.  
**Example:**
```java
enum Day { MON, TUE, WED }
```

## 38. Difference between `ArrayList` and `LinkedList`?
ArrayList: dynamic array, fast random access, slow insert/delete in middle.  
LinkedList: doubly-linked list, fast insert/delete, slower random access.

## 39. Difference between `HashSet` and `TreeSet`?
HashSet: unordered, O(1) operations, uses hashCode(). TreeSet: sorted (natural/comparator), O(log n), implements NavigableSet.

## 40. Difference between `HashMap` and `Hashtable`?
HashMap: non-synchronized, allows null key/values. Hashtable: synchronized, no null.

## 41. What is `ConcurrentHashMap`?
Thread-safe map with high concurrency using segmentation (lock striping). Better than synchronized map.

## 42. Fail-fast vs fail-safe iterators?
Fail-fast: throws ConcurrentModificationException if collection modified during iteration (e.g., ArrayList iterator). Fail-safe: works on a copy (e.g., ConcurrentHashMap iterator).

## 43. Comparator vs Comparable?
Comparable: natural ordering (`compareTo`), inside the class. Comparator: custom ordering, external class/ lambda.

## 44. Thread vs Runnable?
Thread: class that can be extended (single inheritance). Runnable: functional interface, more flexible.  
**Example:**
```java
new Thread(() -> System.out.println("Run")).start();
```

## 45. What is synchronization?
Control access to shared resources by multiple threads using `synchronized` keyword.

## 46. What is `volatile` keyword?
Ensures variable reads/writes are done directly from main memory, not cached, providing visibility across threads.

## 47. What is `transient` keyword?
Prevents serialization of a field.  
**Example:**
```java
private transient String temp;
```

## 48. Difference between `sleep()` and `wait()`?
`sleep()`: Thread static method, does not release lock. `wait()`: Object method, called within synchronized, releases lock.

## 49. How does autoboxing affect method overloading?
Compiler chooses most specific method; may cause ambiguity.  
**Example:**
```java
void print(Integer i) {}   // Autoboxing preferred over varargs
void print(int... i) {}    // varargs least priority
```

## 50. What is the performance impact of autoboxing?
Creates unnecessary objects in loops or heavy operations – can cause memory overhead. Prefer primitives when performance critical.

## 51. Can we use `==` with wrapper objects?
Yes, but it compares references, not values (unless small cached range -128 to 127 for Integer).  
**Example:**
```java
Integer a = 100, b = 100; // same cached object -> true
Integer c = 200, d = 200; // new objects -> false
```

## 52. What is the `Integer` cache?
JVM caches Integer values from -128 to 127 (can be configured). Autoboxing uses cached objects.

## 53. What is object cloning?
Creating a copy of an object via `clone()` method (requires `Cloneable` interface).  
**Example:**
```java
class Person implements Cloneable {
    public Object clone() throws CloneNotSupportedException { return super.clone(); }
}
```

## 54. Shallow vs deep copy?
Shallow: copies primitive fields and references to objects. Deep: creates copies of referenced objects as well.

## 55. Can we have `static` constructors?
No – constructors are for instance initialization. Use static initialization block instead.

## 56. What is a static initializer block?
Runs once when class is loaded.  
**Example:**
```java
static { System.out.println("Class loaded"); }
```

## 57. What is an instance initializer block?
Runs before constructor code for each instance.  
**Example:**
```java
{ System.out.println("Instance init"); }
```

## 58. Difference between `String` literal and `new String()`?
Literal (`"Hi"`) uses string pool; `new String("Hi")` creates new object on heap, bypassing pool.

## 59. What is string interning?
Process of storing only one copy of each distinct string value in a pool (`intern()` method).

## 60. Can we use primitive types with generics? (e.g., `List<int>`)
No – generics require reference types. Use wrapper classes (`List<Integer>`).

## 61. How does autoboxing help with collections?
Primitives can be added to collections like `List<Integer>` directly: `list.add(5);` autoboxed to Integer.

## 62. What is the difference between `int` and `Integer`?
`int`: primitive, cannot be null, more efficient. `Integer`: wrapper object, can be null, needed for generics.

## 63. What is autoboxing in arithmetic expressions?
If an expression mixes primitive and wrapper types, unboxing occurs.  
**Example:**
```java
Integer i = 10;
int j = i + 5; // i unboxed to int
```

## 64. What is the default value of `Integer` object?
`null` (since it's a reference type). For `int`: 0.

## 65. Can we override a `private` method?
No – it's not visible in subclass. Attempting to define same method creates a new method.

## 66. What is the superclass of all classes?
`Object` class.

## 67. What methods does `Object` class provide?
`equals()`, `hashCode()`, `toString()`, `clone()`, `wait()`, `notify()`, `getClass()`, etc.

## 68. Purpose of `toString()` method?
Returns string representation of object. Override for meaningful output.  
**Example:**
```java
@Override public String toString() { return "Person{name=" + name + "}"; }
```

## 69. What is the difference between `final`, `finally`, `finalize()`?
- `final`: keyword for constants/methods/classes.
- `finally`: block guaranteed to execute after try/catch.
- `finalize()`: deprecated method called by GC before object collection.

## 70. What is a marker (tag) interface?
Empty interface like `Serializable`, `Cloneable` – indicates special capability.

## 71. What is the diamond problem? How solved?
Multiple inheritance of classes leads to ambiguity. Java solves by not allowing it; interfaces allow default methods with conflict resolution using overriding.

## 72. What is covariant return type?
Overriding method can return subclass of the original return type.  
**Example:**
```java
class A { A get() { return this; } }
class B extends A { B get() { return this; } }
```

## 73. What is the `strictfp` keyword?
Restricts floating-point calculations to ensure platform-independent results.

## 74. Can a class be `static`?
Only nested classes can be static – called static nested class. Top-level classes cannot be static.

## 75. What is the difference between static nested class and inner class?
Static nested: can be instantiated without outer class instance, cannot access instance fields of outer. Inner class (non-static) requires outer instance.

## 76. What is the difference between `assert` and exception?
`assert` is for debugging (disabled by default), throws AssertionError. Exceptions handle runtime failures.

## 77. What is a `record` class? (Java 14+)
Immutable data carrier with automatically generated constructor, `equals()`, `hashCode()`, `toString()`.  
**Example:**
```java
record Point(int x, int y) {}
```

## 78. What is the `Optional` class?
Container that may or may not hold a value. Avoids `NullPointerException`.  
**Example:**
```java
Optional<String> opt = Optional.ofNullable(getString());
opt.ifPresent(System.out::println);
```

## 79. What is method reference? (:: operator)
Shorthand for lambda expressions.  
**Example:**
```java
list.forEach(System.out::println);
```

## 80. What is the `var` keyword? (Java 10+)
Local variable type inference.  
**Example:**
```java
var list = new ArrayList<String>(); // inferred as ArrayList<String>
```

## 81. Difference between `Stream` and `Collection`?
Collection: data storage; Stream: functional-style pipeline for computations, does not store data.

## 82. What is the `java.util.Objects` class?
Utility class with methods `equals()`, `hashCode()`, `requireNonNull()` etc., for safer null handling.

## 83. Can autoboxing cause `NullPointerException`?
Yes – unboxing a null wrapper throws NPE.  
**Example:**
```java
Integer i = null;
int j = i; // NullPointerException
```

## 84. How does autoboxing behave in switch statements?
Switch allows wrapper types; unboxing happens automatically.  
**Example:**
```java
Integer day = 2;
switch(day) { case 1: ... } // day unboxed to int
```

## 85. What is the `Number` class?
Abstract superclass of numeric wrappers (Integer, Double, etc.).

## 86. What is the `Character` class methods for autoboxing?
`Character.isDigit()`, `isLetter()` operate on char; autoboxing allows `Character` object to be used directly.

## 87. Can we use autoboxing with ternary operator?
Yes, but careful with unintended unboxing.  
**Example:**
```java
boolean flag = true;
Integer i = flag ? 10 : null; // okay
int j = flag ? 10 : null; // NullPointerException when flag false
```

## 88. What happens when you compare `Integer` with `int` using `==`?
Integer unboxed to int, then compared by value.  
**Example:**
```java
Integer i = 128;
int j = 128;
System.out.println(i == j); // true (unboxing)
```

## 89. What is the performance cost of unboxing in loops?
Repeated unboxing in loops may degrade performance. Better to use primitive local variable.  
**Example:**
```java
Integer sum = 0;
for (int i = 0; i < 1000; i++) sum += i; // repeated unboxing/boxing
```

## 90. Why are wrapper classes immutable?
Primitive wrappers are immutable to ensure security, caching, and consistency (like String).

## 91. How to convert `String` to `int`?
`Integer.parseInt("123")` returns int; `Integer.valueOf("123")` returns Integer (autoboxing).

## 92. What is the difference between `valueOf` and `parseInt`?
`valueOf` returns Integer object (may use cache), `parseInt` returns primitive int.

## 93. Can we have a generic class with primitive type parameters?
No – use wrapper class. But with Java 10+ and `var` it's still wrapper.

## 94. What is the difference between `NumberFormatException` and `ArithmeticException`?
NumberFormatException: when parsing invalid number strings (e.g., `"abc"`). ArithmeticException: division by zero, overflow rules.

## 95. What is the `Math` class? Is it a wrapper?
Utility class with static math methods, not a wrapper.

## 96. What is the `Boolean` class's `parseBoolean` vs `valueOf`?
`parseBoolean("true")` returns primitive `true`; `Boolean.valueOf("true")` returns Boolean `TRUE` (cached).

## 97. How does autoboxing interact with method overloading and widening?
Widening (int to long) takes precedence over box/unboxing.  
**Example:**
```java
void m(long l) {}    // chosen if argument int
void m(Integer i) {} // second choice
```

## 98. What is the `@SafeVarargs` annotation?
Suppresses warnings on varargs methods when they don't perform unsafe heap pollution.

## 99. Can we use primitive arrays with autoboxing? e.g., `int[]` to `Integer[]`?
No automatic conversion – manually loop or use streams: `Arrays.stream(intArray).boxed().toArray(Integer[]::new)`.

## 100. What is the most important best practice with autoboxing?
Avoid in performance-critical loops; beware of `null` unboxing; prefer primitives in arithmetic; use `Objects.equals()` for wrapper comparison to avoid NPE.