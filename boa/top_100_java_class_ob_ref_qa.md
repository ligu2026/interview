# Top 100 Java Questions & Answers: Class, Object, Constructor, Primitive, Reference

## 1. What is a class in Java?
**Answer:** A class is a blueprint or template for creating objects. It defines properties (fields) and behaviors (methods) that objects of the class will have.

```java
public class Car {
    String color;  // property
    void drive() { // behavior
        System.out.println("Driving...");
    }
}
```

## 2. What is an object in Java?
**Answer:** An object is an instance of a class, created using the `new` keyword. It occupies memory and has actual values for the fields.

```java
Car myCar = new Car();
myCar.color = "Red";
myCar.drive();
```

## 3. What is a constructor?
**Answer:** A constructor is a special method invoked when an object is created. It initializes the object's state. Its name matches the class name and has no return type.

```java
public class Person {
    String name;
    Person(String n) { // constructor
        name = n;
    }
}
```

## 4. What is a default constructor?
**Answer:** A constructor with no parameters. If you don't define any constructor, Java provides a default no-argument constructor automatically.

```java
public class Student {
    // no constructor defined – default constructor provided
    String name;
}
Student s = new Student(); // default constructor called
```

## 5. Can a class have multiple constructors?
**Answer:** Yes, through constructor overloading – multiple constructors with different parameter lists.

```java
public class Rectangle {
    int width, height;
    Rectangle() { width = height = 0; }
    Rectangle(int w, int h) { width = w; height = h; }
}
```

