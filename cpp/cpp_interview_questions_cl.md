# Top 100 C++ Interview Questions
## Classes, Objects, Interfaces, Abstract Interfaces & Abstract Classes

---

## Table of Contents
1. [Classes & Objects (Q1–Q30)](#classes--objects)
2. [Constructors & Destructors (Q31–Q45)](#constructors--destructors)
3. [Abstract Classes (Q46–Q60)](#abstract-classes)
4. [Interfaces in C++ (Q61–Q75)](#interfaces-in-c)
5. [Abstract Interfaces (Q76–Q85)](#abstract-interfaces)
6. [Advanced & Mixed Topics (Q86–Q100)](#advanced--mixed-topics)

---

## Classes & Objects

---

### Q1. What is a class in C++?

**Answer:**  
A class is a user-defined blueprint or template that groups data (attributes) and functions (methods) that operate on that data. It enables encapsulation — one of the core pillars of OOP.

```cpp
class Car {
public:
    string brand;
    int year;

    void display() {
        cout << brand << " (" << year << ")" << endl;
    }
};

int main() {
    Car c;
    c.brand = "Toyota";
    c.year = 2022;
    c.display(); // Output: Toyota (2022)
}
```

---

### Q2. What is an object in C++?

**Answer:**  
An object is an instance of a class. It occupies memory and holds actual values for the class attributes.

```cpp
Car myCar;       // 'myCar' is an object of class Car
myCar.brand = "Honda";
myCar.year = 2020;
```

---

### Q3. What are access specifiers in C++?

**Answer:**  
Access specifiers control the visibility of class members:
- `public` – accessible from anywhere
- `private` – accessible only within the class
- `protected` – accessible within the class and derived classes

```cpp
class Person {
public:
    string name;       // accessible everywhere
private:
    int age;           // accessible only inside Person
protected:
    string address;    // accessible in Person and subclasses
};
```

---

### Q4. What is the default access specifier in a class vs. a struct?

**Answer:**  
- In a `class`, the default is `private`.
- In a `struct`, the default is `public`.

```cpp
class MyClass {
    int x; // private by default
};

struct MyStruct {
    int x; // public by default
};
```

---

### Q5. What is encapsulation?

**Answer:**  
Encapsulation is the bundling of data and the functions that manipulate it into a single unit (class), while restricting direct access to internal data using access specifiers.

```cpp
class BankAccount {
private:
    double balance;
public:
    void deposit(double amount) { balance += amount; }
    double getBalance() { return balance; }
};
```

---

### Q6. What is the difference between a class and a struct in C++?

**Answer:**  
The only technical difference is the default access level:
- `class` members default to `private`
- `struct` members default to `public`

Both support constructors, methods, and inheritance.

```cpp
struct Point { int x, y; };     // x, y are public
class Point2 { int x, y; };     // x, y are private
```

---

### Q7. How do you define a member function outside the class?

**Answer:**  
Using the scope resolution operator `::`.

```cpp
class Rectangle {
public:
    int width, height;
    int area(); // declaration
};

int Rectangle::area() { // definition outside class
    return width * height;
}
```

---

### Q8. What is a static member variable?

**Answer:**  
A static member variable is shared among all objects of a class. It is not tied to any specific instance.

```cpp
class Counter {
public:
    static int count;
    Counter() { count++; }
};

int Counter::count = 0;

int main() {
    Counter a, b, c;
    cout << Counter::count; // Output: 3
}
```

---

### Q9. What is a static member function?

**Answer:**  
A static member function can be called without an object. It can only access static members.

```cpp
class MathUtil {
public:
    static int square(int x) { return x * x; }
};

int main() {
    cout << MathUtil::square(5); // Output: 25
}
```

---

### Q10. What is the `this` pointer?

**Answer:**  
`this` is an implicit pointer available in all non-static member functions. It points to the current object.

```cpp
class Box {
    int width;
public:
    void setWidth(int width) {
        this->width = width; // disambiguates member vs. parameter
    }
};
```

---

### Q11. What is a friend function?

**Answer:**  
A friend function is a non-member function that has access to the private and protected members of a class.

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}
    friend double area(Circle c);
};

double area(Circle c) {
    return 3.14159 * c.radius * c.radius;
}
```

---

### Q12. What is a friend class?

**Answer:**  
A friend class can access private and protected members of another class.

```cpp
class Engine;

class Car {
    friend class Engine;
private:
    int horsepower = 300;
};

class Engine {
public:
    void show(Car& c) {
        cout << c.horsepower; // allowed due to friendship
    }
};
```

---

### Q13. What is operator overloading?

**Answer:**  
Operator overloading allows you to define custom behavior for operators (+, -, ==, etc.) for user-defined types.

```cpp
class Vector {
public:
    int x, y;
    Vector(int x, int y) : x(x), y(y) {}
    Vector operator+(const Vector& v) {
        return Vector(x + v.x, y + v.y);
    }
};
```

---

### Q14. Can a class have multiple constructors?

**Answer:**  
Yes, through constructor overloading (multiple constructors with different parameters).

```cpp
class Student {
public:
    string name;
    int age;
    Student() : name("Unknown"), age(0) {}
    Student(string n, int a) : name(n), age(a) {}
};
```

---

### Q15. What is a copy constructor?

**Answer:**  
A copy constructor creates a new object as a copy of an existing one. It takes a const reference to the same class.

```cpp
class Book {
public:
    string title;
    Book(const Book& b) : title(b.title) {}
};
```

---

### Q16. What is shallow copy vs. deep copy?

**Answer:**  
- **Shallow copy**: copies the pointer (both objects share the same memory).
- **Deep copy**: allocates new memory and copies the actual data.

```cpp
class Deep {
public:
    int* data;
    Deep(int val) { data = new int(val); }
    Deep(const Deep& d) {        // deep copy
        data = new int(*d.data);
    }
    ~Deep() { delete data; }
};
```

---

### Q17. What is an inline function?

**Answer:**  
An inline function suggests to the compiler to substitute the function body at the call site, reducing overhead for small functions.

```cpp
class Math {
public:
    inline int cube(int x) { return x * x * x; }
};
```

---

### Q18. What is object slicing?

**Answer:**  
Object slicing happens when a derived class object is assigned to a base class object — the derived-specific data is "sliced" off.

```cpp
class Base { public: int x = 1; };
class Derived : public Base { public: int y = 2; };

Derived d;
Base b = d; // y is lost — sliced off
```

---

### Q19. What is the size of an empty class?

**Answer:**  
The size of an empty class is 1 byte. The compiler assigns this to ensure two distinct objects have different addresses.

```cpp
class Empty {};
cout << sizeof(Empty); // Output: 1
```

---

### Q20. Can objects be passed by value, reference, and pointer?

**Answer:**  
Yes. Passing by reference or pointer avoids copying and is preferred for large objects.

```cpp
void byValue(Car c) { ... }       // copy made
void byRef(Car& c) { ... }        // no copy
void byPtr(Car* c) { ... }        // pointer access
```

---

### Q21. What is method overloading?

**Answer:**  
Defining multiple functions with the same name but different parameter lists (compile-time polymorphism).

```cpp
class Printer {
public:
    void print(int x)    { cout << "Int: " << x; }
    void print(double x) { cout << "Double: " << x; }
    void print(string x) { cout << "String: " << x; }
};
```

---

### Q22. What is a const member function?

**Answer:**  
A const member function cannot modify any member variables of the class.

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double getRadius() const { return radius; } // read-only
};
```

---

### Q23. What is a mutable keyword?

**Answer:**  
`mutable` allows a member variable to be modified even inside a const member function.

```cpp
class Logger {
    mutable int callCount = 0;
public:
    void log() const { callCount++; } // allowed due to mutable
};
```

---

### Q24. What is an explicit constructor?

**Answer:**  
`explicit` prevents implicit conversions from occurring via a constructor.

```cpp
class MyInt {
public:
    explicit MyInt(int x) {}
};

MyInt a = 5;   // ERROR: implicit conversion blocked
MyInt b(5);    // OK
```

---

### Q25. What is a delegating constructor?

**Answer:**  
A constructor that calls another constructor of the same class.

```cpp
class Config {
    int width, height;
public:
    Config() : Config(800, 600) {}             // delegates
    Config(int w, int h) : width(w), height(h) {}
};
```

---

### Q26. What is a move constructor?

**Answer:**  
A move constructor transfers ownership of resources from a temporary (rvalue) object, avoiding unnecessary copying.

```cpp
class Buffer {
public:
    int* data;
    Buffer(Buffer&& other) noexcept {
        data = other.data;
        other.data = nullptr; // transfer ownership
    }
};
```

---

### Q27. What is the Rule of Three in C++?

**Answer:**  
If a class defines any of the following, it likely needs all three:
1. Destructor
2. Copy constructor
3. Copy assignment operator

```cpp
class Resource {
    int* ptr;
public:
    Resource(int v) : ptr(new int(v)) {}
    ~Resource() { delete ptr; }
    Resource(const Resource& r) : ptr(new int(*r.ptr)) {}
    Resource& operator=(const Resource& r) {
        if (this != &r) { delete ptr; ptr = new int(*r.ptr); }
        return *this;
    }
};
```

---

### Q28. What is the Rule of Five?

**Answer:**  
Extends Rule of Three to include:
4. Move constructor
5. Move assignment operator

```cpp
Resource(Resource&& r) noexcept : ptr(r.ptr) { r.ptr = nullptr; }
Resource& operator=(Resource&& r) noexcept {
    if (this != &r) { delete ptr; ptr = r.ptr; r.ptr = nullptr; }
    return *this;
}
```

---

### Q29. What is a nested class?

**Answer:**  
A class defined inside another class. The inner class has access to the outer class's static members.

```cpp
class Outer {
public:
    class Inner {
    public:
        void greet() { cout << "Hello from Inner!"; }
    };
};

Outer::Inner obj;
obj.greet();
```

---

### Q30. What is an anonymous class / unnamed class?

**Answer:**  
A class without a name, usually used for one-time-use types.

```cpp
struct {
    int x = 10;
    int y = 20;
} point;

cout << point.x; // Output: 10
```

---

## Constructors & Destructors

---

### Q31. What is a constructor?

**Answer:**  
A constructor is a special member function with the same name as the class. It initializes objects automatically when they are created.

```cpp
class Dog {
public:
    string name;
    Dog(string n) : name(n) { cout << name << " created!"; }
};
```

---

### Q32. What is a destructor?

**Answer:**  
A destructor is a special function called when an object goes out of scope or is deleted. It performs cleanup (releasing memory, closing files, etc.).

```cpp
class FileHandler {
public:
    ~FileHandler() { cout << "File closed."; }
};
```

---

### Q33. Can a constructor be private?

**Answer:**  
Yes. This is used in singleton patterns to prevent direct instantiation.

```cpp
class Singleton {
    static Singleton* instance;
    Singleton() {}
public:
    static Singleton* getInstance() {
        if (!instance) instance = new Singleton();
        return instance;
    }
};
```

---

### Q34. Can a constructor throw exceptions?

**Answer:**  
Yes. If a constructor throws, the object is not fully constructed and the destructor will NOT be called.

```cpp
class Safe {
public:
    Safe(int val) {
        if (val < 0) throw invalid_argument("Negative value!");
    }
};
```

---

### Q35. Can a destructor be virtual?

**Answer:**  
Yes — and it MUST be virtual in a base class if you plan to delete derived objects via base pointers. Otherwise, only the base destructor is called.

```cpp
class Base {
public:
    virtual ~Base() { cout << "Base destroyed"; }
};
class Derived : public Base {
public:
    ~Derived() { cout << "Derived destroyed"; }
};

Base* b = new Derived();
delete b; // calls both destructors correctly
```

---

### Q36. What is a default constructor?

**Answer:**  
A constructor with no parameters (or all default parameters). It is auto-generated by the compiler if no constructor is defined.

```cpp
class Point {
public:
    int x = 0, y = 0;
    Point() {} // default constructor
};
```

---

### Q37. What is an initializer list in a constructor?

**Answer:**  
An initializer list directly initializes member variables before the constructor body runs. It's required for const, reference members, and base class initialization.

```cpp
class Rect {
    const int width;
    int height;
public:
    Rect(int w, int h) : width(w), height(h) {} // initializer list
};
```

---

### Q38. What happens when a derived class object is created?

**Answer:**  
Constructors are called in order: **base class first**, then derived class. Destructors are called in **reverse order**.

```cpp
class A { public: A() { cout << "A "; } ~A() { cout << "~A "; } };
class B : public A { public: B() { cout << "B "; } ~B() { cout << "~B "; } };

B obj; // Output: A B
       // On destruction: ~B ~A
```

---

### Q39. What is a parameterized constructor?

**Answer:**  
A constructor that accepts arguments to initialize an object with specific values.

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}
};
```

---

### Q40. Can you call a constructor explicitly?

**Answer:**  
Yes, using placement new or direct construction syntax.

```cpp
class Foo { public: Foo() { cout << "Foo created"; } };

Foo f = Foo(); // explicit call
```

---

### Q41. What is a copy assignment operator?

**Answer:**  
It defines how one object is assigned the value of another existing object using `=`.

```cpp
class Data {
    int val;
public:
    Data& operator=(const Data& d) {
        if (this != &d) val = d.val;
        return *this;
    }
};
```

---

### Q42. When is a copy constructor called vs. copy assignment?

**Answer:**  
- **Copy constructor**: called during initialization (`MyClass b = a;`)
- **Copy assignment**: called when assigning to an already-existing object (`b = a;`)

```cpp
MyClass a;
MyClass b = a; // copy constructor
MyClass c;
c = a;         // copy assignment
```

---

### Q43. What does `= delete` do to a constructor?

**Answer:**  
It explicitly disables the constructor, preventing the compiler from generating it or users from using it.

```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;  // no copy allowed
};
```

---

### Q44. What does `= default` mean?

**Answer:**  
It tells the compiler to generate the default implementation of a constructor or destructor.

```cpp
class Simple {
public:
    Simple() = default;
    ~Simple() = default;
};
```

---

### Q45. What is a converting constructor?

**Answer:**  
A constructor with a single argument that enables implicit type conversion (unless marked `explicit`).

```cpp
class Celsius {
    double temp;
public:
    Celsius(double t) : temp(t) {} // converting constructor
};

Celsius c = 36.6; // implicit conversion from double
```

---

## Abstract Classes

---

### Q46. What is an abstract class in C++?

**Answer:**  
An abstract class has at least one pure virtual function (`= 0`). It cannot be instantiated directly and is meant to serve as a base class.

```cpp
class Shape {
public:
    virtual double area() = 0; // pure virtual — makes Shape abstract
    virtual ~Shape() {}
};
```

---

### Q47. What is a pure virtual function?

**Answer:**  
A virtual function declared with `= 0`. It has no implementation in the base class and must be overridden in any concrete derived class.

```cpp
class Animal {
public:
    virtual void speak() = 0; // pure virtual
};

class Dog : public Animal {
public:
    void speak() override { cout << "Woof!"; }
};
```

---

### Q48. Can an abstract class have a constructor?

**Answer:**  
Yes. Abstract classes can have constructors, which are called when a derived class object is created.

```cpp
class AbstractBase {
    int id;
public:
    AbstractBase(int i) : id(i) {}
    virtual void show() = 0;
};

class Concrete : public AbstractBase {
public:
    Concrete(int i) : AbstractBase(i) {}
    void show() override { cout << "Concrete"; }
};
```

---

### Q49. Can an abstract class have non-pure virtual functions?

**Answer:**  
Yes. An abstract class can have a mix of pure virtual, virtual, and regular functions.

```cpp
class Base {
public:
    virtual void pureFunc() = 0;    // must override
    virtual void optFunc() { cout << "default"; } // can override
    void regularFunc() { cout << "always same"; } // cannot override
};
```

---

### Q50. What happens if a derived class does not override all pure virtual functions?

**Answer:**  
The derived class also becomes abstract and cannot be instantiated.

```cpp
class Abstract { public: virtual void f() = 0; virtual void g() = 0; };

class PartialConcrete : public Abstract {
public:
    void f() override {} // only partially implemented
    // g() still pure virtual — PartialConcrete is still abstract
};
```

---

### Q51. Can an abstract class have data members?

**Answer:**  
Yes. Abstract classes can hold data members to be used by derived classes.

```cpp
class Vehicle {
protected:
    int speed;
public:
    Vehicle(int s) : speed(s) {}
    virtual void describe() = 0;
};
```

---

### Q52. Can you have a pointer or reference to an abstract class?

**Answer:**  
Yes. This is the primary way to use abstract classes (polymorphism via base pointer/reference).

```cpp
Shape* s = new Circle(5.0); // Shape is abstract, Circle is concrete
cout << s->area();
delete s;
```

---

### Q53. What is the difference between an abstract class and a concrete class?

**Answer:**  
- **Abstract class**: has at least one pure virtual function; cannot be instantiated.
- **Concrete class**: implements all pure virtual functions; can be instantiated.

```cpp
class Abstract { public: virtual void f() = 0; };    // cannot instantiate
class Concrete : public Abstract { public: void f() {} }; // can instantiate
```

---

### Q54. Can a pure virtual function have a body?

**Answer:**  
Yes! A pure virtual function can have an optional implementation (usually for fallback), but the class is still abstract.

```cpp
class Base {
public:
    virtual void greet() = 0;
};

void Base::greet() { cout << "Hello from Base (fallback)"; }

class Child : public Base {
public:
    void greet() override { Base::greet(); } // calling base body
};
```

---

### Q55. What is a virtual destructor and why is it important in abstract classes?

**Answer:**  
A virtual destructor ensures the correct destructor is called when a derived object is deleted via a base pointer.

```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() {}     // virtual destructor
    virtual void process() = 0;
};
```

---

### Q56. What is the vtable (virtual table)?

**Answer:**  
The vtable is a compile-time mechanism: a table of function pointers for virtual functions. Each class with virtual functions gets a vtable, and each object has a hidden `vptr` (pointer to the vtable).

```cpp
// Conceptually:
// Shape's vtable: { &Shape::area }
// Circle's vtable: { &Circle::area }
Shape* s = new Circle();
s->area(); // resolved at runtime via vtable
```

---

### Q57. Can abstract classes be used with templates?

**Answer:**  
Yes. Abstract classes work with template mechanisms to enforce interface contracts.

```cpp
template <typename T>
class Container {
public:
    virtual void add(T item) = 0;
    virtual T get(int index) = 0;
};
```

---

### Q58. What is the difference between an abstract class and a virtual class?

**Answer:**  
All abstract classes contain virtual functions, but a "virtual class" often refers to virtual base classes used in multiple inheritance to avoid diamond problem duplication. They are different concepts.

```cpp
class A { public: virtual void f() = 0; };   // abstract class
class B : virtual public A {};               // virtual (base) class for diamond resolution
```

---

### Q59. What is the diamond problem?

**Answer:**  
When a class inherits from two classes that both inherit from a common base, there can be ambiguity. `virtual` inheritance solves this.

```cpp
class A { public: int x; };
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {}; // D has only one copy of A::x
```

---

### Q60. Can you have an array of abstract class objects?

**Answer:**  
No, you cannot create an array of abstract objects directly. But you can have an array of pointers to abstract types.

```cpp
Shape* shapes[3];
shapes[0] = new Circle(1.0);
shapes[1] = new Rectangle(2.0, 3.0);
shapes[2] = new Triangle(1.5, 2.5);
```

---

## Interfaces in C++

---

### Q61. Does C++ have interfaces like Java?

**Answer:**  
C++ has no `interface` keyword. Interfaces are simulated using abstract classes where all functions are pure virtual and there are no data members.

```cpp
class IDrawable {
public:
    virtual void draw() = 0;
    virtual ~IDrawable() {}
};
```

---

### Q62. How do you implement an interface in C++?

**Answer:**  
A class "implements" a C++ interface by inheriting from it and overriding all pure virtual functions.

```cpp
class Circle : public IDrawable {
public:
    void draw() override { cout << "Drawing circle"; }
};
```

---

### Q63. Can a class implement multiple interfaces in C++?

**Answer:**  
Yes. C++ supports multiple inheritance, allowing a class to implement multiple abstract interfaces.

```cpp
class ISerializable { public: virtual string serialize() = 0; virtual ~ISerializable() {} };
class ILoggable    { public: virtual void log() = 0; virtual ~ILoggable() {} };

