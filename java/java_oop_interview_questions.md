# Top 50 Java OOP Interview Questions & Answers

---

## 1. What is Object-Oriented Programming (OOP)?

OOP is a programming paradigm that organizes software design around **objects** — data structures that bundle state (fields) and behavior (methods). Java's four pillars are **Encapsulation, Inheritance, Polymorphism, and Abstraction**.

---

## 2. What is a Class vs an Object?

- A **class** is a blueprint/template.
- An **object** is an instance of that class.

```java
class Car {
    String brand;
    void drive() { System.out.println(brand + " is driving"); }
}

Car myCar = new Car();   // myCar is an object
myCar.brand = "Toyota";
myCar.drive();           // Toyota is driving
```

---

## 3. What are the four pillars of OOP?

| Pillar | Description |
|---|---|
| Encapsulation | Bundling data + methods; restricting direct access |
| Inheritance | A class acquires properties of another class |
| Polymorphism | One interface, many implementations |
| Abstraction | Hiding implementation details, exposing only essentials |

---

## 4. What is Encapsulation?

Encapsulation wraps data (fields) with controlled access via `private` fields and `public` getters/setters.

```java
class BankAccount {
    private double balance;

    public double getBalance() { return balance; }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }
}
```

---

## 5. What is Inheritance?

Inheritance allows a child class to reuse fields and methods of a parent class using the `extends` keyword.

```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Woof!"); }
}

Dog d = new Dog();
d.eat();   // inherited
d.bark();  // own method
```

---

## 6. What is Polymorphism?

Polymorphism lets a single interface represent different underlying forms. Java supports:
- **Compile-time** (method overloading)
- **Runtime** (method overriding)

```java
class Shape {
    void draw() { System.out.println("Drawing shape"); }
}

class Circle extends Shape {
    @Override
    void draw() { System.out.println("Drawing circle"); }
}

Shape s = new Circle();  // runtime polymorphism
s.draw();                // Drawing circle
```

---

## 7. What is Abstraction?

Abstraction hides complex implementation and exposes only what is necessary, via `abstract` classes or `interface`.

```java
abstract class Vehicle {
    abstract void startEngine();  // no implementation

    void refuel() { System.out.println("Refueling..."); }
}

class Bike extends Vehicle {
    @Override
    void startEngine() { System.out.println("Bike engine started"); }
}
```

---

## 8. What is an Abstract Class?

An abstract class cannot be instantiated. It may have both abstract and concrete methods.

```java
abstract class Shape {
    abstract double area();       // must override

    void display() {              // concrete — can use as-is
        System.out.println("Area: " + area());
    }
}

class Rectangle extends Shape {
    double width, height;
    Rectangle(double w, double h) { width = w; height = h; }

    @Override
    double area() { return width * height; }
}
```

---

## 9. What is an Interface?

An interface is a pure contract — all methods are implicitly `public abstract` (prior to Java 8). From Java 8 onward, `default` and `static` methods are allowed.

```java
interface Flyable {
    void fly();
    default void land() { System.out.println("Landing..."); }
}

class Bird implements Flyable {
    @Override
    public void fly() { System.out.println("Bird flying"); }
}
```

---

## 10. Abstract Class vs Interface — Key Differences

| Feature | Abstract Class | Interface |
|---|---|---|
| Instantiation | No | No |
| Multiple inheritance | No | Yes |
| Constructor | Yes | No |
| Fields | Any | `public static final` only |
| Method types | Abstract + concrete | Abstract + default + static |

---

## 11. What is Method Overloading?

Same method name, different parameter lists (compile-time polymorphism).

```java
class Calculator {
    int add(int a, int b)          { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c)   { return a + b + c; }
}
```

---

## 12. What is Method Overriding?

A subclass provides its own implementation of a method already defined in the parent class (runtime polymorphism). Rules:
- Same name, return type, and parameters
- Cannot reduce visibility
- Use `@Override` annotation

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}

class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow"); }
}
```

---

## 13. What is the `super` keyword?

`super` refers to the parent class. Used to call parent constructors or overridden methods.

```java
class Person {
    String name;
    Person(String name) { this.name = name; }
}