## 6. What is primitive data type?
**Answer:** A primitive type is a basic data type built into Java, not an object. Examples: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`.

```java
int age = 25;
boolean isPass = true;
double salary = 45000.50;
```

## 7. What are reference types in Java?
**Answer:** Reference types store references (memory addresses) to objects. They include classes, interfaces, arrays, enums. A reference variable points to an object in heap memory.

```java
String str = "Hello"; // str is a reference to a String object
int[] arr = new int[5]; // arr is a reference to an array
```

## 8. How do primitive and reference types differ in memory?
**Answer:** Primitives hold values directly in the stack (or inside objects). Reference variables hold an address pointing to an object stored in the heap.

```java
int a = 10;        // value 10 stored in stack
String s = "Hi";   // reference s in stack, object "Hi" in heap
```

## 9. What is the default value of a primitive variable in a class field?
**Answer:** Primitives have default values (0 for numeric, false for boolean, '\u0000' for char). Reference types default to `null`.

```java
public class Test {
    int num;      // default 0
    boolean flag; // default false
    String text;  // default null
}
```

## 10. Can a constructor call another constructor of the same class?
**Answer:** Yes, using `this()` for constructor chaining. It must be the first statement in the constructor.

```java
public class Point {
    int x, y;
    Point() { this(0, 0); }
    Point(int x, int y) { this.x = x; this.y = y; }
}
```

## 11. What is the purpose of the `new` keyword?
**Answer:** `new` allocates memory for an object on the heap and returns a reference to it. It also invokes the constructor.

```java
Dog d = new Dog(); // memory allocated, constructor called
```

## 12. Can we have a constructor with `void` return type?
**Answer:** No. If you add `void`, it becomes a regular method, not a constructor. Java will treat it as a method named same as class.

```java
public class Example {
    void Example() { } // method, not constructor
    Example() { }      // constructor
}
```

## 13. What is constructor overloading?
**Answer:** Defining multiple constructors with different parameter lists. The appropriate one is called based on arguments passed during object creation.

```java
public class Employee {
    String name;
    int id;
    Employee() { name = "Unknown"; id = 0; }
    Employee(String n) { name = n; id = 0; }
    Employee(String n, int i) { name = n; id = i; }
}
```

## 14. Explain the difference between local variable and instance variable.
**Answer:** Local variables are declared inside methods/constructors, must be initialized before use, and exist only during method execution. Instance variables belong to objects, have default values, and exist as long as the object exists.

```java
public class Demo {
    int instanceVar; // default 0
    void method() {
        int localVar = 5; // must initialize
    }
}
```

## 15. What happens if you don't provide any constructor in a class?
**Answer:** Java provides a default no-argument constructor automatically. It initializes instance variables to default values.

```java
class Animal { }
Animal a = new Animal(); // default constructor works
```

## 16. Can we call a constructor directly after object creation?
**Answer:** No. Constructors are automatically called only once at object creation. To reinitialize, create a method like `init()`.

```java
Box b = new Box(); // constructor called automatically
// b.Box(); // ERROR: cannot call constructor directly
```

## 17. What is a copy constructor in Java?
**Answer:** Java doesn't have a built-in copy constructor, but you can create one manually – a constructor that takes an object of the same type and copies its fields.

```java
public class Color {
    int red, green, blue;
    Color(Color c) {
        this.red = c.red;
        this.green = c.green;
        this.blue = c.blue;
    }
}
```

## 18. How do you pass primitive types to a method?
**Answer:** Primitives are passed by value – a copy of the value is made. Changes inside the method do not affect the original variable.

```java
void change(int x) { x = 10; }
int a = 5;
change(a);
System.out.println(a); // prints 5, unchanged
```

## 19. How are reference types passed to a method?
**Answer:** References are passed by value – a copy of the reference (address) is passed. The copy points to the same object, so object state can be changed, but reassigning the parameter doesn't affect original reference.

```java
void modify(StringBuilder sb) { sb.append("!"); }
StringBuilder msg = new StringBuilder("Hi");
modify(msg);
System.out.println(msg); // prints "Hi!"
```

## 20. What is the difference between `==` and `.equals()` for objects?
**Answer:** `==` compares reference equality (same object?). `.equals()` compares content/logical equality (can be overridden).

```java
String s1 = new String("Hi");
String s2 = new String("Hi");
System.out.println(s1 == s2);      // false
System.out.println(s1.equals(s2)); // true
```

## 21. Can a constructor be private? Why?
**Answer:** Yes. A private constructor prevents instantiation from outside the class, used in singleton pattern or utility classes.

```java
public class Singleton {
    private static Singleton instance;
    private Singleton() { } // private constructor
    public static Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }
}
```

## 22. What is the `this` keyword?
**Answer:** `this` refers to the current object instance. It is used to access instance variables, call other constructors (`this()`), or pass current object as argument.

```java
public class Person {
    String name;
    Person(String name) { this.name = name; }
}
```

## 23. What is the `static` keyword in relation to classes?
**Answer:** `static` members belong to the class rather than any instance. They can be accessed without creating an object.

```java
public class Counter {
    static int count = 0;
    Counter() { count++; }
}
```

## 24. Can a class have a static constructor? (No, Java has no static constructors)
**Answer:** No, but you can use a static initializer block to initialize static fields.

```java
class Config {
    static String version;
    static {
        version = "1.0";
    }
}
```

## 25. What is the purpose of the `super` keyword in constructors?
**Answer:** `super()` calls the parent class constructor. It must be the first statement in a constructor. If omitted, Java inserts `super()` implicitly.

```java
class Parent { Parent() { System.out.println("Parent"); } }
class Child extends Parent {
    Child() { super(); // optional – implicit
        System.out.println("Child");
    }
}
```

## 26. Explain constructor chaining.
**Answer:** Constructor chaining is the process where a constructor calls another constructor (of same or parent class) using `this()` or `super()`. It ensures initialization hierarchy.

```java
class A {
    A() { System.out.println("A"); }
}
class B extends A {
    B() { this(5); System.out.println("B"); }
    B(int x) { super(); System.out.println("B int"); }
}
```

## 27. What is an inner class? Can it access outer class instance variables?
**Answer:** An inner class (non-static nested class) is defined inside another class. It has access to all members (including private) of the outer class.

```java
public class Outer {
    private int data = 10;
    class Inner {
        void display() { System.out.println(data); } // accessible
    }
}
```

## 28. What is an anonymous class in Java?
**Answer:** An anonymous class is a class without a name, defined and instantiated in a single expression. Often used for implementing interfaces or extending classes on the fly.

```java
Runnable r = new Runnable() {
    public void run() { System.out.println("Running"); }
};
```

## 29. How do you create an array of objects?
**Answer:** First create the array (reference array), then create each object individually.

```java
Book[] library = new Book[10]; // array of references (null)
for (int i = 0; i < library.length; i++) {
    library[i] = new Book();
}
```

## 30. What is the `null` literal?
**Answer:** `null` is a special literal that represents "no object". It can be assigned to any reference variable. Dereferencing `null` causes `NullPointerException`.

```java
String s = null;
// s.length(); // Throws NullPointerException
```

## 31. Can a primitive be `null`?
**Answer:** No. Primitives cannot be `null`. They always hold a value (default 0, false, etc.). `int i = null;` is a compile error.

## 32. What is a wrapper class? Give an example.
**Answer:** Wrapper classes provide object representation for primitive types (e.g., `Integer` for `int`, `Boolean` for `boolean`). They are used in collections and where objects are required.

```java
int num = 10;
Integer wrapped = Integer.valueOf(num); // or autoboxing: Integer wrapped = num;
```

## 33. What is autoboxing and unboxing?
**Answer:** Autoboxing is automatic conversion from primitive to wrapper type; unboxing is the reverse. Java does it implicitly where needed.

```java
Integer i = 5; // autoboxing (int to Integer)
int j = i;     // unboxing (Integer to int)
```

## 34. Can a constructor throw an exception?
**Answer:** Yes. Constructors can declare `throws` and throw checked or unchecked exceptions.

```java
public class FileReader {
    public FileReader(String filename) throws FileNotFoundException {
        // ...
    }
}
```

## 35. What is the difference between method overloading and constructor overloading?
**Answer:** Both have similar rules: different parameters. Constructor overloading provides multiple ways to create objects; method overloading provides multiple behaviors with same name.

```java
// Constructor overloading
Point(int x, int y) { }
Point(double x, double y) { }
// Method overloading
void print(int a) { }
void print(String s) { }
```

## 36. How many objects are created? `Student s1 = new Student(); Student s2 = s1;`
**Answer:** One object created. `s1` and `s2` are two references pointing to the same object.

## 37. What is the garbage collector? How does it relate to references?
**Answer:** Garbage collector automatically deletes objects that are no longer reachable through any reference. When no references point to an object, it becomes eligible for GC.

```java
Student s = new Student();
s = null; // object becomes eligible for GC (if no other references)
```

## 38. Can you declare a constructor as `final`?
**Answer:** No. Constructors cannot be `static`, `final`, `abstract`, or `synchronized`. They are not inherited; they are invoked.

## 39. Explain the difference between `int[] arr` and `int arr[]`.
**Answer:** No difference – both declare an array variable of type `int`. `int[] arr` is the preferred style.

```java
int[] arr1;  // recommended
int arr2[];  // also valid (C-style)
```

## 40. What is the purpose of the `final` keyword with a reference variable?
**Answer:** `final` reference cannot be reassigned to point to a different object. But the object's state can still be modified.

```java
final StringBuilder sb = new StringBuilder("Hi");
sb.append("!"); // allowed
// sb = new StringBuilder(); // ERROR: cannot reassign
```

## 41. Can we have an abstract class with no abstract methods? What about constructors?
**Answer:** Yes, an abstract class can have concrete methods. It can have constructors, called when subclasses are instantiated.

```java
abstract class Base {
    Base() { System.out.println("Base constructed"); }
    void concrete() { }
}
```

## 42. What is a no-argument constructor? Is it the same as default constructor?
**Answer:** A no-argument constructor is any constructor with no parameters. The default constructor is the one Java provides only if you define none. A user-defined no-argument constructor is not "default" but is also a no-arg constructor.

```java
class Demo {
    Demo() { } // user-defined no-argument constructor
}
```

## 43. How do you initialize instance variables in Java?
**Answer:** Using constructors, initializer blocks, or direct assignment at declaration.

```java
public class Init {
    int a = 5;               // direct
    int b;                   // then in constructor
    int c;
    { c = 7; }               // initializer block
    Init() { b = 10; }
}
```

## 44. What is the output of: `new Child()` if `Parent` has parameterized constructor only?
**Answer:** Compilation error because `Child()` constructor implicitly calls `super()` which doesn't exist. Fix: explicit `super(parameters)` or add no-arg constructor in `Parent`.

## 45. Can a constructor be generic?
**Answer:** Yes, a constructor can declare its own type parameters (generic constructor), even if the class is not generic.

```java
public class Box {
    public <T> Box(T item) {
        System.out.println(item.getClass().getName());
    }
}
```

## 46. What is the `instanceof` operator?
**Answer:** `instanceof` tests whether an object reference is an instance of a particular class or interface type.

```java
String s = "Hello";
boolean b = s instanceof String; // true
```

## 47. Explain stack vs heap memory regarding primitives and objects.
**Answer:** Primitives and object references are stored in stack (local variables). Actual objects are stored in heap. Instance variables (primitives or references) are stored inside the object on the heap.

## 48. Can you call one constructor from another using `this()`? Where must it appear?
**Answer:** Yes. The `this()` call must be the first statement in the constructor. Otherwise, compilation error.

```java
public class Time {
    int h, m;
    Time() { this(0, 0); }
    Time(int h) { this(h, 0); } // first statement
    Time(int h, int m) { this.h = h; this.m = m; }
}
```

## 49. What is a local class?
**Answer:** A class defined inside a method or block. It has access to final or effectively final local variables of the enclosing method.

```java
void outerMethod() {
    class LocalClass {
        void inner() { System.out.println("Inside local class"); }
    }
    LocalClass lc = new LocalClass();
    lc.inner();
}
```

## 50. Explain method local inner class access to method parameters.
**Answer:** It can only access parameters/variables that are final or effectively final (not changed after initialization). Since Java 8, effectively final is allowed.

```java
void calculate(int x) { // x is effectively final
    class Calculator {
        void show() { System.out.println(x); } // allowed
    }
}
```

## 51. What is a static nested class? How is it different from inner class?
**Answer:** Static nested class is a class defined with `static` keyword inside another class. It doesn't have access to instance members of outer class (only static members). Inner class (non-static) has access to all outer members.

```java
class Outer {
    static class StaticNested { }
    class Inner { }
}
```

## 52. Can an object be created without using `new` keyword? List ways.
**Answer:** Yes: using reflection (`Class.newInstance()`), cloning (`clone()`), deserialization, or factory methods (e.g., `Integer.valueOf()`).

```java
// Reflection
MyClass obj = MyClass.class.newInstance(); // deprecated, use getDeclaredConstructor()
// Cloning
MyClass obj2 = (MyClass) obj.clone();
```

## 53. What is the difference between shallow copy and deep copy?
**Answer:** Shallow copy copies references to objects (not the objects themselves). Deep copy creates new instances of referenced objects.

```java
class Address { String city; }
class Person {
    String name;
    Address addr;
    // shallow: copy addr reference
    // deep: new Address(addr.city)
}
```

## 54. Can a constructor be overloaded with different return types (since it has no return type)?
**Answer:** Constructor overloading is based only on parameters, not return type (there is no return type at all). So you cannot have two constructors differing only by a hypothetical return type.

## 55. What is the purpose of a `static initializer` block?
**Answer:** It runs once when the class is first loaded, used to initialize static fields or perform setup.

```java
class Database {
    static Connection conn;
    static {
        // load driver, create connection
    }
}
```

## 56. Can we have an instance initializer block? When does it run?
**Answer:** Yes, a non-static initializer block runs before the constructor body every time an object is created. It's used to share initialization code across constructors.

```java
class Demo {
    int x;
    { x = 5; } // instance initializer
    Demo() { }
    Demo(int y) { } // initializer runs first
}
```

## 57. What is the order of initialization: instance init block, constructor, field initializer?
**Answer:** Field initializers and instance init blocks run in the order they appear, before constructor body.

```java
class Order {
    int a = 1;          // 1
    { a = 2; }          // 2
    Order() { a = 3; }  // 3 (last)
}
```

## 58. Explain the concept of reference variable casting.
**Answer:** Upcasting (assigning subclass reference to superclass variable) is automatic. Downcasting (superclass to subclass) requires explicit cast and may throw `ClassCastException`.

```java
Animal a = new Dog();        // upcast automatic
Dog d = (Dog) a;             // downcast explicit
// Cat c = (Cat) a;          // runtime ClassCastException
```

## 59. What is the Java `Object` class? Which methods does it have?
**Answer:** `Object` is the root of all classes. Important methods: `equals()`, `hashCode()`, `toString()`, `getClass()`, `clone()`, `finalize()`, `wait()`, `notify()`, `notifyAll()`.

```java
Object obj = new Object();
System.out.println(obj.toString()); // e.g., java.lang.Object@15db9742
```

## 60. What is the purpose of `toString()` method? Should you override it?
**Answer:** `toString()` returns a string representation of an object. Override it to provide meaningful output for debugging/logging.

```java
public class Point {
    int x, y;
    @Override
    public String toString() { return "(" + x + "," + y + ")"; }
}
```

## 61. How does `equals()` and `hashCode()` contract work?
**Answer:** If two objects are equal according to `equals()`, they must have same `hashCode()`. If `hashCode()` is equal, objects may or may not be equal. Always override both together.

```java
@Override
public boolean equals(Object o) { ... }
@Override
public int hashCode() { return Objects.hash(x, y); }
```

## 62. What is the difference between `String` and `StringBuilder` regarding references?
**Answer:** `String` is immutable – any modification creates a new object. `StringBuilder` is mutable – modifies the same object, so references see the change.

```java
String s = "Hi";
StringBuilder sb = new StringBuilder("Hi");
s.concat("!");        // new object, s unchanged
sb.append("!");       // sb object changed
```

## 63. Can we have a constructor that returns a value?
**Answer:** No. Constructor does not have a return type. It cannot return a value. It implicitly returns the created object.

## 64. What is a default constructor's visibility?
**Answer:** The default constructor has the same access modifier as the class. If class is `public`, default constructor is `public`; if class is package-private, default constructor is package-private.

## 65. Explain the diamond problem with multiple inheritance and how Java avoids it.
**Answer:** Java doesn't support multiple inheritance of classes to avoid the diamond problem (ambiguity when two parent classes have same method). Interfaces can have default methods, but conflict is resolved by overriding or explicit choice.

```java
interface A { default void m() { } }
interface B { default void m() { } }
class C implements A, B { // must override m()
    public void m() { A.super.m(); } // choose one
}
```

## 66. What is a `record` in Java (since Java 14/16)? How does it relate to classes?
**Answer:** A record is a special immutable class that acts as a transparent data carrier. It automatically provides constructor, equals, hashCode, toString.

```java
record Person(String name, int age) { }
Person p = new Person("Alice", 30);
System.out.println(p.name()); // accessor
```

## 67. Can a constructor invoke a non-final method? Any risk?
**Answer:** Yes, but avoid calling overridable methods from constructor, because the subclass might not be fully initialized yet, leading to unexpected behavior.

```java
class Base {
    Base() { show(); } // dangerous if show is overridden
    void show() { }
}
```

## 68. How do you prevent a class from being instantiated?
**Answer:** Make the constructor private, or declare the class `abstract` (though abstract can be subclassed and instantiated via subclass).

```java
public class Utility {
    private Utility() { throw new AssertionError(); }
}
```

## 69. What is the difference between `String[] args` and `String... args` in main method?
**Answer:** `String... args` is varargs, allows passing zero or more strings. Both can be used in `main`. They are almost same; varargs is syntactic sugar for array.

```java
public static void main(String... args) { } // valid
```

## 70. What is the default value of a reference variable declared inside a method?
**Answer:** Local reference variables must be initialized before use; there is no default value. The compiler enforces this.

```java
void method() {
    String s; // no default, cannot use before initialization
    // s.length(); // compile error
}
```

## 71. Can we have an array of primitive types? What is stored?
**Answer:** Yes, array of primitives stores actual values, not references. For `int[]`, each element is an `int` value.

```java
int[] nums = new int[3]; // nums[0] = 0, nums[1]=0...
nums[0] = 10;
```

## 72. What is the difference between `length` property of array and `length()` method of String?
**Answer:** Array has `length` (field); String has `length()` (method). Both return number of elements/characters.

```java
int[] arr = {1,2,3};
System.out.println(arr.length);    // 3
String str = "Hi";
System.out.println(str.length());  // 2
```

## 73. Why are primitive types not objects? Efficiency benefit?
**Answer:** Primitives are stored directly in stack, more memory-efficient and faster. Objects have overhead (header, reference, vtable). Java keeps primitives for performance.

## 74. What is the meaning of "pass-by-value" in Java for references?
**Answer:** The reference value (memory address) is passed by value, not the object itself. The method gets a copy of the reference, which still points to same object.

```java
void reassign(StringBuilder sb) { sb = new StringBuilder("new"); } // doesn't affect caller
```

## 75. How do you create an immutable class?
**Answer:** Make class `final`, fields `final` and private, no setters, no methods modify state, if fields are mutable return defensive copies.

```java
public final class ImmutablePoint {
    private final int x, y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    // no setters
}
```

## 76. Can a constructor be synchronized?
**Answer:** No, it's redundant because constructor runs in a single thread context until object is published. Use synchronized blocks inside if needed.

## 77. What is an enumeration (enum) in Java? Is it a class?
**Answer:** `enum` is a special class type used to define constants. It implicitly extends `java.lang.Enum` and cannot be instantiated with `new`. It can have fields, constructors (private), methods.

```java
enum Day { MON, TUE, WED; }
Day d = Day.MON;
```

## 78. Explain reference types: strong, soft, weak, phantom references.
**Answer:** Strong (normal). Soft – garbage collected only when memory low. Weak – garbage collected eagerly. Phantom – used for pre-mortem cleanup. Part of `java.lang.ref`.

```java
WeakReference<Object> wr = new WeakReference<>(obj);
```

## 79. Can a constructor access static members?
**Answer:** Yes. Constructors can access both static and instance members (instance members belong to the object being constructed).

## 80. What is the purpose of `finalize()` method? Is it deprecated?
**Answer:** `finalize()` was called before garbage collection, but it's deprecated (Java 9) and unreliable. Use `Cleaner` or `try-with-resources`.

## 81. Explain object composition vs inheritance.
**Answer:** Composition (has-a) uses references to other objects; inheritance (is-a) extends a class. Composition gives more flexibility and is preferred over inheritance for code reuse.

```java
class Engine { }
class Car {
    private Engine engine; // composition
}
```

## 82. What is the `Class` object? How to obtain it?
**Answer:** Every loaded class has a `Class` instance representing its metadata. Obtain via `.class`, `getClass()`, or `Class.forName()`.

```java
Class<?> c1 = String.class;
Class<?> c2 = "hello".getClass();
Class<?> c3 = Class.forName("java.lang.String");
```

## 83. Can a constructor be abstract?
**Answer:** No. Abstract methods have no body and must be overridden; constructors cannot be overridden, so abstract constructor is meaningless.

## 84. What is the difference between `this` and `super` in constructors?
**Answer:** `this()` calls another constructor of same class; `super()` calls parent constructor. Both must be first statement, cannot coexist.

## 85. What is a "package-private" access level? How does it relate to classes?
**Answer:** No explicit modifier. Accessible only within the same package. Top-level classes can be `public` or package-private (default).

```java
class PackagePrivateClass { } // visible only in package
```

## 86. Can we overload a constructor based on order of parameters? Give example.
**Answer:** Yes, as long as the parameter types differ in order.

```java
Person(String name, int age) { }
Person(int age, String name) { } // valid overload
```

## 87. What are annotation types? How can they be used with classes?
**Answer:** Annotations provide metadata. They can be placed on classes, constructors, fields etc. Example: `@Override`, `@Deprecated`.

```java
@Deprecated
class OldClass { }
```

## 88. What is a `sealed` class (Java 17)? How does it affect object creation?
**Answer:** Sealed class restricts which subclasses can extend it. Use `permits` clause. Constructors work normally.

```java
sealed class Shape permits Circle, Rectangle { }
final class Circle extends Shape { }
```

## 89. What happens when you assign a primitive to a wrapper and vice versa? (autoboxing memory)
**Answer:** Autoboxing creates a wrapper object on heap (cached for small range). Unboxing extracts primitive value.

```java
Integer i = 100; // uses Integer.valueOf(100) – cached for -128..127
int j = i;       // unboxing: i.intValue()
```

## 90. Explain the difference between `int[] a = {1,2,3}` and `int[] a = new int[]{1,2,3}`.
**Answer:** The first is array initializer syntax allowed only in declaration. The second can be used in any expression (e.g., method argument). Both create same array object.

```java
int[] a = {1,2,3};               // declaration only
int[] b = new int[]{1,2,3};      // anywhere
method(new int[]{1,2,3});        // valid
```

## 91. Can a class contain itself as a field? What is the effect?
**Answer:** Yes, it can contain a reference to an object of the same type (e.g., linked list node). This does not cause infinite memory because the reference is just a pointer.

```java
class Node {
    int data;
    Node next; // self-reference
}
```

## 92. What is the `var` keyword (local variable type inference)? Does it work with primitive and reference?
**Answer:** `var` can infer both primitive and reference types from initializer. It is not a dynamic type; type is fixed at compile time.

```java
var count = 10;        // int
var name = "Java";     // String
var list = new ArrayList<String>(); // ArrayList<String>
```

## 93. What is a lambda expression? How does it relate to object references?
**Answer:** Lambda expression is an instance of a functional interface – a shorthand for anonymous class. It captures references to effectively final variables.

```java
Runnable r = () -> System.out.println("Run");
// equivalent to new Runnable() { public void run() { ... } }
```

## 94. What is the difference between `Object.clone()` and copy constructor?
**Answer:** `clone()` is a protected method of Object; requires implementing `Cloneable`. Copy constructor is more flexible, easier to understand, and doesn't throw checked exceptions.

```java
// copy constructor
public MyClass(MyClass other) { this.field = other.field; }
```

## 95. Can a constructor be `native`?
**Answer:** Yes, a native constructor is possible for JNI (calling C/C++ code). But it's obscure.

## 96. What is a default method in interface? How does it affect implementing class constructors?
**Answer:** Default methods have body in interface. They don't affect constructors. Implementing class can override or use default implementation.

```java
interface Vehicle {
    default void start() { System.out.println("Start"); }
}
```

## 97. Why is it discouraged to call overridable methods in constructor?
**Answer:** Subclass overriding method may rely on subclass state not yet initialized (constructor of subclass hasn't run). This leads to unpredictable bugs.

```java
class Base {
    Base() { print(); }
    void print() { System.out.println("Base"); }
}
class Derived extends Base {
    int x = 5;
    void print() { System.out.println(x); } // prints 0, not 5
}
```

## 98. What is the purpose of `Objects.requireNonNull()`?
**Answer:** Utility method to validate that a reference is not null, throwing `NullPointerException` if it is. Useful in constructors.

```java
public Person(String name) {
    this.name = Objects.requireNonNull(name, "Name must not be null");
}
```

## 99. Can a constructor be `strictfp`?
**Answer:** Yes, `strictfp` ensures floating-point consistency across platforms. It can be applied to constructors.

## 100. How does method overloading work with primitive widening and boxing?
**Answer:** Java chooses the most specific method. Widening (int→long) is preferred over boxing (int→Integer) and varargs.

```java
void show(int i) { }
void show(Integer i) { }
show(10); // calls int version (widening)
```