class DataObject : public ISerializable, public ILoggable {
public:
    string serialize() override { return "data"; }
    void log() override { cout << "Logging..."; }
};
```

---

### Q64. What are the naming conventions for interfaces in C++?

**Answer:**  
Common conventions include prefixing with `I` (e.g., `IShape`, `IDrawable`) or suffixing with `able` or `Interface`. It's a convention, not a language rule.

```cpp
class IComparable {
public:
    virtual int compareTo(const IComparable& other) = 0;
    virtual ~IComparable() {}
};
```

---

### Q65. What is a pure abstract class (interface)?

**Answer:**  
A class with ONLY pure virtual functions and no data members — the closest C++ equivalent of a Java/C# interface.

```cpp
class IAnimal {
public:
    virtual void speak() = 0;
    virtual void move() = 0;
    virtual ~IAnimal() = default;
};
```

---

### Q66. Can an interface have default implementations in C++?

**Answer:**  
Not directly as in C#/Java's default interface methods, but a pure virtual function can optionally have a body that derived classes can call explicitly.

```cpp
class ILogger {
public:
    virtual void log(string msg) = 0;
};

void ILogger::log(string msg) {
    cout << "[DEFAULT LOG]: " << msg;
}

class ConsoleLogger : public ILogger {
public:
    void log(string msg) override {
        ILogger::log(msg); // call default implementation
    }
};
```

---

### Q67. Can an interface inherit from another interface in C++?

**Answer:**  
Yes. An abstract class (interface) can inherit from another abstract class, extending the interface contract.

```cpp
class IShape {
public:
    virtual double area() = 0;
    virtual ~IShape() {}
};