class Employee extends Person {
    int id;
    Employee(String name, int id) {
        super(name);   // call parent constructor
        this.id = id;
    }
}
```

---

## 14. What is the `this` keyword?

`this` refers to the current object instance. Used to resolve field-parameter name conflicts or chain constructors.

```java
class Point {
    int x, y;

    Point(int x, int y) {
        this.x = x;  // distinguishes field from parameter
        this.y = y;
    }
}
```

---

## 15. What is a Constructor?

A constructor initializes a newly created object. It has the same name as the class and no return type.

```java
class Book {
    String title;

    // Default constructor
    Book() { title = "Unknown"; }

    // Parameterized constructor
    Book(String title) { this.title = title; }
}
```

---

## 16. What is Constructor Overloading?

Multiple constructors with different parameter lists in the same class.

```java
class Rectangle {
    int width, height;

    Rectangle()              { this(1, 1); }       // calls next constructor
    Rectangle(int side)      { this(side, side); }
    Rectangle(int w, int h)  { width = w; height = h; }
}
```

---

## 17. Can constructors be inherited?

**No.** Constructors are not inherited. However, a subclass constructor implicitly calls `super()` (the no-arg parent constructor) unless another `super(...)` call is explicitly provided.

---

## 18. What is a Copy Constructor?

A constructor that initializes an object using another object of the same class.

```java
class Student {
    String name;
    int age;

    Student(Student s) {
        this.name = s.name;
        this.age  = s.age;
    }
}
```

---

## 19. What is the difference between `==` and `.equals()`?

- `==` compares **references** (memory addresses).
- `.equals()` compares **content** (can be overridden).

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);       // false — different objects
System.out.println(a.equals(b));  // true  — same content
```

---

## 20. What is `hashCode()` and why should it be overridden with `equals()`?

`hashCode()` returns an integer hash for the object. The **contract**: if `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must also be true. Collections like `HashMap` rely on this.

```java
class Point {
    int x, y;

    @Override
    public boolean equals(Object o) {
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return 31 * x + y;
    }
}
```

---

## 21. What is a Static method / field?

`static` members belong to the **class**, not to any instance.

```java
class Counter {
    static int count = 0;

    Counter() { count++; }

    static int getCount() { return count; }
}

new Counter(); new Counter();
System.out.println(Counter.getCount()); // 2
```

---

## 22. What is the `final` keyword?

| Context | Effect |
|---|---|
| `final` variable | Value cannot change (constant) |
| `final` method | Cannot be overridden |
| `final` class | Cannot be subclassed |

```java
final class Immutable { }         // cannot extend
// class Sub extends Immutable { } // ❌ compile error

class Parent {
    final void show() { System.out.println("Parent"); }
}
```

---

## 23. What is an Immutable Class?

An object whose state cannot change after construction. Rules:
1. `final` class
2. `private final` fields
3. No setters
4. Deep-copy mutable fields

```java
final class Money {
    private final double amount;
    private final String currency;

    Money(double amount, String currency) {
        this.amount   = amount;
        this.currency = currency;
    }

    public double getAmount()   { return amount; }
    public String getCurrency() { return currency; }
}
```

---

## 24. What is a Singleton Pattern?

Ensures a class has only **one** instance.

```java
class Singleton {
    private static Singleton instance;

    private Singleton() {}   // private constructor

    public static Singleton getInstance() {
        if (instance == null)
            instance = new Singleton();
        return instance;
    }
}
```

---

## 25. What is an Inner Class?

A class defined within another class.

```java
class Outer {
    int x = 10;

    class Inner {
        void show() { System.out.println("x = " + x); }
    }
}

Outer o = new Outer();
Outer.Inner i = o.new Inner();
i.show();  // x = 10
```

---

## 26. What is a Static Nested Class?

A `static` class inside another class. Does **not** need an outer class instance.

```java
class Outer {
    static class Nested {
        void greet() { System.out.println("Hello from Nested"); }
    }
}

Outer.Nested n = new Outer.Nested();
n.greet();
```

---

## 27. What is an Anonymous Class?

A class without a name, defined and instantiated in one expression — typically for one-time use.

```java
interface Greeter { void greet(); }

Greeter g = new Greeter() {
    @Override
    public void greet() { System.out.println("Hello!"); }
};

