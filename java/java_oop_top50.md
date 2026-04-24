# Top 50 Java Object & Class Core Questions and Answers

---

## 🔷 Classes & Objects

**Q1. What is a class in Java?**
A class is a blueprint or template that defines the structure (fields) and behavior (methods) of objects. It does not occupy memory on its own.

```java
public class Dog {
    String name;
    void bark() { System.out.println("Woof!"); }
}
```

---

**Q2. What is an object in Java?**
An object is an instance of a class. It occupies memory and has its own state (field values) and behavior (methods).

```java
Dog d = new Dog(); // d is an object of class Dog
d.name = "Rex";
d.bark();
```

---

**Q3. What is the difference between a class and an object?**
| Class | Object |
|-------|--------|
| Blueprint/template | Instance of a class |
| Defined once | Can have many instances |
| No memory allocated (except static) | Memory allocated on heap |

---

**Q4. How do you create an object in Java?**
Using the `new` keyword, which allocates memory and calls the constructor.

```java
Car car = new Car();
```

---

**Q5. What is a constructor?**
A constructor is a special method that is called when an object is created. It has the same name as the class and no return type.

```java
public class Car {
    String model;
    public Car(String model) {
        this.model = model;
    }
}
```

---

**Q6. What is a default constructor?**
A no-argument constructor automatically provided by Java if no constructor is defined. It initializes fields to default values (`0`, `null`, `false`).

```java
public class Box { }
// Java provides: public Box() { }
```

---

**Q7. Can a class have multiple constructors?**
Yes — this is called **constructor overloading**.

```java
public class Point {
    int x, y;
    public Point() { x = 0; y = 0; }
    public Point(int x, int y) { this.x = x; this.y = y; }
}
```

---

**Q8. What is `this` keyword?**
`this` refers to the current instance of the class. Used to distinguish instance variables from parameters and to call other constructors.

```java
public class Person {
    String name;
    public Person(String name) {
        this.name = name; // 'this.name' is the field
    }
}
```

---

**Q9. What is constructor chaining?**
Calling one constructor from another using `this()` (same class) or `super()` (parent class).

```java
public class Rectangle {
    int w, h;
    public Rectangle() { this(1, 1); }
    public Rectangle(int w, int h) { this.w = w; this.h = h; }
}
```

---

**Q10. What are instance variables vs local variables?**
- **Instance variables**: Declared inside the class but outside methods. Belong to the object. Have default values.
- **Local variables**: Declared inside a method. Must be initialized before use. No default values.

---

## 🔷 Inheritance

**Q11. What is inheritance?**
Inheritance allows a class (child/subclass) to acquire fields and methods of another class (parent/superclass) using the `extends` keyword.

```java
public class Animal { void eat() { System.out.println("Eating"); } }
public class Dog extends Animal { void bark() { System.out.println("Woof"); } }
```

---

**Q12. What is `super` keyword?**
`super` refers to the parent class. Used to call parent constructors or methods.

```java
public class Dog extends Animal {
    public Dog() { super(); } // calls Animal's constructor
}
```

---

**Q13. Does Java support multiple inheritance?**
Not through classes. Java only supports **single inheritance** for classes. Multiple inheritance is achieved via **interfaces**.

---

**Q14. What is method overriding?**
A subclass provides its own implementation of a method already defined in the parent class. The method signature must match exactly.

```java
class Animal { void sound() { System.out.println("..."); } }
class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow"); }
}
```

---

**Q15. What is the difference between overloading and overriding?**
| Overloading | Overriding |
|-------------|------------|
| Same class, different parameters | Parent-child, same signature |
| Compile-time polymorphism | Runtime polymorphism |
| Return type can differ | Return type must be same (or covariant) |

---

**Q16. Can we override a `static` method?**
No. Static methods belong to the class, not instances. You can **hide** a static method in a subclass, but it is not true overriding.

---

**Q17. What is the `final` keyword with classes and methods?**
- `final` class: Cannot be subclassed (e.g., `String`).
- `final` method: Cannot be overridden.
- `final` variable: Cannot be reassigned after initialization.

---

**Q18. What is an abstract class?**
A class declared with `abstract` that cannot be instantiated. It may contain abstract methods (no body) that subclasses must implement.

```java
abstract class Shape {
    abstract double area();
}
class Circle extends Shape {
    double r;
    double area() { return Math.PI * r * r; }
}
```

---

**Q19. What is the difference between abstract class and interface?**
| Abstract Class | Interface |
|----------------|-----------|
| Can have constructors | No constructors |
| Can have state (fields) | Only constants (`public static final`) |
| Single inheritance | Multiple implementation |
| `extends` | `implements` |
| Can have concrete methods | Default/static methods (Java 8+) |

---

**Q20. Can an abstract class have a constructor?**
Yes. It is called when a subclass object is created via `super()`.

---

## 🔷 Interfaces

**Q21. What is an interface?**
An interface is a contract that defines what a class must do. All methods are implicitly `public abstract` (unless `default` or `static`).

```java
interface Flyable {
    void fly();
}
class Bird implements Flyable {
    public void fly() { System.out.println("Flapping wings"); }
}
```

---

**Q22. What are default methods in interfaces (Java 8+)?**
Methods in interfaces with a body, allowing backward compatibility without breaking existing implementations.

```java
interface Greet {
    default void hello() { System.out.println("Hello!"); }
}
```

---

**Q23. Can a class implement multiple interfaces?**
Yes — this is how Java achieves multiple inheritance of behavior.