class IResizable : public IShape {
public:
    virtual void resize(double factor) = 0;
};
```

---

### Q68. What is runtime polymorphism in the context of interfaces?

**Answer:**  
When you call a method through an interface pointer/reference, the correct derived class method is dispatched at runtime via the vtable.

```cpp
IAnimal* a = new Dog();
a->speak(); // calls Dog::speak() at runtime — not resolved at compile time
delete a;
```

---

### Q69. Can a non-abstract class be used as an interface?

**Answer:**  
Technically yes, but it's bad design. Interfaces should contain only pure virtual functions to enforce contracts without forcing implementations.

```cpp
// Discouraged pattern:
class BadInterface {
public:
    virtual void doSomething() { /* default behavior */ }
};
```

---

### Q70. What is the Liskov Substitution Principle (LSP) in C++?

**Answer:**  
Objects of a derived class should be substitutable for objects of the base class without altering the correctness of the program. Interfaces help enforce LSP.

```cpp
void makeSound(IAnimal* animal) {
    animal->speak(); // works for Dog, Cat, Bird — all substitutable
}
```

---

### Q71. What is the Interface Segregation Principle (ISP)?

**Answer:**  
Clients should not be forced to depend on methods they don't use. Prefer smaller, specific interfaces over large fat ones.

```cpp
// Fat interface (bad):
class IWorker { virtual void work() = 0; virtual void eat() = 0; };