g.greet();
```

---

## 28. What is a Functional Interface?

An interface with exactly **one abstract method** — usable as a lambda target. Annotated with `@FunctionalInterface`.

```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b);
}

MathOperation add = (a, b) -> a + b;
System.out.println(add.operate(3, 5));  // 8
```

---

## 29. What is Upcasting and Downcasting?

- **Upcasting**: child → parent (implicit, safe).
- **Downcasting**: parent → child (explicit, may throw `ClassCastException`).

```java
Animal a = new Dog();      // upcasting — implicit
Dog d = (Dog) a;           // downcasting — explicit

if (a instanceof Dog) {    // safe check
    Dog d2 = (Dog) a;
}
```

---

## 30. What is the `instanceof` operator?

Checks whether an object is an instance of a class or interface.

```java
Object obj = "Hello";
System.out.println(obj instanceof String);   // true
System.out.println(obj instanceof Integer);  // false
```

Java 16+ pattern matching:
```java
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());  // s is already cast
}
```

---

## 31. What is Multiple Inheritance and why doesn't Java support it for classes?

Multiple inheritance (a class inheriting from multiple classes) causes the **Diamond Problem** — ambiguity about which parent method to use. Java resolves this by allowing multiple **interface** implementation instead.

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void hello() { A.super.hello(); }  // must resolve explicitly
}
```

---

## 32. What is method hiding (static methods and inheritance)?

Static methods are **hidden**, not overridden. The method called depends on the reference type, not the actual object type.

```java
class Parent {
    static void display() { System.out.println("Parent"); }
}

class Child extends Parent {
    static void display() { System.out.println("Child"); }
}

Parent p = new Child();
p.display();  // Parent — static binding, uses reference type
```

---

## 33. What is the Object class?

Every Java class implicitly extends `java.lang.Object`. Key methods:
- `toString()`, `equals()`, `hashCode()`
- `clone()`, `finalize()` *(deprecated)*
- `getClass()`, `wait()`, `notify()`, `notifyAll()`

---

## 34. What is `toString()`?

Returns a string representation of an object. Override it for meaningful output.

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public String toString() {
        return "Point(" + x + ", " + y + ")";
    }
}

System.out.println(new Point(3, 4));  // Point(3, 4)
```

---

## 35. What is `clone()` and what are shallow vs deep copy?

- **Shallow copy**: copies field values — object references are shared.
- **Deep copy**: recursively copies referenced objects.

```java
class Address implements Cloneable {
    String city;
    Address(String city) { this.city = city; }
}

class Person implements Cloneable {
    String name;
    Address address;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person copy = (Person) super.clone();      // shallow
        copy.address = new Address(address.city);  // deep — new object
        return copy;
    }
}
```

---

## 36. What is an Enum in Java?

A special class representing a fixed set of constants, with full OOP support.

```java
enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN;

    public boolean isWeekend() {
        return this == SAT || this == SUN;
    }
}

System.out.println(Day.SAT.isWeekend());  // true
```

---

## 37. What is a Generic class?

Generics enable type-safe classes and methods without casting.

```java
class Box<T> {
    private T value;
    Box(T value) { this.value = value; }
    T get() { return value; }
}

Box<String>  s = new Box<>("Hello");
Box<Integer> i = new Box<>(42);
```

---

## 38. What is a Bounded Type Parameter?

Restricts a generic type to a specific hierarchy.

```java
class NumberBox<T extends Number> {
    T value;
    NumberBox(T v) { this.value = v; }
    double doubled() { return value.doubleValue() * 2; }
}

NumberBox<Integer> box = new NumberBox<>(10);
System.out.println(box.doubled());  // 20.0
```

---

## 39. What is Covariant Return Type?

An overriding method can return a subtype of the return type declared in the parent.

```java
class Animal { Animal create() { return new Animal(); } }

class Dog extends Animal {
    @Override
    Dog create() { return new Dog(); }  // Dog is a subtype of Animal
}
```

---

## 40. What is the difference between Composition and Inheritance?

- **Inheritance** ("is-a"): `Dog is-a Animal`.
- **Composition** ("has-a"): `Car has-a Engine`. Prefer composition for flexibility and looser coupling.

```java
class Engine {
    void start() { System.out.println("Engine started"); }
}

class Car {
    private Engine engine = new Engine();  // composition

