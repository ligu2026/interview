# C++ Interview Questions: Structs, Classes, Interfaces, Abstract Classes, Value & Reference Types

> **80 curated questions with answers and code examples** — organized by topic for systematic review.

---

## Table of Contents

1. [Struct (Q1–Q12)](#struct)
2. [Class (Q13–Q27)](#class)
3. [Struct vs Class (Q28–Q35)](#struct-vs-class)
4. [Abstract Class (Q36–Q46)](#abstract-class)
5. [Interfaces / Pure Abstract Classes (Q47–Q55)](#interfaces--pure-abstract-classes)
6. [Value Types & Value Semantics (Q56–Q65)](#value-types--value-semantics)
7. [Reference Types & Reference Semantics (Q66–Q75)](#reference-types--reference-semantics)
8. [Mixed & Advanced Topics (Q76–Q80)](#mixed--advanced-topics)

---

## Struct

---

### Q1. What is a `struct` in C++?

**Answer:**  
A `struct` is a user-defined aggregate type that groups variables (members) of different types under one name. In C++ (unlike C), a `struct` can have constructors, destructors, member functions, access specifiers, and even inheritance.

```cpp
struct Point {
    int x;
    int y;

    // Member function
    double distanceFromOrigin() const {
        return std::sqrt(x * x + y * y);
    }
};

Point p{3, 4};
std::cout << p.distanceFromOrigin(); // 5.0
```

---

### Q2. What is the default access specifier in a `struct`?

**Answer:**  
`public`. All members of a `struct` are `public` by default unless explicitly specified otherwise.

```cpp
struct Foo {
    int x;       // public by default
private:
    int y;       // explicitly private
};

Foo f;
f.x = 10;   // OK
// f.y = 5; // Error: y is private
```

---

### Q3. Can a `struct` have constructors and destructors?

**Answer:**  
Yes. In C++, a `struct` is almost identical to a `class` and can have constructors, destructors, and any other special member functions.

```cpp
struct Timer {
    int seconds;

    Timer(int s) : seconds(s) {
        std::cout << "Timer started\n";
    }

    ~Timer() {
        std::cout << "Timer stopped\n";
    }
};

Timer t(30); // "Timer started"
             // "Timer stopped" on destruction
```

---

### Q4. Can a `struct` inherit from another `struct` or class?

**Answer:**  
Yes. A `struct` can inherit from other structs or classes. Default inheritance in a `struct` is `public`.

```cpp
struct Animal {
    std::string name;
    virtual void speak() const = 0;
};

struct Dog : Animal {      // public inheritance by default
    void speak() const override {
        std::cout << name << " says: Woof!\n";
    }
};

Dog d;
d.name = "Rex";
d.speak(); // Rex says: Woof!
```

---

### Q5. What is aggregate initialization in structs?

**Answer:**  
A struct with no user-declared constructors, no private/protected non-static data members, no base classes (C++17 relaxed this), and no virtual functions can be initialized with a brace-enclosed list.

```cpp
struct RGB {
    uint8_t r, g, b;
};

RGB red   = {255, 0, 0};
RGB green = {0, 255, 0};

std::cout << (int)red.r; // 255
```

---

### Q6. What is a POD struct and why does it matter?

**Answer:**  
POD (Plain Old Data) structs have trivial constructors/destructors, standard layout, and are compatible with C. They can be safely used with `memcpy`, `memset`, and in C interop.

```cpp
struct Vector3 {
    float x, y, z;
};

static_assert(std::is_pod<Vector3>::value, "Must be POD");

Vector3 a{1.f, 2.f, 3.f};
Vector3 b;
std::memcpy(&b, &a, sizeof(Vector3)); // Safe for POD
```

---

### Q7. Can a struct have static members?

**Answer:**  
Yes. Static members belong to the struct type, not individual instances, and must be defined outside the struct (unless `inline` in C++17).

```cpp
struct Counter {
    static int count;
    Counter() { ++count; }
    ~Counter() { --count; }
};

int Counter::count = 0; // Definition outside struct

Counter c1, c2;
std::cout << Counter::count; // 2
```

---

### Q8. What is a forward declaration of a struct?

**Answer:**  
A forward declaration tells the compiler a struct exists without providing its full definition. Useful for breaking circular dependencies. You can only use pointers/references to forward-declared types.

```cpp
struct Node; // forward declaration

struct List {
    Node* head; // OK: pointer to incomplete type
};

struct Node {
    int data;
    Node* next;
};
```

---

### Q9. What happens when you copy a struct?

**Answer:**  
By default, copying a struct performs a **member-wise copy** (shallow copy). If a member is a raw pointer, both copies will point to the same memory — a potential issue.

```cpp
struct Buffer {
    int* data;
    int size;
};

Buffer a;
a.size = 3;
a.data = new int[3]{1, 2, 3};

Buffer b = a;            // shallow copy
b.data[0] = 99;

std::cout << a.data[0]; // 99 — both point to same array!
delete[] a.data;         // deleting once is enough, but b.data is now dangling
```

---

### Q10. How do you make a struct non-copyable?

**Answer:**  
Delete the copy constructor and copy assignment operator.

```cpp
struct UniqueResource {
    UniqueResource() = default;
    UniqueResource(const UniqueResource&) = delete;
    UniqueResource& operator=(const UniqueResource&) = delete;
};

UniqueResource r1;
// UniqueResource r2 = r1; // Error: use of deleted function
```

---

### Q11. Can a struct be used as a template?

**Answer:**  
Yes. Structs support the full template mechanism just like classes.

```cpp
template<typename T>
struct Pair {
    T first;
    T second;

    T sum() const { return first + second; }
};

Pair<int>    pi{3, 7};
Pair<double> pd{1.5, 2.5};

std::cout << pi.sum(); // 10
std::cout << pd.sum(); // 4.0
```

---

### Q12. What is a bit-field in a struct?

**Answer:**  
A bit-field specifies the exact number of bits a member occupies. Useful for memory-efficient storage of flags or small integers.

```cpp
struct Flags {
    uint8_t isAlive    : 1;
    uint8_t isVisible  : 1;
    uint8_t level      : 6; // 0-63
};

Flags f{};
f.isAlive   = 1;
f.isVisible = 1;
f.level     = 42;

std::cout << sizeof(f); // typically 1 byte
```

---

## Class

---

### Q13. What is a class in C++?

**Answer:**  
A `class` is a user-defined type that encapsulates data (members) and behavior (methods). It forms the foundation of object-oriented programming in C++, supporting encapsulation, inheritance, and polymorphism.

```cpp
class BankAccount {
private:
    double balance;
    std::string owner;

public:
    BankAccount(const std::string& name, double initial)
        : owner(name), balance(initial) {}

    void deposit(double amount) { balance += amount; }
    void withdraw(double amount) {
        if (amount <= balance) balance -= amount;
    }
    double getBalance() const { return balance; }
};

BankAccount acc("Alice", 1000.0);
acc.deposit(500.0);
std::cout << acc.getBalance(); // 1500.0
```

---

### Q14. What is the default access specifier in a class?

**Answer:**  
`private`. All members are private unless explicitly specified.

```cpp
class Foo {
    int x;       // private by default
public:
    int y;       // public
};
```

---

### Q15. What are the special member functions of a class?

**Answer:**  
There are six special member functions the compiler can generate:

| Function | Signature |
|---|---|
| Default constructor | `T()` |
| Copy constructor | `T(const T&)` |
| Copy assignment | `T& operator=(const T&)` |
| Move constructor | `T(T&&)` |
| Move assignment | `T& operator=(T&&)` |
| Destructor | `~T()` |

```cpp
class Str {
    char* data;
public:
    Str(const char* s) : data(new char[strlen(s)+1]) {
        strcpy(data, s);
    }
    ~Str() { delete[] data; }

    // Copy constructor (deep copy)
    Str(const Str& other) : data(new char[strlen(other.data)+1]) {
        strcpy(data, other.data);
    }

    // Move constructor (steal resource)
    Str(Str&& other) noexcept : data(other.data) {
        other.data = nullptr;
    }
};
```

---

### Q16. What is encapsulation and how does a class enforce it?

**Answer:**  
Encapsulation hides internal implementation details and exposes only a controlled public interface. A class enforces it through access specifiers (`private`, `protected`, `public`).

```cpp
class Temperature {
private:
    double celsius; // hidden

public:
    void setCelsius(double c) {
        if (c < -273.15) throw std::invalid_argument("Below absolute zero");
        celsius = c;
    }
    double getCelsius() const { return celsius; }
    double getFahrenheit() const { return celsius * 9.0/5.0 + 32.0; }
};
```

---

### Q17. What is the `this` pointer?

**Answer:**  
`this` is an implicit pointer available inside non-static member functions that points to the object on which the method was called.

```cpp
class Builder {
    std::string name;
    int value;
public:
    Builder& setName(const std::string& n) {
        this->name = n;
        return *this; // enables method chaining
    }
    Builder& setValue(int v) {
        this->value = v;
        return *this;
    }
    void print() const {
        std::cout << name << ": " << value << "\n";
    }
};

Builder b;
b.setName("Speed").setValue(100).print(); // Speed: 100
```

---

### Q18. What is a `const` member function?

**Answer:**  
A `const` member function promises not to modify the object's observable state. It can be called on `const` objects.

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}

    double area() const {           // can be called on const Circle
        return 3.14159 * radius * radius;
    }

    void scale(double factor) {     // non-const: modifies state
        radius *= factor;
    }
};

const Circle c(5.0);
std::cout << c.area(); // OK
// c.scale(2.0);       // Error: const object
```

---

### Q19. What is operator overloading in a class?

**Answer:**  
Operator overloading allows you to define the behavior of C++ operators for user-defined types.

```cpp
class Vector2D {
public:
    double x, y;
    Vector2D(double x, double y) : x(x), y(y) {}

    Vector2D operator+(const Vector2D& rhs) const {
        return {x + rhs.x, y + rhs.y};
    }

    bool operator==(const Vector2D& rhs) const {
        return x == rhs.x && y == rhs.y;
    }

    friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
};

Vector2D a{1, 2}, b{3, 4};
std::cout << a + b; // (4, 6)
```

---

### Q20. What are `friend` functions and classes?

**Answer:**  
A `friend` declaration grants external functions or classes access to private and protected members.

```cpp
class Wallet {
    double balance = 0;
    friend class ATM; // ATM can access private members
    friend void audit(const Wallet&);
};

class ATM {
public:
    void withdraw(Wallet& w, double amount) {
        w.balance -= amount; // direct access
    }
};

void audit(const Wallet& w) {
    std::cout << "Balance: " << w.balance; // direct access
}
```

---

### Q21. What is a `mutable` member?

**Answer:**  
A `mutable` member can be modified even inside a `const` member function. Useful for caches and lazy evaluation.

```cpp
class ExpensiveCalc {
    mutable bool cached = false;
    mutable double result = 0;
    double input;

public:
    ExpensiveCalc(double v) : input(v) {}

    double compute() const {
        if (!cached) {
            result = input * input * 42.0; // expensive
            cached = true;
        }
        return result;
    }
};

const ExpensiveCalc ec(3.0);
std::cout << ec.compute(); // computes and caches
std::cout << ec.compute(); // returns cached value
```

---

### Q22. What is an initializer list in a constructor?

**Answer:**  
A member initializer list initializes member variables before the constructor body runs. It is more efficient than assignment inside the body and is **required** for `const` members, references, and base class initialization.

```cpp
class Rectangle {
    const int width;   // const: must use initializer list
    int& refHeight;    // reference: must use initializer list
    int area;

public:
    Rectangle(int w, int& h)
        : width(w), refHeight(h), area(w * h) // initializer list
    {
        // body runs after all members are initialized
    }
};
```

---

### Q23. What is the Rule of Three / Rule of Five?

**Answer:**  
- **Rule of Three:** If you define a destructor, copy constructor, or copy assignment operator, you probably need all three.
- **Rule of Five:** In C++11+, add move constructor and move assignment operator.
- **Rule of Zero:** Prefer using RAII types (smart pointers, containers) so the compiler-generated defaults work correctly.

```cpp
class MyArray {
    int* data;
    size_t size;
public:
    MyArray(size_t n) : data(new int[n]), size(n) {}
    ~MyArray() { delete[] data; }                                      // 1

    MyArray(const MyArray& o) : data(new int[o.size]), size(o.size) { // 2
        std::copy(o.data, o.data + size, data);
    }

    MyArray& operator=(const MyArray& o) {                             // 3
        if (this != &o) {
            delete[] data;
            size = o.size;
            data = new int[size];
            std::copy(o.data, o.data + size, data);
        }
        return *this;
    }

    MyArray(MyArray&& o) noexcept : data(o.data), size(o.size) {      // 4
        o.data = nullptr; o.size = 0;
    }

    MyArray& operator=(MyArray&& o) noexcept {                         // 5
        if (this != &o) { delete[] data; data = o.data; size = o.size;
                          o.data = nullptr; o.size = 0; }
        return *this;
    }
};
```

---

### Q24. What is a static member function?

**Answer:**  
A static member function belongs to the class, not any instance. It has no `this` pointer and can only access static members.

```cpp
class Logger {
    static int instanceCount;
public:
    Logger() { ++instanceCount; }
    ~Logger() { --instanceCount; }

    static int getCount() { return instanceCount; } // no 'this'
};
int Logger::instanceCount = 0;

Logger a, b, c;
std::cout << Logger::getCount(); // 3
```

---

### Q25. What is a nested class?

**Answer:**  
A class defined within another class. The nested class does not automatically have access to the outer class's private members (unless it is a `friend`).

```cpp
class LinkedList {
public:
    class Node {             // nested class
    public:
        int data;
        Node* next;
        Node(int d) : data(d), next(nullptr) {}
    };

    Node* head = nullptr;

    void push(int val) {
        Node* n = new Node(val);
        n->next = head;
        head = n;
    }
};

LinkedList list;
list.push(1);
list.push(2);
```

---

### Q26. What is polymorphism and how do virtual functions enable it?

**Answer:**  
Polymorphism allows a base class pointer/reference to call overridden methods in derived classes at runtime via the vtable mechanism.

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
};

class Square : public Shape {
    double side;
public:
    Square(double s) : side(s) {}
    double area() const override { return side * side; }
};

std::vector<std::unique_ptr<Shape>> shapes;
shapes.push_back(std::make_unique<Circle>(3));
shapes.push_back(std::make_unique<Square>(4));

for (const auto& s : shapes)
    std::cout << s->area() << "\n"; // 28.27, 16
```

---

### Q27. What is the difference between `override` and `final`?

**Answer:**  
- `override` tells the compiler the function is overriding a virtual function (helps catch typos/mismatches).
- `final` prevents further overriding of a virtual function, or prevents inheritance from a class.

```cpp
class Base {
public:
    virtual void foo() {}
    virtual void bar() {}
};

class Mid : public Base {
public:
    void foo() override {}        // OK
    void bar() override final {}  // bar cannot be overridden further
};

class Derived : public Mid {
public:
    void foo() override {}        // OK
    // void bar() override {}     // Error: bar is final
};

class Sealed final : public Base { // Sealed cannot be inherited
public:
    void foo() override {}
};
```

---

## Struct vs Class

---

### Q28. What are the key differences between `struct` and `class` in C++?

**Answer:**

| Feature | `struct` | `class` |
|---|---|---|
| Default access | `public` | `private` |
| Default inheritance | `public` | `private` |
| Intended use | Passive data grouping | Encapsulated objects with behavior |
| Templates | Yes | Yes |
| Virtual functions | Yes | Yes |

```cpp
struct Point { int x, y; };   // x, y are public
class  Point { int x, y; };   // x, y are private
```

---

### Q29. When should you use `struct` vs `class`?

**Answer:**  
Use `struct` for simple, passive data containers (no invariants, mostly public). Use `class` for objects with encapsulation, invariants, and non-trivial behavior.

```cpp
// Good struct use: simple data
struct Config {
    int width = 800;
    int height = 600;
    bool fullscreen = false;
};

// Good class use: encapsulated, invariant-preserving
class Connection {
    bool connected = false;
    std::string host;
public:
    void connect(const std::string& h);
    void disconnect();
    bool isConnected() const { return connected; }
};
```

---

### Q30. Does inheritance work differently between struct and class?

**Answer:**  
The only difference is the **default** inheritance mode. Struct defaults to `public`, class defaults to `private`.

```cpp
struct Base { int x = 1; };

struct S : Base {};            // public inheritance (default)
class  C : Base {};            // private inheritance (default)
class  C2 : public Base {};    // explicit public

S s; std::cout << s.x;        // OK
// C c; std::cout << c.x;     // Error: x is inaccessible
C2 c2; std::cout << c2.x;    // OK
```

---

### Q31. Can a struct have virtual functions?

**Answer:**  
Yes. A `struct` can have virtual functions, making it polymorphic. The compiler will add a vtable pointer, which breaks POD status.

```cpp
struct Animal {
    virtual void speak() const { std::cout << "...\n"; }
    virtual ~Animal() = default;
};

struct Cat : Animal {
    void speak() const override { std::cout << "Meow\n"; }
};

Animal* a = new Cat();
a->speak(); // Meow — runtime dispatch works
delete a;
```

---

### Q32. How does struct/class affect template type deduction?

**Answer:**  
There is no difference for templates — both `struct` and `class` keywords can be used to declare template type parameters (they are interchangeable in that context).

```cpp
template<class T>    // identical to template<typename T>
struct Stack {
    std::vector<T> data;
    void push(T val) { data.push_back(val); }
    T pop() { T v = data.back(); data.pop_back(); return v; }
};

Stack<int> s;
s.push(10);
```

---

### Q33. Can a class inherit from a struct and vice versa?

**Answer:**  
Yes. `struct` and `class` can freely inherit from each other. The keyword used for the derived type determines the default access.

```cpp
struct Data { int value = 42; };

class Wrapper : public Data {   // class inheriting from struct
public:
    int doubled() const { return value * 2; }
};

struct Extended : Wrapper {     // struct inheriting from class (public by default)
    int tripled() const { return value * 3; }
};

Extended e;
std::cout << e.doubled();  // 84
std::cout << e.tripled();  // 126
```

---

### Q34. What is the memory layout difference between a struct and a class?

**Answer:**  
None, as long as access specifiers and virtual functions are the same. The compiler lays out both identically for equivalent member declarations.

```cpp
struct S { int a; double b; };
class  C { public: int a; double b; };

static_assert(sizeof(S) == sizeof(C)); // true
static_assert(offsetof(S, a) == offsetof(C, a)); // true
```

---

### Q35. How do you pass a struct/class to a function efficiently?

**Answer:**  
- Small POD structs: pass by value (fits in registers).
- Large structs/classes: pass by `const&` to avoid copying.
- To modify: pass by pointer or non-const reference.

```cpp
struct Vec3 { float x, y, z; };         // small: 12 bytes
class Image { /* megabytes */ };

void processVec(Vec3 v) { /* by value */ }                // OK for small
void processImage(const Image& img) { /* by const ref */ }  // avoid copy
void modifyVec(Vec3& v) { v.x = 0; }                      // by ref to modify
```

---

## Abstract Class

---

### Q36. What is an abstract class in C++?

**Answer:**  
An abstract class contains at least one **pure virtual function** (`= 0`). It cannot be instantiated directly; it serves as an interface/contract for derived classes.

```cpp
class AbstractShape {
public:
    virtual double area() const = 0;     // pure virtual
    virtual double perimeter() const = 0;
    virtual ~AbstractShape() = default;

    // Can still have concrete implementation
    void printInfo() const {
        std::cout << "Area: " << area()
                  << ", Perimeter: " << perimeter() << "\n";
    }
};

// AbstractShape s; // Error: cannot instantiate abstract class
```

---

### Q37. Can an abstract class have a constructor?

**Answer:**  
Yes. Abstract classes can have constructors (used by derived classes through base initializer lists) but cannot be instantiated directly.

```cpp
class Named {
protected:
    std::string name;
public:
    Named(const std::string& n) : name(n) {}
    virtual void describe() const = 0;
    virtual ~Named() = default;
};

class Employee : public Named {
    int id;
public:
    Employee(const std::string& n, int id)
        : Named(n), id(id) {} // calls abstract base constructor

    void describe() const override {
        std::cout << "Employee[" << id << "]: " << name << "\n";
    }
};
```

---

### Q38. Can an abstract class have non-pure virtual functions?

**Answer:**  
Yes. An abstract class can mix pure virtual functions with regular virtual (or even non-virtual) functions. Derived classes inherit the concrete implementations.

```cpp
class Logger {
public:
    virtual void log(const std::string& msg) = 0; // must override

    void logError(const std::string& msg) {        // concrete
        log("[ERROR] " + msg);
    }

    void logInfo(const std::string& msg) {         // concrete
        log("[INFO] " + msg);
    }

    virtual ~Logger() = default;
};

class ConsoleLogger : public Logger {
public:
    void log(const std::string& msg) override {
        std::cout << msg << "\n";
    }
};

ConsoleLogger cl;
cl.logError("Disk full");  // [ERROR] Disk full
```

---

### Q39. What happens if a derived class doesn't implement all pure virtual functions?

**Answer:**  
The derived class also becomes abstract and cannot be instantiated.

```cpp
class A {
public:
    virtual void foo() = 0;
    virtual void bar() = 0;
};

class B : public A {
public:
    void foo() override {} // implements foo but NOT bar
    // B is still abstract because bar() is not implemented
};

class C : public B {
public:
    void bar() override {} // now both foo and bar are implemented
};

// B b; // Error: B is abstract
C c;     // OK
```

---

### Q40. Can a pure virtual function have an implementation?

**Answer:**  
Yes. A pure virtual function can still have a body (definition), which can be called explicitly via scope resolution. This is rare but useful for providing default behavior.

```cpp
class Base {
public:
    virtual void doWork() = 0;
    virtual ~Base() = default;
};

void Base::doWork() { // body for pure virtual
    std::cout << "Default work\n";
}

class Derived : public Base {
public:
    void doWork() override {
        Base::doWork();              // explicit call to base body
        std::cout << "Extra work\n";
    }
};

Derived d;
d.doWork();
// Output:
// Default work
// Extra work
```

---

### Q41. What is the vtable and how does it relate to abstract classes?

**Answer:**  
The vtable (virtual dispatch table) is a compiler-generated array of function pointers for all virtual functions of a class. Each object with virtual functions carries a hidden `vptr` pointing to its class's vtable. Abstract classes have vtable entries for pure virtuals that point to a placeholder (calling them would be undefined behavior).

```cpp
class Base {
public:
    virtual void foo() = 0;
    virtual void bar() { std::cout << "Base::bar\n"; }
    virtual ~Base() = default;
};

// Conceptual vtable for Base:
// [0] foo  -> __purevirt (trap)
// [1] bar  -> Base::bar
// [2] ~Base

class Derived : public Base {
public:
    void foo() override { std::cout << "Derived::foo\n"; }
};

// Vtable for Derived:
// [0] foo  -> Derived::foo
// [1] bar  -> Base::bar (inherited)
// [2] ~Derived
```

---

### Q42. How does the `virtual` destructor relate to abstract classes?

**Answer:**  
Abstract base classes should always declare a `virtual` destructor. Without it, deleting a derived object via a base pointer causes undefined behavior (only the base destructor runs).

```cpp
class Base {
public:
    virtual void work() = 0;
    virtual ~Base() { std::cout << "~Base\n"; }  // virtual destructor!
};

class Derived : public Base {
public:
    void work() override {}
    ~Derived() { std::cout << "~Derived\n"; }
};

Base* ptr = new Derived();
delete ptr;
// Output (correct):
// ~Derived
// ~Base

// Without virtual ~Base, only ~Base would run => resource leak
```

---

### Q43. Can an abstract class implement an interface through multiple inheritance?

**Answer:**  
Yes. C++ supports multiple inheritance, and abstract classes can inherit from multiple abstract (interface) classes.

```cpp
class IDrawable {
public:
    virtual void draw() const = 0;
    virtual ~IDrawable() = default;
};

class ISerializable {
public:
    virtual std::string serialize() const = 0;
    virtual ~ISerializable() = default;
};

class Widget : public IDrawable, public ISerializable {
    std::string label;
public:
    Widget(const std::string& l) : label(l) {}
    void draw() const override { std::cout << "Drawing: " << label << "\n"; }
    std::string serialize() const override { return "{label:" + label + "}"; }
};

Widget w("Button");
w.draw();
std::cout << w.serialize();
```

---

### Q44. What is the difference between an abstract class and a concrete class?

**Answer:**  
An abstract class has at least one pure virtual function and cannot be instantiated. A concrete class implements all pure virtual functions and can be instantiated.

```cpp
class AbstractAnimal {         // abstract
public:
    virtual void sound() = 0;
    void breathe() { std::cout << "Breathing\n"; }
    virtual ~AbstractAnimal() = default;
};

class Cat : public AbstractAnimal { // concrete
public:
    void sound() override { std::cout << "Meow\n"; }
};

// AbstractAnimal a; // Error
Cat c;               // OK
c.sound();           // Meow
c.breathe();         // Breathing
```

---

### Q45. Can you have a pointer or reference to an abstract class?

**Answer:**  
Yes! Pointers and references to abstract class types are fully valid and are the primary mechanism for polymorphism.

```cpp
class IProcessor {
public:
    virtual int process(int x) = 0;
    virtual ~IProcessor() = default;
};

class Doubler : public IProcessor {
public:
    int process(int x) override { return x * 2; }
};

class Squarer : public IProcessor {
public:
    int process(int x) override { return x * x; }
};

void run(IProcessor& proc, int val) {  // reference to abstract class
    std::cout << proc.process(val) << "\n";
}

Doubler d;
Squarer s;
run(d, 5);  // 10
run(s, 5);  // 25
```

---

### Q46. What is the Template Method Pattern using abstract classes?

**Answer:**  
The base class defines the algorithm skeleton in a concrete method, calling pure virtual "hooks" that subclasses fill in.

```cpp
class DataMiner {
public:
    // Template method — defines the algorithm
    void mine() {
        openFile();
        extractData();
        parseData();
        closeFile();
    }

protected:
    virtual void extractData() = 0; // hook
    virtual void parseData()   = 0; // hook

private:
    void openFile()  { std::cout << "Opening file\n"; }
    void closeFile() { std::cout << "Closing file\n"; }
};

class CSVMiner : public DataMiner {
    void extractData() override { std::cout << "Extracting CSV\n"; }
    void parseData()   override { std::cout << "Parsing CSV\n";    }
};

CSVMiner miner;
miner.mine();
```

---

## Interfaces / Pure Abstract Classes

---

### Q47. What is an interface in C++?

**Answer:**  
C++ has no `interface` keyword. An interface is simulated with a class that has **only** pure virtual functions and a virtual destructor (no data members, no concrete methods).

```cpp
class IComparable {
public:
    virtual bool lessThan(const IComparable& other) const = 0;
    virtual bool equals(const IComparable& other) const = 0;
    virtual ~IComparable() = default;
};
```

---

### Q48. How do you implement an interface in C++?

**Answer:**  
Inherit from the interface class and provide concrete implementations for all pure virtual functions.

```cpp
class IShape {
public:
    virtual double area() const = 0;
    virtual std::string name() const = 0;
    virtual ~IShape() = default;
};

class Triangle : public IShape {
    double base, height;
public:
    Triangle(double b, double h) : base(b), height(h) {}
    double area() const override { return 0.5 * base * height; }
    std::string name() const override { return "Triangle"; }
};

std::unique_ptr<IShape> shape = std::make_unique<Triangle>(6, 4);
std::cout << shape->name() << " area: " << shape->area(); // Triangle area: 12
```

---

### Q49. What is the difference between an interface and an abstract class in C++?

**Answer:**

| | Interface (pure abstract) | Abstract Class |
|---|---|---|
| Data members | None | Allowed |
| Concrete methods | None | Allowed |
| Constructors | Virtual destructor only | Can have constructors |
| Purpose | Contract/capability | Partial implementation |

```cpp
// Interface: pure contract
class IFlyable {
public:
    virtual void fly() = 0;
    virtual ~IFlyable() = default;
};

// Abstract class: partial implementation
class Vehicle {
protected:
    int speed;
public:
    Vehicle(int s) : speed(s) {}
    virtual void move() = 0;         // still abstract
    int getSpeed() const { return speed; } // concrete
};
```

---

### Q50. Can a class implement multiple interfaces?

**Answer:**  
Yes. C++ supports multiple inheritance, allowing a class to implement multiple interfaces.

```cpp
class IReadable {
public:
    virtual std::string read() = 0;
    virtual ~IReadable() = default;
};

class IWritable {
public:
    virtual void write(const std::string& data) = 0;
    virtual ~IWritable() = default;
};

class ISeekable {
public:
    virtual void seek(long pos) = 0;
    virtual ~ISeekable() = default;
};

class File : public IReadable, public IWritable, public ISeekable {
    std::string content;
    long position = 0;
public:
    std::string read() override { return content.substr(position); }
    void write(const std::string& data) override { content += data; }
    void seek(long pos) override { position = pos; }
};
```

---

### Q51. What is the diamond problem and how does virtual inheritance solve it?

**Answer:**  
When two base classes share a common ancestor, multiple inheritance creates two copies of the ancestor's members (the diamond problem). `virtual` inheritance ensures only one shared copy exists.

```cpp
class Device {
public:
    int id = 0;
    virtual ~Device() = default;
};

class Scanner  : virtual public Device {};
class Printer  : virtual public Device {};

class AllInOne : public Scanner, public Printer {
    // Only ONE Device subobject (virtual inheritance)
};

AllInOne aio;
aio.id = 42;          // unambiguous: one id
std::cout << aio.id;  // 42
```

---

### Q52. How do you check if an object implements an interface at runtime?

**Answer:**  
Use `dynamic_cast`. A successful cast to an interface pointer means the object implements it.

```cpp
class IFlyable {
public:
    virtual void fly() = 0;
    virtual ~IFlyable() = default;
};

class Bird : public IFlyable {
public:
    void fly() override { std::cout << "Bird flying\n"; }
};

class Dog {
public:
    virtual ~Dog() = default;
};

void tryFly(void* obj, bool isBird) {
    // Better pattern: use base class pointer
}

Bird b;
IFlyable* flyer = dynamic_cast<IFlyable*>(&b);
if (flyer) flyer->fly(); // Bird flying
```

---

### Q53. What is the Liskov Substitution Principle and how does it relate to interfaces?

**Answer:**  
LSP states that objects of a derived class must be usable wherever the base class/interface is expected without breaking correctness. Interfaces define the contract; implementations must honor it.

```cpp
class IStack {
public:
    virtual void push(int val) = 0;
    virtual int  pop()        = 0;
    virtual bool empty() const = 0;
    virtual ~IStack() = default;
};

// LSP-compliant: ArrayStack honors the IStack contract
class ArrayStack : public IStack {
    std::vector<int> data;
public:
    void push(int v) override { data.push_back(v); }
    int  pop()       override { int v = data.back(); data.pop_back(); return v; }
    bool empty() const override { return data.empty(); }
};

void processStack(IStack& s) {
    s.push(1); s.push(2);
    while (!s.empty()) std::cout << s.pop() << " ";
}

ArrayStack as;
processStack(as); // 2 1
```

---

### Q54. How do you create a factory function returning an interface pointer?

**Answer:**  
A factory function returns a smart pointer to the interface type, hiding the concrete implementation.

```cpp
class ILogger {
public:
    virtual void log(const std::string& msg) = 0;
    virtual ~ILogger() = default;
};

class ConsoleLogger : public ILogger {
public:
    void log(const std::string& msg) override {
        std::cout << "[Console] " << msg << "\n";
    }
};

class FileLogger : public ILogger {
public:
    void log(const std::string& msg) override {
        std::cout << "[File] " << msg << "\n";
    }
};

std::unique_ptr<ILogger> createLogger(const std::string& type) {
    if (type == "console") return std::make_unique<ConsoleLogger>();
    if (type == "file")    return std::make_unique<FileLogger>();
    throw std::invalid_argument("Unknown logger type");
}

auto logger = createLogger("console");
logger->log("Hello"); // [Console] Hello
```

---

### Q55. What is the Interface Segregation Principle?

**Answer:**  
ISP (from SOLID) states that clients should not be forced to depend on interfaces they don't use. Prefer many small, focused interfaces over one large interface.

```cpp
// Violation: one fat interface
class IAnimal {
public:
    virtual void walk()  = 0;
    virtual void swim()  = 0;
    virtual void fly()   = 0; // forces Fish to implement fly!
};

// Better: segregated interfaces
class IWalkable { public: virtual void walk() = 0; virtual ~IWalkable() = default; };
class ISwimmable { public: virtual void swim() = 0; virtual ~ISwimmable() = default; };
class IFlyable2  { public: virtual void fly()  = 0; virtual ~IFlyable2() = default; };

class Duck  : public IWalkable, public ISwimmable, public IFlyable2 {
    void walk() override { std::cout << "Duck walks\n"; }
    void swim() override { std::cout << "Duck swims\n"; }
    void fly()  override { std::cout << "Duck flies\n"; }
};

class Fish : public ISwimmable {
    void swim() override { std::cout << "Fish swims\n"; }
};
```

---

## Value Types & Value Semantics

---

### Q56. What are value types in C++?

**Answer:**  
Value types store their data directly (not via a pointer). Copying a value type creates an independent copy. Primitives (`int`, `double`) and most structs/classes with proper copy semantics are value types.

```cpp
int a = 5;
int b = a;    // independent copy
b = 10;
std::cout << a; // still 5

struct Point { int x, y; };
Point p1{1, 2};
Point p2 = p1;  // deep copy
p2.x = 99;
std::cout << p1.x; // still 1
```

---

### Q57. What is value semantics?

**Answer:**  
Value semantics means objects behave like values: copying produces equal but independent objects, assignment replaces the value, and equality is by content (not identity). `std::string`, `std::vector`, and `std::array` all have value semantics.

```cpp
std::string s1 = "hello";
std::string s2 = s1;      // independent copy
s2 += " world";

std::cout << s1; // "hello"  — unchanged
std::cout << s2; // "hello world"

// Equality is by value, not address
std::cout << (s1 == "hello"); // true
```

---

### Q58. What is the difference between pass-by-value and pass-by-reference?

**Answer:**  
- **By value:** A copy is made; modifications don't affect the original.
- **By reference:** An alias is created; modifications affect the original.

```cpp
void byValue(int x) {
    x = 99;                  // modifies local copy
}

void byReference(int& x) {
    x = 99;                  // modifies original
}

int n = 5;
byValue(n);     std::cout << n; // 5
byReference(n); std::cout << n; // 99
```

---

### Q59. What is a copy constructor and when is it called?

**Answer:**  
The copy constructor initializes a new object as a copy of an existing one. It is called during object initialization from another object, when passing by value, and when returning by value (though RVO may elide it).

```cpp
class Box {
public:
    int value;
    Box(int v) : value(v) {}
    Box(const Box& other) : value(other.value) {
        std::cout << "Copy constructor called\n";
    }
};

Box a(10);
Box b = a;          // copy constructor
Box c(a);           // copy constructor

void take(Box x) {} // copy constructor when called
take(a);
```

---

### Q60. What is copy elision and RVO (Return Value Optimization)?

**Answer:**  
The compiler may eliminate unnecessary copies when returning local objects. Named RVO (NRVO) handles named local variables; RVO handles temporaries. Since C++17, RVO on temporaries is **guaranteed**.

```cpp
class Heavy {
public:
    Heavy() { std::cout << "Constructed\n"; }
    Heavy(const Heavy&) { std::cout << "Copied\n"; }
    Heavy(Heavy&&) { std::cout << "Moved\n"; }
};

Heavy makeHeavy() {
    return Heavy(); // RVO: no copy/move, constructed in-place
}

Heavy h = makeHeavy();
// Output (with RVO): "Constructed" — no copy!
```

---

### Q61. What is move semantics and how do they relate to value types?

**Answer:**  
Move semantics transfer ownership of resources rather than copying them, making value-type operations on large objects cheap. The move constructor/assignment operator "steal" the internals of a temporary (rvalue).

```cpp
std::vector<int> makeData() {
    std::vector<int> v(1000000, 42);
    return v; // moved (or NRVO), not copied
}

std::vector<int> data = makeData(); // cheap: O(1) move, not O(n) copy

std::vector<int> a(1000, 1);
std::vector<int> b = std::move(a); // a is now empty
std::cout << a.size(); // 0
std::cout << b.size(); // 1000
```

---

### Q62. What is `std::move` and when should you use it?

**Answer:**  
`std::move` is a cast to an rvalue reference, signaling that the object can be "moved from". Use it when you no longer need the original and want to transfer ownership cheaply. **Do not use a moved-from object** (it's in a valid but unspecified state).

```cpp
std::string s1 = "Hello, World!";
std::string s2 = std::move(s1);  // s1 is moved-from

std::cout << s2;             // "Hello, World!"
std::cout << s1.empty();     // likely true (unspecified state)

// In class: move constructor takes rvalue reference
class Buffer {
    std::vector<int> data;
public:
    Buffer(Buffer&& other) noexcept : data(std::move(other.data)) {}
};
```

---

### Q63. What are `std::pair` and `std::tuple` as value types?

**Answer:**  
Both are value types that hold a fixed number of heterogeneous values. They have full value semantics (copyable, moveable, comparable).

```cpp
// std::pair
std::pair<std::string, int> nameAge{"Alice", 30};
auto [name, age] = nameAge;  // structured bindings (C++17)
std::cout << name << " is " << age; // Alice is 30

// std::tuple
auto record = std::make_tuple("Bob", 25, 175.5);
std::cout << std::get<0>(record); // Bob
std::cout << std::get<2>(record); // 175.5

// With structured bindings
auto [n, a, h] = record;
std::cout << n << " " << a << " " << h;
```

---

### Q64. What is `std::optional` and how does it model optional value semantics?

**Answer:**  
`std::optional<T>` (C++17) represents a value of type `T` that may or may not be present. It has full value semantics and avoids raw pointer/null patterns for optional values.

```cpp
#include <optional>

std::optional<int> divide(int a, int b) {
    if (b == 0) return std::nullopt;
    return a / b;
}

auto result = divide(10, 2);
if (result) std::cout << *result; // 5

auto bad = divide(10, 0);
std::cout << bad.has_value();     // false
std::cout << bad.value_or(-1);    // -1 (default)
```

---

### Q65. What is slicing in value semantics?

**Answer:**  
Slicing occurs when a derived class object is copied into a base class variable by value, losing derived members and virtual dispatch.

```cpp
class Base {
public:
    int x = 1;
    virtual std::string who() const { return "Base"; }
};

class Derived : public Base {
public:
    int y = 2;
    std::string who() const override { return "Derived"; }
};

Derived d;
Base b = d;        // SLICING: y is lost, vtable is Base's

std::cout << b.who();   // "Base" — not "Derived"!
std::cout << b.x;       // 1
// b.y;                 // Error: y doesn't exist in Base

// Solution: use pointer/reference
Base& ref = d;
std::cout << ref.who(); // "Derived" — no slicing
```

---

## Reference Types & Reference Semantics

---

### Q66. What are references in C++ and how do they work?

**Answer:**  
A reference is an alias for an existing object. It must be initialized at declaration and cannot be reseated (made to refer to a different object). Unlike a pointer, it cannot be null.

```cpp
int x = 10;
int& ref = x;      // ref is an alias for x

ref = 20;
std::cout << x;    // 20 — same object modified

int y = 5;
// ref = y;        // This assigns y's value to x (through ref), doesn't reseat ref
std::cout << x;    // 5 (x now holds 5)
std::cout << (&ref == &x); // true — same address
```

---

### Q67. What is the difference between lvalue and rvalue references?

**Answer:**  
- **lvalue reference (`T&`):** Binds to named, persistent objects (lvalues).
- **rvalue reference (`T&&`):** Binds to temporaries or moved-from objects (rvalues). Used for move semantics and perfect forwarding.

```cpp
int x = 5;
int& lref = x;       // lvalue reference — binds to x
// int& bad = 5;     // Error: can't bind lvalue ref to rvalue

int&& rref = 10;     // rvalue reference — binds to temporary
rref = 20;           // OK: can modify through rvalue ref

std::string s = "hello";
std::string&& rstr = std::move(s); // rvalue ref to moved s
```

---

### Q68. What is a `const` reference and why is it useful?

**Answer:**  
A `const T&` can bind to both lvalues and rvalues (including temporaries), and guarantees no modification. It is the canonical way to accept large objects in function parameters without copying.

```cpp
struct BigData { std::vector<int> items; };

void process(const BigData& data) {  // no copy, can't modify
    std::cout << data.items.size();
}

BigData bd{{1, 2, 3, 4, 5}};
process(bd);                      // lvalue: no copy
process(BigData{{10, 20, 30}});   // rvalue: binds temp to const ref
```

---

### Q69. What is reference semantics in C++?

**Answer:**  
Reference semantics means multiple names can refer to the same underlying object. Changes through one name are visible through another. Pointers and references implement reference semantics.

```cpp
class Counter {
public:
    int count = 0;
    void increment() { ++count; }
};

Counter c;
Counter& ref1 = c;
Counter& ref2 = c;

ref1.increment();
ref2.increment();

std::cout << c.count;    // 2 — all names see the same object
std::cout << ref1.count; // 2
```

---

### Q70. How do pointers differ from references?

**Answer:**

| Feature | Pointer (`T*`) | Reference (`T&`) |
|---|---|---|
| Nullable | Yes (`nullptr`) | No |
| Reseat | Yes (reassignable) | No |
| Syntax | `*ptr`, `ptr->` | `ref`, `ref.` |
| Arithmetic | Yes | No |
| Arrays | Yes | No (use `std::span`) |

```cpp
int x = 1, y = 2;

// Pointer
int* ptr = &x;
ptr = &y;           // reseated to y
*ptr = 99;

// Reference
int& ref = x;
// ref = y;         // assigns y's value to x, doesn't reseat
ref = 42;           // x is now 42

// Pointer can be null
int* nptr = nullptr;
if (nptr) { /* safe */ }

// int& nref; // Error: reference must be initialized
```

---

### Q71. What are smart pointers and why prefer them over raw pointers?

**Answer:**  
Smart pointers (`unique_ptr`, `shared_ptr`, `weak_ptr`) automatically manage the lifetime of heap objects, preventing leaks and dangling references.

```cpp
#include <memory>

// unique_ptr: sole ownership
auto u = std::make_unique<std::string>("hello");
std::cout << *u;          // "hello"
// Automatically deleted when u goes out of scope

// shared_ptr: shared ownership
auto s1 = std::make_shared<int>(42);
auto s2 = s1;             // reference count = 2
std::cout << s1.use_count(); // 2

// weak_ptr: non-owning observer (breaks cycles)
std::weak_ptr<int> w = s1;
if (auto locked = w.lock()) {
    std::cout << *locked; // 42
}
```

---

### Q72. What is `std::ref` and `std::cref`?

**Answer:**  
`std::ref` and `std::cref` create reference wrappers (`std::reference_wrapper<T>`), which behave like references but are copyable — useful when passing references to algorithms or `std::bind`.

```cpp
#include <functional>

void addTen(int& x) { x += 10; }

int a = 5;
auto f = std::bind(addTen, std::ref(a)); // reference wrapper
f();
std::cout << a; // 15 — a was modified

// Without std::ref, bind copies a
auto g = std::bind(addTen, a); // copy of a
g();
std::cout << a; // still 15 — original unchanged
```

---

### Q73. What is a dangling reference and how do you avoid it?

**Answer:**  
A dangling reference is a reference that outlives the object it refers to, resulting in undefined behavior.

```cpp
// DANGEROUS: returns reference to local
int& dangerous() {
    int local = 42;
    return local; // local destroyed at }
}                 // caller holds dangling reference!

int& r = dangerous(); // UB: r refers to destroyed local
std::cout << r;       // undefined behavior

// SAFE: return by value
int safe() {
    int local = 42;
    return local; // copy returned
}

// SAFE: return reference to object that outlives function
int global = 100;
int& safeGlobal() { return global; }
```

---

### Q74. What is perfect forwarding and `std::forward`?

**Answer:**  
Perfect forwarding preserves the value category (lvalue/rvalue) of arguments when passing them to inner functions. It uses forwarding references (`T&&` in a template) and `std::forward<T>`.

```cpp
template<typename T>
void wrapper(T&& arg) {  // forwarding reference (not rvalue ref)
    inner(std::forward<T>(arg));  // preserves lvalue/rvalue
}

void inner(int& x)  { std::cout << "lvalue: " << x << "\n"; }
void inner(int&& x) { std::cout << "rvalue: " << x << "\n"; }

int x = 10;
wrapper(x);       // lvalue: 10
wrapper(42);      // rvalue: 42
wrapper(std::move(x)); // rvalue: 10
```

---

### Q75. What is a reference-counted class and how does it compare to value semantics?

**Answer:**  
Reference-counted types (like `std::shared_ptr`) allow multiple owners. Copying increments the count; destruction decrements it. This is reference semantics (multiple handles, one object) vs value semantics (copy = independent object).

```cpp
// Value semantics: each copy is independent
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = v1;
v2.push_back(4);
std::cout << v1.size(); // 3 — unchanged

// Reference semantics via shared_ptr
auto p1 = std::make_shared<std::vector<int>>(std::initializer_list<int>{1, 2, 3});
auto p2 = p1;          // both point to same vector
p2->push_back(4);
std::cout << p1->size(); // 4 — shared!
```

---

## Mixed & Advanced Topics

---

### Q76. What is RAII and how do structs/classes implement it?

**Answer:**  
RAII (Resource Acquisition Is Initialization) ties resource lifetime to object lifetime: acquire in the constructor, release in the destructor. This makes resource management exception-safe.

```cpp
class FileHandle {
    FILE* handle;
public:
    FileHandle(const char* path, const char* mode)
        : handle(fopen(path, mode)) {
        if (!handle) throw std::runtime_error("Cannot open file");
    }
    ~FileHandle() {
        if (handle) fclose(handle); // always runs, even if exception thrown
    }

    // Non-copyable, moveable
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    FILE* get() { return handle; }
};

// Usage: no manual cleanup needed
{
    FileHandle f("data.txt", "r");
    // use f.get()...
} // fclose called automatically
```

---

### Q77. What is the difference between `new`/`delete` and stack allocation for classes?

**Answer:**  
Stack allocation is automatic (destructor called when out of scope, no overhead). Heap allocation (`new`/`delete`) gives explicit lifetime control but requires manual management (use smart pointers).

```cpp
class Heavy {
    std::array<int, 10000> data{};
public:
    Heavy()  { std::cout << "Constructed\n"; }
    ~Heavy() { std::cout << "Destructed\n"; }
};

// Stack allocation: automatic lifetime
{
    Heavy h;         // constructed
}                    // destructed automatically

// Heap allocation: manual lifetime
Heavy* p = new Heavy();      // constructed
// ... use p ...
delete p;                    // destructed manually

// Prefer smart pointers:
auto sp = std::make_unique<Heavy>(); // constructed
// sp goes out of scope -> destructed automatically
```

---

### Q78. What is CRTP (Curiously Recurring Template Pattern)?

**Answer:**  
CRTP achieves static polymorphism by having a base class template take the derived class as a template parameter. It avoids virtual dispatch overhead.

```cpp
template<typename Derived>
class Printable {
public:
    void print() const {
        // static dispatch — no vtable needed
        static_cast<const Derived*>(this)->printImpl();
    }
};

class Point : public Printable<Point> {
    int x, y;
public:
    Point(int x, int y) : x(x), y(y) {}
    void printImpl() const {
        std::cout << "Point(" << x << ", " << y << ")\n";
    }
};

class Color : public Printable<Color> {
    int r, g, b;
public:
    Color(int r, int g, int b) : r(r), g(g), b(b) {}
    void printImpl() const {
        std::cout << "Color(" << r << "," << g << "," << b << ")\n";
    }
};

Point p{3, 4};
Color c{255, 0, 128};
p.print(); // Point(3, 4)
c.print(); // Color(255, 0, 128)
```

---

### Q79. What is the difference between composition and inheritance?

**Answer:**  
- **Inheritance** ("is-a"): Derives behavior. Tightly couples classes. Prefer for polymorphism.
- **Composition** ("has-a"): Embeds objects. Loosely couples classes. More flexible.

```cpp
// Inheritance: Car IS-A Vehicle
class Vehicle {
protected:
    int speed;
public:
    Vehicle(int s) : speed(s) {}
    virtual void move() { std::cout << "Moving at " << speed << "\n"; }
};
class Car : public Vehicle {
public:
    Car(int s) : Vehicle(s) {}
    void move() override { std::cout << "Car driving at " << speed << "\n"; }
};

// Composition: Car HAS-A Engine (preferred for reusability)
class Engine {
public:
    void start() { std::cout << "Engine started\n"; }
    void stop()  { std::cout << "Engine stopped\n"; }
};

class ElectricCar {
    Engine engine; // composed
    int    speed;
public:
    ElectricCar(int s) : speed(s) {}
    void drive() { engine.start(); std::cout << "Driving\n"; }
    void park()  { engine.stop(); }
};
```

---

### Q80. How do you design a type-safe heterogeneous container using interfaces?

**Answer:**  
Use a base interface with smart pointers to store objects of different concrete types in a single container while retaining polymorphic behavior.

```cpp
class ITask {
public:
    virtual void execute() = 0;
    virtual std::string name() const = 0;
    virtual ~ITask() = default;
};

class PrintTask : public ITask {
    std::string msg;
public:
    PrintTask(std::string m) : msg(std::move(m)) {}
    void execute() override { std::cout << msg << "\n"; }
    std::string name() const override { return "PrintTask"; }
};

class ComputeTask : public ITask {
    int a, b;
public:
    ComputeTask(int a, int b) : a(a), b(b) {}
    void execute() override { std::cout << "Result: " << a + b << "\n"; }
    std::string name() const override { return "ComputeTask"; }
};

// Heterogeneous container via interface
std::vector<std::unique_ptr<ITask>> taskQueue;
taskQueue.push_back(std::make_unique<PrintTask>("Hello, C++!"));
taskQueue.push_back(std::make_unique<ComputeTask>(7, 35));
taskQueue.push_back(std::make_unique<PrintTask>("Done."));

for (auto& task : taskQueue) {
    std::cout << "[" << task->name() << "] ";
    task->execute();
}
// [PrintTask]   Hello, C++!
// [ComputeTask] Result: 42
// [PrintTask]   Done.
```

---

## Quick Reference Summary

| Concept | Key Point |
|---|---|
| `struct` default access | `public` |
| `class` default access | `private` |
| Abstract class | Has ≥1 pure virtual function (`= 0`) |
| Interface (C++) | Pure abstract class — all methods pure virtual |
| Value semantics | Copy = independent duplicate |
| Reference semantics | Multiple names → same object |
| Rule of Five | Destructor + copy ctor + copy= + move ctor + move= |
| RAII | Resource lifetime tied to object lifetime |
| Slicing | Derived copied into base by value → loses derived info |
| Diamond problem | Solved with `virtual` inheritance |
| CRTP | Static polymorphism without vtable overhead |
| `std::move` | Casts to rvalue ref; enables move semantics |
| `std::forward` | Preserves value category in templates |

---

*Good luck with your C++ interviews!*