// Segregated (good):
class IWorkable { public: virtual void work() = 0; };
class IFeedable { public: virtual void eat() = 0; };
```

---

### Q72. How do you check if an object implements an interface at runtime?

**Answer:**  
Use `dynamic_cast`. If the cast returns a non-null pointer, the object implements that interface.

```cpp
IAnimal* a = new Dog();
ILoggable* log = dynamic_cast<ILoggable*>(a);
if (log) log->log(); // Dog implements ILoggable
```

---

### Q73. What is a marker interface in C++?

**Answer:**  
A marker interface has no methods — it's used to "tag" a class for identification or capability.

```cpp
class ISerializable {}; // marker — no methods

class Config : public ISerializable {
    // signals Config can be serialized
};
```

---

### Q74. What is a callback interface?

**Answer:**  
An interface used to implement callback mechanisms (similar to event handlers or listener patterns).

```cpp
class IClickListener {
public:
    virtual void onClick() = 0;
    virtual ~IClickListener() {}
};

class Button {
    IClickListener* listener;
public:
    void setListener(IClickListener* l) { listener = l; }
    void click() { if (listener) listener->onClick(); }
};
```

---

### Q75. How do interfaces enable the Dependency Inversion Principle?

**Answer:**  
High-level modules depend on interfaces (abstractions), not concrete implementations. This decouples components.

```cpp
class IDatabase {
public:
    virtual void save(string data) = 0;
};