    void start() { engine.start(); }
}
```

---

## 41. What is Aggregation vs Composition?

| | Aggregation | Composition |
|---|---|---|
| Relationship | Weak "has-a" | Strong "has-a" |
| Lifecycle | Child can exist independently | Child dies with parent |
| Example | University → Department | House → Room |

```java
// Composition — Room cannot exist without House
class House {
    private final Room room = new Room();
}

// Aggregation — Student can exist without Classroom
class Classroom {
    private Student student;  // passed in, not owned
}
```

---

## 42. What is a Marker Interface?

An interface with no methods, used to signal metadata to the JVM or frameworks.

```java
// Built-in examples:
java.io.Serializable
java.lang.Cloneable
java.util.RandomAccess
```

---

## 43. What is `Serializable`?

Marks a class so its instances can be converted to a byte stream (serialization).

```java
import java.io.Serializable;

class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    String name;
    transient String password;  // excluded from serialization
}
```

---

## 44. What are Access Modifiers?

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| (default) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

---

## 45. What is an Association?

A relationship where objects of one class are linked to objects of another, without ownership. Can be one-to-one, one-to-many, etc.

```java
class Teacher { String name; }

class Student {
    String name;
    Teacher teacher;  // associated, but neither owns the other
}
```

---

## 46. What is a design pattern? Name a few OOP patterns.

A design pattern is a reusable solution to a common software design problem.

| Category | Pattern | Purpose |
|---|---|---|
| Creational | Singleton | One instance |
| Creational | Factory Method | Delegates object creation |
| Structural | Decorator | Add behavior dynamically |
| Behavioral | Observer | Event notification |
| Behavioral | Strategy | Swap algorithms at runtime |

---

## 47. What is the Factory Pattern?

Delegates object creation to a factory method instead of `new`.

```java
interface Shape { void draw(); }
class Circle  implements Shape { public void draw() { System.out.println("Circle");  } }
class Square  implements Shape { public void draw() { System.out.println("Square");  } }

class ShapeFactory {
    static Shape create(String type) {
        return switch (type) {
            case "circle" -> new Circle();
            case "square" -> new Square();
            default -> throw new IllegalArgumentException("Unknown: " + type);
        };
    }
}

Shape s = ShapeFactory.create("circle");
s.draw();  // Circle
```

---

## 48. What is the difference between early binding and late binding?

- **Early (static) binding**: resolved at compile time — `static`, `private`, `final` methods.
- **Late (dynamic) binding**: resolved at runtime — overridden instance methods.

```java
class Base   { void greet() { System.out.println("Base");  } }
class Derived extends Base { void greet() { System.out.println("Derived"); } }

Base obj = new Derived();
obj.greet();  // Derived — late binding (runtime decision)
```

---

## 49. What is the SOLID principle?

| Letter | Principle | Summary |
|---|---|---|
| S | Single Responsibility | One class, one reason to change |
| O | Open/Closed | Open for extension, closed for modification |
| L | Liskov Substitution | Subtypes must be substitutable for their base types |
| I | Interface Segregation | Prefer small, specific interfaces over large ones |
| D | Dependency Inversion | Depend on abstractions, not concrete classes |

```java
// D — Dependency Inversion example
interface MessageSender { void send(String msg); }

class EmailSender implements MessageSender {
    public void send(String msg) { System.out.println("Email: " + msg); }
}

class Notification {
    private MessageSender sender;               // depends on abstraction
    Notification(MessageSender s) { sender = s; }
    void notify(String msg) { sender.send(msg); }
}
```

---

## 50. What is the difference between Procedural and Object-Oriented Programming?

| Aspect | Procedural | Object-Oriented |
|---|---|---|
| Focus | Functions / procedures | Objects and their interactions |
| Data | Shared, passed around | Encapsulated within objects |
| Reuse | Functions | Inheritance & Composition |
| Maintenance | Harder as scale grows | Easier via modularity |
| Examples | C, Pascal | Java, C++, Python |

```java
// Procedural style
static double calcArea(double w, double h) { return w * h; }

// OOP style
class Rectangle {
    double width, height;
    Rectangle(double w, double h) { width = w; height = h; }
    double area() { return width * height; }
}
```

---

*Happy interviewing! 🚀*