```java
class Duck implements Flyable, Swimmable { ... }
```

---

**Q24. Can an interface extend another interface?**
Yes, using `extends`.

```java
interface A { void methodA(); }
interface B extends A { void methodB(); }
```

---

## 🔷 Encapsulation & Access Modifiers

**Q25. What is encapsulation?**
Wrapping data (fields) and methods into a single unit (class), and restricting direct access to fields using access modifiers. Achieved via private fields + public getters/setters.

```java
public class Account {
    private double balance;
    public double getBalance() { return balance; }
    public void deposit(double amount) { balance += amount; }
}
```

---

**Q26. What are the four access modifiers in Java?**
| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| (default) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

---

**Q27. What is the difference between `private` and `protected`?**
`private` is accessible only within the same class. `protected` is accessible within the same package and by subclasses.

---

## 🔷 Polymorphism

**Q28. What is polymorphism?**
The ability of an object to take many forms. In Java: method overloading (compile-time) and method overriding (runtime).

---

**Q29. What is runtime polymorphism?**
When a parent class reference holds a child class object, and the overridden method is resolved at runtime.

```java
Animal a = new Dog(); // Dog overrides sound()
a.sound();            // calls Dog's sound() at runtime
```

---

**Q30. What is `instanceof` keyword?**
Checks whether an object is an instance of a specific class or interface.

```java
if (animal instanceof Dog) {
    Dog d = (Dog) animal;
}
```

---

## 🔷 Static Members

**Q31. What is a static variable?**
A variable shared across all instances of a class. Belongs to the class, not any object.

```java
class Counter {
    static int count = 0;
    Counter() { count++; }
}
```

---

**Q32. What is a static method?**
A method that belongs to the class and can be called without creating an object. Cannot access instance variables or `this`.

```java
public static int add(int a, int b) { return a + b; }
```

---

**Q33. What is a static block?**
A block that runs once when the class is loaded, used to initialize static variables.

```java
static {
    System.out.println("Class loaded");
}
```

---

**Q34. Can we override static methods?**
No. Static methods can be hidden but not overridden. They are resolved at compile time, not runtime.

---

## 🔷 Object Class Methods

**Q35. What methods does every Java class inherit from `Object`?**
- `toString()` — string representation
- `equals(Object o)` — equality check
- `hashCode()` — hash value
- `getClass()` — runtime class
- `clone()` — object copy
- `finalize()` — called before GC
- `wait()`, `notify()`, `notifyAll()` — threading

---

**Q36. What is the contract between `equals()` and `hashCode()`?**
If two objects are equal via `equals()`, they **must** have the same `hashCode()`. If you override one, you must override both.

---

**Q37. What does `toString()` return by default?**
`ClassName@hexHashCode` (e.g., `Dog@1b6d3586`). Override it to return a meaningful string.

---

**Q38. What is shallow copy vs deep copy?**
- **Shallow copy**: Copies field values as-is. Object references point to the same objects.
- **Deep copy**: Creates new copies of all referenced objects too.

---

## 🔷 Inner Classes

**Q39. What is an inner class?**
A class defined inside another class. Has access to all members of the outer class.

```java
class Outer {
    class Inner {
        void show() { System.out.println("Inner class"); }
    }
}
```

---

**Q40. What is a static nested class?**
A static class inside another class. Does **not** have access to instance members of the outer class.

```java
class Outer {
    static class Nested { void show() { ... } }
}
```

---

**Q41. What is an anonymous class?**
A class defined and instantiated in one expression. Often used to override methods on the fly.

```java
Runnable r = new Runnable() {
    public void run() { System.out.println("Running"); }
};
```

---

## 🔷 Object Lifecycle & Memory

**Q42. Where are objects stored in Java?**
Objects are stored on the **heap**. Local variables and references are stored on the **stack**.

---

**Q43. What is garbage collection?**
Java automatically reclaims memory from objects that are no longer referenced. You cannot force GC but can suggest it with `System.gc()`.

---

**Q44. What is the difference between `==` and `equals()`?**
- `==` compares **references** (whether they point to the same object).
- `equals()` compares **content/value** (as defined by the class).

```java
String a = new String("hello");
String b = new String("hello");
a == b;       // false (different objects)
a.equals(b);  // true (same content)
```

---

## 🔷 Advanced OOP Concepts

**Q45. What is cohesion?**
Cohesion measures how closely related the responsibilities of a class are. **High cohesion** = a class does one thing well (preferred).

---

**Q46. What is coupling?**
Coupling measures how dependent classes are on each other. **Low coupling** = classes are independent (preferred).

---

**Q47. What is the difference between composition and inheritance?**
- **Inheritance**: "is-a" relationship (Dog is an Animal).
- **Composition**: "has-a" relationship (Car has an Engine). Preferred for flexibility.

```java
class Engine { void start() { ... } }
class Car {
    Engine engine = new Engine(); // composition
}
```

---

**Q48. What is an immutable class?**
A class whose objects cannot be modified after creation. Make fields `private final`, no setters, and return new objects from methods. Example: `String`.

```java
public final class ImmutablePoint {
    private final int x, y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
}
```

---

**Q49. What is the Singleton pattern?**
A design pattern ensuring only one instance of a class exists.

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

---

**Q50. What is the difference between early binding and late binding?**
- **Early binding (static/compile-time)**: Method call resolved at compile time. Used in overloading and `static` methods.
- **Late binding (dynamic/runtime)**: Method call resolved at runtime. Used in method overriding via polymorphism.

---

*End of Top 50 Java Object & Class Core Q&A*