class AppService {
    IDatabase* db; // depends on abstraction, not concrete DB
public:
    AppService(IDatabase* d) : db(d) {}
    void store(string d) { db->save(d); }
};
```

---

## Abstract Interfaces

---

### Q76. What is an abstract interface in C++?

**Answer:**  
An abstract interface is a class with:
- Only pure virtual functions
- No data members (or only constants)
- A virtual destructor

It is the strictest form of abstraction in C++.

```cpp
class IStream {
public:
    virtual void open(string path) = 0;
    virtual void close() = 0;
    virtual void write(string data) = 0;
    virtual string read() = 0;
    virtual ~IStream() = default;
};
```

---

### Q77. How is an abstract interface different from an abstract class?

**Answer:**  

| Feature             | Abstract Class         | Abstract Interface     |
|---------------------|------------------------|------------------------|
| Data members        | Allowed                | Not allowed (ideally)  |
| Non-pure methods    | Allowed                | Not allowed            |
| Purpose             | Partial abstraction    | Full contract          |
| Constructors        | Allowed                | Usually not needed     |

```cpp
class AbstractClass { // partial
    int id;
    virtual void save() = 0;
    void log() { cout << "logging"; }
};

class Interface { // pure contract
    virtual void save() = 0;
    virtual ~Interface() {}
};
```

---

### Q78. Can an abstract interface have a virtual destructor?

**Answer:**  
Yes — and it SHOULD, to ensure proper cleanup of derived objects through interface pointers.

```cpp
class IProcessor {
public:
    virtual void process() = 0;
    virtual ~IProcessor() = default; // important!
};
```

---

### Q79. When should you use an abstract interface vs. an abstract class?

**Answer:**  
- **Abstract interface**: when you want to define a pure contract with no shared implementation.
- **Abstract class**: when you want to share common behavior or state among subclasses.

```cpp
// Use interface when:
class IRenderable { virtual void render() = 0; };

// Use abstract class when:
class Widget {
protected:
    string color;
public:
    virtual void draw() = 0;
    void setColor(string c) { color = c; } // shared
};
```

---

### Q80. How do abstract interfaces support the Open/Closed Principle?

**Answer:**  
Code is open for extension (new classes implement the interface) but closed for modification (existing code doesn't change when new types are added).

```cpp
class IPayment {
public:
    virtual void pay(double amount) = 0;
    virtual ~IPayment() {}
};

class CreditCard : public IPayment { void pay(double a) override { /*...*/ } };
class Bitcoin    : public IPayment { void pay(double a) override { /*...*/ } };
// Adding PayPal doesn't require changing existing code
```

---

### Q81. Can abstract interfaces be used with smart pointers?

**Answer:**  
Yes. This is the recommended approach for automatic memory management with polymorphic types.

```cpp
shared_ptr<IAnimal> animal = make_shared<Dog>();
animal->speak();
// automatically deleted when no more references
```

---

### Q82. How do you enforce a strict abstract interface in C++?

**Answer:**  
Declare all functions pure virtual, delete copy/move constructors, and provide a virtual destructor.

```cpp
class IStrict {
public:
    IStrict() = default;
    IStrict(const IStrict&) = delete;
    IStrict& operator=(const IStrict&) = delete;
    virtual ~IStrict() = default;
    virtual void execute() = 0;
};
```

---

### Q83. How do abstract interfaces help with unit testing (mocking)?

**Answer:**  
They allow creating mock implementations for testing without depending on real services.

```cpp
class IEmailService {
public:
    virtual void send(string to, string msg) = 0;
    virtual ~IEmailService() {}
};

class MockEmailService : public IEmailService {
public:
    void send(string to, string msg) override {
        cout << "[MOCK] Email to: " << to; // no real email sent
    }
};
```

---

### Q84. How are abstract interfaces used in plugin architectures?

**Answer:**  
Plugins implement a known interface, and the host loads them dynamically — without knowing concrete types at compile time.

```cpp
class IPlugin {
public:
    virtual string getName() = 0;
    virtual void run() = 0;
    virtual ~IPlugin() {}
};

// Each plugin DLL exports a class implementing IPlugin
```

---

### Q85. Can two classes implement the same abstract interface independently?

**Answer:**  
Yes. This is the entire point — multiple unrelated classes can satisfy the same contract.

```cpp
class IFlyable { public: virtual void fly() = 0; };

class Bird     : public IFlyable { void fly() override { cout << "Bird flies"; } };
class Airplane : public IFlyable { void fly() override { cout << "Plane flies"; } };
class Drone    : public IFlyable { void fly() override { cout << "Drone flies"; } };
```

---

## Advanced & Mixed Topics

---

### Q86. What is virtual inheritance and when is it used?

**Answer:**  
Virtual inheritance ensures only one copy of a base class exists in diamond inheritance scenarios.

```cpp
class Animal { public: string name; };
class Mammal  : virtual public Animal {};
class WingedAnimal : virtual public Animal {};
class Bat     : public Mammal, public WingedAnimal {};
// Bat has only ONE copy of Animal::name
```

---

### Q87. What is the difference between override and overload?

**Answer:**  
- **Overload**: same function name, different parameters (compile-time).
- **Override**: replacing a base class virtual function in a derived class (runtime).

```cpp
class Base {
public:
    virtual void show(int x) { cout << x; }
};
class Derived : public Base {
public:
    void show(int x) override { cout << "Derived: " << x; } // override
    void show(double x) { cout << "Double: " << x; }        // overload
};
```

---

### Q88. What is the `override` keyword?

**Answer:**  
`override` tells the compiler that a function is intended to override a base class virtual function. It produces a compile-time error if no matching base function exists.

```cpp
class Base { public: virtual void run() {} };
class Derived : public Base {
public:
    void run() override {}   // OK
    // void Run() override {}  // ERROR: no Base::Run() to override
};
```

---

### Q89. What is the `final` keyword?

**Answer:**  
- `final` on a class: prevents further inheritance.
- `final` on a function: prevents further overriding.

```cpp
class Sealed final {};       // cannot be subclassed
// class Sub : public Sealed {}; // ERROR

class Base {
public:
    virtual void go() final {} // cannot be overridden in subclasses
};
```

---

### Q90. What is RTTI (Run-Time Type Information)?

**Answer:**  
RTTI allows determining the type of an object at runtime. It's used through `typeid` and `dynamic_cast`.

```cpp
#include <typeinfo>
Base* b = new Derived();
cout << typeid(*b).name(); // prints Derived's type name

Derived* d = dynamic_cast<Derived*>(b); // safe downcast
```

---

### Q91. What is the difference between `dynamic_cast` and `static_cast`?

**Answer:**  
- `static_cast`: compile-time cast; no runtime safety check.
- `dynamic_cast`: runtime cast; returns `nullptr` if the cast is invalid (for pointers).

```cpp
Base* b = new Derived();
Derived* d1 = static_cast<Derived*>(b);  // assumes Derived, risky
Derived* d2 = dynamic_cast<Derived*>(b); // safe, checks at runtime
```

---

### Q92. What is a mixin class?

**Answer:**  
A mixin adds functionality to a class via multiple inheritance without being a full base class. It's a lightweight form of code reuse.

```cpp
class Serializable {
public:
    string toJSON() { return "{\"type\":\"object\"}"; }
};

class Loggable {
public:
    void log() { cout << "Logging..."; }
};

class DataModel : public Serializable, public Loggable {};
```

---

### Q93. What is CRTP (Curiously Recurring Template Pattern)?

**Answer:**  
CRTP is a template technique where a class derives from a template instantiation of itself, enabling static polymorphism without vtable overhead.

```cpp
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Concrete : public Base<Concrete> {
public:
    void implementation() { cout << "Concrete impl"; }
};
```

---

### Q94. What is a factory method pattern using abstract classes?

**Answer:**  
The factory method pattern uses an abstract base class to define an interface for creating objects, letting subclasses decide which type to instantiate.

```cpp
class IShape { public: virtual double area() = 0; virtual ~IShape() {} };
class Circle : public IShape { double r; public: Circle(double r):r(r){} double area() override { return 3.14*r*r; } };
class Square : public IShape { double s; public: Square(double s):s(s){} double area() override { return s*s; } };

IShape* createShape(string type) {
    if (type == "circle") return new Circle(5);
    if (type == "square") return new Square(4);
    return nullptr;
}
```

---

### Q95. What is the Template Method Pattern with abstract classes?

**Answer:**  
The base class defines the skeleton of an algorithm, deferring some steps to derived classes via pure virtual functions.

```cpp
class DataProcessor {
public:
    void process() {       // template method
        readData();
        processData();
        writeData();
    }
    virtual void readData() = 0;
    virtual void processData() = 0;
    virtual void writeData() = 0;
};
```

---

### Q96. What is the Strategy Pattern using interfaces?

**Answer:**  
Define a family of algorithms behind an interface and make them interchangeable at runtime.

```cpp
class ISortStrategy {
public:
    virtual void sort(vector<int>& data) = 0;
    virtual ~ISortStrategy() {}
};

class BubbleSort : public ISortStrategy {
public:
    void sort(vector<int>& data) override { /* bubble sort */ }
};

class Sorter {
    ISortStrategy* strategy;
public:
    Sorter(ISortStrategy* s) : strategy(s) {}
    void sort(vector<int>& d) { strategy->sort(d); }
};
```

---

### Q97. Can you use `std::function` instead of interfaces for callbacks?

**Answer:**  
Yes. `std::function` is more flexible for callbacks and avoids inheritance overhead, but lacks the explicit contract of an interface.

```cpp
#include <functional>
class Button {
    function<void()> onClick;
public:
    void setHandler(function<void()> f) { onClick = f; }
    void click() { if(onClick) onClick(); }
};

Button b;
b.setHandler([]{ cout << "Clicked!"; });
b.click();
```

---

### Q98. What is the difference between composition and inheritance?

**Answer:**  
- **Inheritance** ("is-a"): a derived class IS a type of base class.
- **Composition** ("has-a"): a class contains an instance of another class.

Prefer composition over inheritance for flexibility.

```cpp
// Inheritance:
class Dog : public Animal {};

// Composition:
class Dog {
    Animal animal; // has-a Animal
    Bark barkModule;
};
```

---

### Q99. How do you implement the Observer Pattern using interfaces?

**Answer:**  
Define an `IObserver` interface. Subjects hold a list of observers and notify them of changes.

```cpp
class IObserver {
public:
    virtual void update(string event) = 0;
    virtual ~IObserver() {}
};

class EventSystem {
    vector<IObserver*> observers;
public:
    void subscribe(IObserver* o) { observers.push_back(o); }
    void notify(string event) {
        for (auto o : observers) o->update(event);
    }
};

class Logger : public IObserver {
public:
    void update(string event) override { cout << "Log: " << event; }
};
```

---

### Q100. What are best practices for designing class hierarchies and interfaces in C++?

**Answer:**  
1. **Prefer interfaces** (pure abstract classes) for defining contracts.
2. **Use virtual destructors** in any class used polymorphically.
3. **Follow SOLID principles** — especially Single Responsibility and Interface Segregation.
4. **Prefer composition over deep inheritance hierarchies**.
5. **Use `override` and `final`** to catch errors at compile time.
6. **Use smart pointers** (`shared_ptr`, `unique_ptr`) for polymorphic objects.
7. **Keep abstract interfaces minimal** — small, focused contracts.
8. **Avoid data members in interfaces** — use abstract classes if shared state is needed.

```cpp
// Best practice example:
class IShape {
public:
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual ~IShape() = default; // virtual destructor
};

class Circle final : public IShape { // final: no further subclassing
    double radius;
public:
    explicit Circle(double r) : radius(r) {}
    double area() const override { return 3.14159 * radius * radius; }
    double perimeter() const override { return 2 * 3.14159 * radius; }
};
```

---

## Summary Table

| Concept              | Instantiable | Data Members | Pure Virtual | Use Case                        |
|----------------------|:------------:|:------------:|:------------:|----------------------------------|
| Concrete Class       | ✅           | ✅           | ❌           | Regular objects                  |
| Abstract Class       | ❌           | ✅           | ✅ (some)    | Shared state + partial contract  |
| Abstract Interface   | ❌           | ❌           | ✅ (all)     | Pure behavioral contract         |
| Mixin Class          | ✅           | sometimes    | optional     | Reusable behavior composition    |
| Singleton Class      | ✅ (1 only)  | ✅           | optional     | Single shared instance           |

---

*End of Top 100 C++ Interview Questions*
