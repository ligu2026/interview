# Top 50 C++ Interview Questions
## Constructors & Destructors

---

## Table of Contents
1. [Constructor Fundamentals (Q1–Q10)](#constructor-fundamentals)
2. [Constructor Types (Q11–Q22)](#constructor-types)
3. [Destructor Fundamentals (Q23–Q30)](#destructor-fundamentals)
4. [Inheritance & Polymorphism (Q31–Q38)](#inheritance--polymorphism)
5. [Special Member Functions & Rules (Q39–Q46)](#special-member-functions--rules)
6. [Advanced Topics (Q47–Q50)](#advanced-topics)

---

## Constructor Fundamentals

---

### Q1. What is a constructor in C++?

**Answer:**  
A constructor is a special member function that is automatically called when an object is created. It has the same name as the class and no return type (not even `void`). Its purpose is to initialize the object's data members to a valid state.

```cpp
class Car {
    std::string brand;
    int year;
public:
    Car(std::string b, int y) : brand(b), year(y) {
        std::cout << "Car created: " << brand << "\n";
    }
};

Car c("Toyota", 2024); // Output: Car created: Toyota
```

---

### Q2. What are the properties of a constructor?

**Answer:**  
- Same name as the class
- No return type (not even `void`)
- Can be overloaded (multiple constructors with different parameters)
- Cannot be `virtual` (but can call virtual functions — with caveats)
- Cannot be `const`, `volatile`, or `static`
- Automatically called at object creation

```cpp
class Point {
public:
    int x, y;
    Point()          : x(0), y(0) {}       // default
    Point(int a, int b) : x(a), y(b) {}   // parameterized
    // Both are valid constructors — overloaded
};
```

---

### Q3. What is a member initializer list and why is it preferred?

**Answer:**  
A member initializer list directly initializes members before the constructor body runs. It is **required** for `const` members, reference members, and base class initialization. It is also more efficient — it avoids default-constructing and then assigning.

```cpp
class Employee {
    const int id;
    std::string name;
    double salary;
public:
    // Initializer list — preferred:
    Employee(int i, std::string n, double s)
        : id(i), name(std::move(n)), salary(s) {}

    // Constructor body assignment — worse:
    // Employee(int i, std::string n, double s) {
    //     id = i;      // ERROR: const member can't be assigned
    //     name = n;    // default-constructs name, then assigns
    //     salary = s;
    // }
};
```

---

### Q4. In what order are members initialized?

**Answer:**  
Members are **always** initialized in the order they are **declared in the class**, regardless of the order in the initializer list. Writing the initializer list in a different order than the declaration order can cause subtle bugs.

```cpp
class Ordered {
    int a;
    int b;
    int c;
public:
    // Even though list says b, c, a — initialized as a, b, c
    Ordered(int v) : b(v), c(b * 2), a(c + 1) {
        // a is initialized FIRST (using c, which isn't initialized yet!)
        // This is a bug! Always match declaration order.
    }
};

class Safe {
    int a, b, c;
public:
    Safe(int v) : a(v), b(a * 2), c(b + 1) {} // correct order
};
```

---

### Q5. What is a default constructor?

**Answer:**  
A default constructor takes no arguments (or has all default arguments). The compiler auto-generates one if no user-defined constructor is declared. If any constructor is declared, the compiler does NOT auto-generate a default constructor.

```cpp
class Implicit {
    int x = 0; // in-class initializer
    // No constructor declared — compiler generates:
    // Implicit() = default;
};

class Explicit {
    int x;
public:
    Explicit(int v) : x(v) {} // user constructor declared
    // NO implicit default constructor!
};

Implicit a;   // OK
// Explicit b; // ERROR: no default constructor
Explicit c(5); // OK
```

---

### Q6. What does `= default` mean for a constructor?

**Answer:**  
`= default` explicitly requests the compiler to generate the default implementation. Useful when you've declared other constructors (which suppress the auto-generated default) but still need the default behavior.

```cpp
class Widget {
    int value;
public:
    Widget() = default;           // compiler-generated default
    Widget(int v) : value(v) {}  // user-defined parameterized

    Widget(const Widget&) = default;            // default copy
    Widget& operator=(const Widget&) = default; // default assignment
};

Widget a;      // uses = default constructor
Widget b(42);  // uses parameterized constructor
```

---

### Q7. What does `= delete` mean for a constructor?

**Answer:**  
`= delete` explicitly disables a constructor, making it a compile error to call it. Used to prevent copying, default construction, or unwanted conversions.

```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;            // no copy
    NonCopyable& operator=(const NonCopyable&) = delete; // no copy-assign
};

NonCopyable a;
// NonCopyable b = a; // ERROR: use of deleted function
// NonCopyable c(a);  // ERROR: use of deleted function

class NoDefault {
public:
    NoDefault() = delete;        // prevent default construction
    NoDefault(int v) {}
};
// NoDefault x;  // ERROR
NoDefault y(5); // OK
```

---

### Q8. What is in-class member initialization (C++11)?

**Answer:**  
You can provide default values for member variables directly in the class body. These are used if the constructor's initializer list doesn't override them.

```cpp
class Config {
    int timeout   = 30;
    std::string host = "localhost";
    bool verbose  = false;
public:
    Config() {}                          // uses all defaults
    Config(int t) : timeout(t) {}       // overrides timeout only
    Config(std::string h, bool v)
        : host(std::move(h)), verbose(v) {} // overrides host and verbose
};

Config a;           // timeout=30, host="localhost", verbose=false
Config b(60);       // timeout=60, host="localhost", verbose=false
Config c("prod", true); // timeout=30, host="prod", verbose=true
```

---

### Q9. What is a delegating constructor (C++11)?

**Answer:**  
A delegating constructor calls another constructor of the same class from its initializer list, avoiding code duplication. The delegated constructor runs first, then the delegating constructor's body.

```cpp
class Rectangle {
    int width, height;
    std::string color;

    void validate() {
        if (width <= 0 || height <= 0)
            throw std::invalid_argument("Invalid dimensions");
    }
public:
    Rectangle(int w, int h, std::string c)
        : width(w), height(h), color(std::move(c)) { validate(); }

    Rectangle(int w, int h)
        : Rectangle(w, h, "white") {} // delegates — validate() called once

    Rectangle()
        : Rectangle(1, 1) {}          // delegates to above
};

Rectangle r1(10, 5, "red"); // full constructor
Rectangle r2(8, 4);         // delegates → Rectangle(8,4,"white")
Rectangle r3;               // delegates → Rectangle(1,1)
```

---

### Q10. Can a constructor throw an exception?

**Answer:**  
Yes. If a constructor throws, the object is considered not fully constructed — its destructor will **NOT** be called. However, destructors of already-constructed base classes and member subobjects **will** be called. This makes RAII with smart pointers critical for exception safety.

```cpp
class SafeResource {
    std::unique_ptr<int> p;
    std::unique_ptr<int> q;
public:
    SafeResource()
        : p(std::make_unique<int>(1)),
          q(std::make_unique<int>(2)) {
        if (someCondition) throw std::runtime_error("Init failed");
        // If thrown: p and q's destructors ARE called (RAII)
        // SafeResource's destructor is NOT called
    }
};
```

---

## Constructor Types

---

### Q11. What is a parameterized constructor?

**Answer:**  
A constructor that accepts one or more arguments, allowing objects to be initialized with specific values at creation time.

```cpp
class Color {
    int r, g, b;
public:
    Color(int red, int green, int blue)
        : r(red), g(green), b(blue) {}

    void print() const {
        std::cout << "RGB(" << r << "," << g << "," << b << ")";
    }
};

Color red(255, 0, 0);
Color green(0, 255, 0);
red.print();   // Output: RGB(255,0,0)
```

---

### Q12. What is a copy constructor?

**Answer:**  
A copy constructor creates a new object as a copy of an existing one. Its signature is `ClassName(const ClassName&)`. The compiler generates one by default that performs memberwise copy.

```cpp
class Matrix {
    int rows, cols;
    int* data;
public:
    Matrix(int r, int c) : rows(r), cols(c), data(new int[r * c]{}) {}

    // Copy constructor — deep copy
    Matrix(const Matrix& other)
        : rows(other.rows), cols(other.cols),
          data(new int[other.rows * other.cols]) {
        std::copy(other.data, other.data + rows * cols, data);
        std::cout << "Copy constructor called\n";
    }

    ~Matrix() { delete[] data; }
};

Matrix m1(3, 3);
Matrix m2 = m1;  // copy constructor called
Matrix m3(m1);   // also copy constructor
```

---

### Q13. What is a move constructor (C++11)?

**Answer:**  
A move constructor transfers ownership of resources from a temporary (rvalue) object to a new object, avoiding expensive deep copies. Takes an rvalue reference `ClassName&&`. The source is left in a valid but empty state.

```cpp
class Buffer {
    size_t size;
    int* data;
public:
    Buffer(size_t s) : size(s), data(new int[s]{}) {}

    // Move constructor
    Buffer(Buffer&& other) noexcept
        : size(other.size), data(other.data) {
        other.size = 0;
        other.data = nullptr; // transfer ownership
        std::cout << "Move constructor called\n";
    }

    ~Buffer() { delete[] data; }
};

Buffer b1(1000);
Buffer b2 = std::move(b1); // move constructor — no allocation
// b1.data is now nullptr
```

---

### Q14. What is a converting constructor?

**Answer:**  
A constructor with a single non-default argument that enables **implicit type conversion** from that argument's type to the class type. Marking it `explicit` prevents implicit conversion.

```cpp
class Celsius {
    double temp;
public:
    Celsius(double t) : temp(t) {} // converting constructor
    double get() const { return temp; }
};

Celsius c = 36.6;   // implicit conversion from double — allowed
std::cout << c.get(); // Output: 36.6

class Kelvin {
    double temp;
public:
    explicit Kelvin(double t) : temp(t) {} // explicit: no implicit conversion
};

// Kelvin k = 300.0; // ERROR: explicit blocks implicit conversion
Kelvin k(300.0);    // OK: direct initialization
```

---

### Q15. What is an explicit constructor and when should you use it?

**Answer:**  
`explicit` prevents a constructor from being used for implicit conversions or copy-initialization. Use it on any single-argument constructor where implicit conversion would be surprising or unsafe.

```cpp
class SmartInt {
    int val;
public:
    explicit SmartInt(int v) : val(v) {}
    int get() const { return val; }
};

void print(SmartInt s) { std::cout << s.get(); }

// print(42);        // ERROR: implicit conversion blocked
print(SmartInt(42)); // OK: explicit construction
SmartInt s = SmartInt(10); // OK: explicit
// SmartInt s2 = 10; // ERROR: copy-initialization blocked
```

---

### Q16. What is a copy assignment operator?

**Answer:**  
The copy assignment operator (`operator=`) is called when assigning one **existing** object from another existing object. Unlike the copy constructor, the target object already exists and may hold resources that need to be released first.

```cpp
class String {
    char* data;
    size_t len;
public:
    String(const char* s = "") : len(strlen(s)), data(new char[len+1]) {
        strcpy(data, s);
    }

    String& operator=(const String& other) {
        if (this == &other) return *this; // self-assignment guard
        delete[] data;                    // release old resource
        len  = other.len;
        data = new char[len + 1];         // allocate new
        strcpy(data, other.data);         // copy data
        return *this;
    }

    ~String() { delete[] data; }
};

String a("hello");
String b("world");
b = a; // copy assignment operator called
```

---

### Q17. What is a move assignment operator (C++11)?

**Answer:**  
The move assignment operator transfers resources from a temporary (rvalue) to an existing object. It must release the current resources first, then steal from the source.

```cpp
class Buffer {
    int* data;
    size_t size;
public:
    Buffer(size_t s) : size(s), data(new int[s]{}) {}

    Buffer& operator=(Buffer&& other) noexcept {
        if (this == &other) return *this;
        delete[] data;           // release current resources
        data = other.data;       // steal from source
        size = other.size;
        other.data = nullptr;    // leave source in valid state
        other.size = 0;
        return *this;
    }

    ~Buffer() { delete[] data; }
};

Buffer a(100);
Buffer b(200);
b = std::move(a); // move assignment — no allocation
```

---

### Q18. What is the Rule of Three?

**Answer:**  
If a class defines any one of these, it almost certainly needs to define all three:
1. Destructor
2. Copy constructor
3. Copy assignment operator

This is because any class that manages a resource (raw pointer, file handle, etc.) needs custom behavior for all three operations.

```cpp
class RawArray {
    int* data;
    int  size;
public:
    RawArray(int n) : size(n), data(new int[n]{}) {}  // acquires resource

    ~RawArray() { delete[] data; }                    // 1. Destructor

    RawArray(const RawArray& o)                       // 2. Copy constructor
        : size(o.size), data(new int[o.size]) {
        std::copy(o.data, o.data + size, data);
    }

    RawArray& operator=(const RawArray& o) {          // 3. Copy assignment
        if (this == &o) return *this;
        delete[] data;
        size = o.size;
        data = new int[size];
        std::copy(o.data, o.data + size, data);
        return *this;
    }
};
```

---

### Q19. What is the Rule of Five (C++11)?

**Answer:**  
Extends the Rule of Three to include move semantics:
1. Destructor
2. Copy constructor
3. Copy assignment operator
4. Move constructor
5. Move assignment operator

Defining any one implies you should consider all five.

```cpp
class Complete {
    int* data;
    size_t size;
public:
    Complete(size_t n) : size(n), data(new int[n]{}) {}

    ~Complete()                                  { delete[] data; }               // 1

    Complete(const Complete& o)                  // 2
        : size(o.size), data(new int[o.size])
        { std::copy(o.data, o.data+size, data); }

    Complete& operator=(const Complete& o) {     // 3
        if (this != &o) {
            delete[] data;
            size = o.size;
            data = new int[size];
            std::copy(o.data, o.data+size, data);
        }
        return *this;
    }

    Complete(Complete&& o) noexcept              // 4
        : size(o.size), data(o.data)
        { o.data = nullptr; o.size = 0; }

    Complete& operator=(Complete&& o) noexcept { // 5
        if (this != &o) {
            delete[] data;
            data = o.data; size = o.size;
            o.data = nullptr; o.size = 0;
        }
        return *this;
    }
};
```

---

### Q20. What is the Rule of Zero?

**Answer:**  
The Rule of Zero says: if your class doesn't directly manage resources, define **none** of the five special functions — let the compiler generate them all. Use smart pointers and standard containers so that resource management is handled by existing RAII types.

```cpp
// Rule of Zero — no special functions needed:
class Person {
    std::string name;                  // manages its own memory
    std::vector<int> scores;           // manages its own memory
    std::unique_ptr<Address> address;  // manages its own memory
public:
    Person(std::string n, std::unique_ptr<Address> a)
        : name(std::move(n)), address(std::move(a)) {}
    // Compiler generates correct destructor, copy, move automatically
};
```

---

### Q21. What is the difference between copy construction and copy assignment?

**Answer:**  
- **Copy construction**: the object is being **created** for the first time. No prior resources to release.
- **Copy assignment**: the object **already exists**. Must handle self-assignment and release old resources before acquiring new ones.

```cpp
MyClass a(10);     // regular construction
MyClass b = a;     // COPY CONSTRUCTION (b doesn't exist yet)
MyClass c(20);
c = a;             // COPY ASSIGNMENT (c already exists, must release old data)
```

---

### Q22. What is NRVO (Named Return Value Optimization)?

**Answer:**  
NRVO is a compiler optimization that constructs a return value directly in the caller's stack frame, eliminating the copy or move constructor call when returning a local object by value.

```cpp
class Heavy {
public:
    Heavy() { std::cout << "Constructed\n"; }
    Heavy(const Heavy&) { std::cout << "Copied\n"; }
    Heavy(Heavy&&) { std::cout << "Moved\n"; }
};

Heavy make() {
    Heavy h;     // constructed directly in caller's location (NRVO)
    return h;    // no copy, no move — elided by compiler
}

int main() {
    Heavy obj = make();
    // Output: Constructed
    // (no "Copied" or "Moved" — NRVO in action)
}
```

---

## Destructor Fundamentals

---

### Q23. What is a destructor?

**Answer:**  
A destructor is a special member function called automatically when an object goes out of scope or is `delete`d. It has the same name as the class prefixed with `~` and takes no parameters, has no return type, and cannot be overloaded.

```cpp
class FileHandle {
    FILE* file;
public:
    FileHandle(const char* name) {
        file = fopen(name, "r");
        std::cout << "File opened\n";
    }

    ~FileHandle() {
        if (file) fclose(file);
        std::cout << "File closed\n";
    }
};

{
    FileHandle fh("data.txt"); // opened
} // destroyed here — file closed automatically
```

---

### Q24. When is a destructor called?

**Answer:**  
The destructor is called when:
1. A local (stack) object goes out of scope
2. A `delete` expression is executed on a heap object
3. A temporary object's lifetime ends
4. A program exits (for static/global objects)
5. An exception propagates past the object's scope (stack unwinding)

```cpp
class Tracer {
    std::string name;
public:
    Tracer(std::string n) : name(n) { std::cout << name << " born\n"; }
    ~Tracer() { std::cout << name << " died\n"; }
};

{
    Tracer a("A");
    Tracer b("B");
    Tracer c("C");
} // Output: C died, B died, A died  (LIFO order)
```

---

### Q25. What is the order of destruction for local objects?

**Answer:**  
Local objects are destroyed in **reverse order of construction** (LIFO — Last In, First Out), same as how a stack works.

```cpp
class Item {
    int id;
public:
    Item(int i) : id(i) { std::cout << "Created " << id << "\n"; }
    ~Item() { std::cout << "Destroyed " << id << "\n"; }
};

{
    Item x(1); // created first
    Item y(2);
    Item z(3); // created last
}
// Output:
// Created 1 / Created 2 / Created 3
// Destroyed 3 / Destroyed 2 / Destroyed 1
```

---

### Q26. Can a destructor throw an exception?

**Answer:**  
Technically yes, but it is **strongly discouraged** and considered dangerous. If a destructor throws during stack unwinding (exception already in flight), `std::terminate()` is called. Always declare destructors `noexcept`.

```cpp
class Dangerous {
public:
    ~Dangerous() noexcept(false) {
        throw std::runtime_error("Destructor threw!"); // dangerous
    }
};

// Better practice:
class Safe {
public:
    ~Safe() noexcept { // default for destructors in C++11
        try { cleanup(); }
        catch (...) { /* swallow — never let destructor throw */ }
    }
    void cleanup() { /* may throw */ }
};
```

---

### Q27. Can a destructor be called explicitly?

**Answer:**  
Yes, but this is almost never needed in normal code. Explicit destructor calls are used in advanced scenarios like placement new, where you manage object lifetime manually.

```cpp
#include <new>

alignas(MyClass) char buffer[sizeof(MyClass)];
MyClass* obj = new(buffer) MyClass(42);  // placement new

obj->~MyClass(); // explicit destructor call — required with placement new
// Do NOT call delete on obj here!
```

---

### Q28. What is a virtual destructor and when is it required?

**Answer:**  
A virtual destructor ensures that when a derived class object is deleted through a base class pointer, the **derived class destructor** is called first (then base). Without it, only the base destructor runs — causing resource leaks in derived classes.

```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base destroyed\n"; }
};

class Derived : public Base {
    int* data;
public:
    Derived() : data(new int[100]) {}
    ~Derived() {
        delete[] data; // would NOT be called without virtual destructor!
        std::cout << "Derived destroyed\n";
    }
};

Base* obj = new Derived();
delete obj;
// Output:
// Derived destroyed
// Base destroyed
```

---

### Q29. What is a pure virtual destructor?

**Answer:**  
A class can have a pure virtual destructor (`virtual ~Base() = 0`) to make the class abstract while still requiring a body definition (since all destructors in the hierarchy are called). This is the only pure virtual function that **must** have a definition.

```cpp
class AbstractBase {
public:
    virtual ~AbstractBase() = 0; // pure virtual
};

AbstractBase::~AbstractBase() {  // body required!
    std::cout << "AbstractBase destroyed\n";
}

class Concrete : public AbstractBase {
public:
    ~Concrete() { std::cout << "Concrete destroyed\n"; }
};

// AbstractBase ab; // ERROR: abstract class
Concrete c;
// Output on destruction:
// Concrete destroyed
// AbstractBase destroyed
```

---

### Q30. What happens to the destructor during exception stack unwinding?

**Answer:**  
When an exception is thrown, all local objects in the current scope and enclosing scopes are destroyed (their destructors are called) as the stack unwinds. This is the RAII guarantee — resources are always released even during exceptions.

```cpp
class Guard {
    std::string name;
public:
    Guard(std::string n) : name(n) { std::cout << name << " acquired\n"; }
    ~Guard() { std::cout << name << " released\n"; }
};

void riskyFunction() {
    Guard g1("Lock");
    Guard g2("File");
    throw std::runtime_error("Something failed");
    // ~Guard for g2 then g1 are called during unwinding
}

try { riskyFunction(); }
catch (const std::exception& e) { std::cout << "Caught: " << e.what(); }
// Output:
// Lock acquired / File acquired / File released / Lock released / Caught: Something failed
```

---

## Inheritance & Polymorphism

---

### Q31. In what order are base class and derived class constructors called?

**Answer:**  
Constructors are called **base-first, derived-last**: virtual bases → non-virtual bases (left to right) → member subobjects (declaration order) → the class's own constructor body.

```cpp
class A {
public:
    A() { std::cout << "A "; }
    ~A() { std::cout << "~A "; }
};
class B : public A {
public:
    B() { std::cout << "B "; }
    ~B() { std::cout << "~B "; }
};
class C : public B {
public:
    C() { std::cout << "C "; }
    ~C() { std::cout << "~C "; }
};

C obj;
// Construction: A B C
// Destruction:  ~C ~B ~A
```

---

### Q32. How do you call a base class constructor from a derived class?

**Answer:**  
Call the base class constructor in the derived class's initializer list.

```cpp
class Animal {
    std::string name;
    int age;
public:
    Animal(std::string n, int a) : name(std::move(n)), age(a) {
        std::cout << "Animal: " << name << "\n";
    }
};

class Dog : public Animal {
    std::string breed;
public:
    Dog(std::string n, int a, std::string b)
        : Animal(std::move(n), a), breed(std::move(b)) { // call base constructor
        std::cout << "Dog breed: " << breed << "\n";
    }
};

Dog d("Rex", 3, "Labrador");
// Output: Animal: Rex / Dog breed: Labrador
```

---

### Q33. Can a derived class inherit constructors (C++11)?

**Answer:**  
Yes, using `using Base::Base;` in the derived class. This brings all base class constructors into scope for the derived class.

```cpp
class Base {
public:
    Base(int x) { std::cout << "Base(" << x << ")\n"; }
    Base(int x, int y) { std::cout << "Base(" << x << "," << y << ")\n"; }
};

class Derived : public Base {
public:
    using Base::Base; // inherit all Base constructors
    // No need to redeclare Base(int) or Base(int,int)
};

Derived d1(10);      // Output: Base(10)
Derived d2(10, 20);  // Output: Base(10,20)
```

---

### Q34. What happens if you call a virtual function from a constructor?

**Answer:**  
During construction, the virtual dispatch mechanism is **not fully active**. Calling a virtual function from a constructor calls the version of the function for the **class being constructed**, not the most derived override. This is often a source of bugs.

```cpp
class Base {
public:
    Base() { init(); }               // calls Base::init — NOT Derived::init!
    virtual void init() { std::cout << "Base::init\n"; }
};

class Derived : public Base {
public:
    Derived() : Base() {}
    void init() override { std::cout << "Derived::init\n"; }
};

Derived d;
// Output: Base::init  (NOT Derived::init — vtable not fully set up yet)
```

---

### Q35. What is the order of destruction in inheritance?

**Answer:**  
Destructors are called in **reverse order of construction**: derived class first, then base class. This ensures derived members are cleaned up before base members they may depend on.

```cpp
class Resource {
public:
    Resource()  { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource released\n"; }
};

class Base {
    Resource r;
public:
    Base()  { std::cout << "Base created\n"; }
    virtual ~Base() { std::cout << "Base destroyed\n"; }
};

class Derived : public Base {
public:
    Derived()  { std::cout << "Derived created\n"; }
    ~Derived() { std::cout << "Derived destroyed\n"; }
};

{
    Derived d;
}
// Output:
// Resource acquired → Base created → Derived created
// Derived destroyed → Base destroyed → Resource released
```

---

### Q36. What is the slicing problem with constructors?

**Answer:**  
When a derived class object is assigned to a base class object (by value), the derived-specific data is "sliced off". The base copy constructor/assignment is called, losing the derived portion.

```cpp
class Base { public: int x = 1; virtual std::string type() { return "Base"; } };
class Derived : public Base { public: int y = 2; std::string type() override { return "Derived"; } };

Derived d;
Base b = d; // SLICING! y is lost; b.type() returns "Base"

std::cout << b.type(); // Output: Base (not Derived)

// Fix: use pointers or references
Base* bp = new Derived();
std::cout << bp->type(); // Output: Derived (no slicing via pointer)
```

---

### Q37. Can a base class constructor call a derived class method safely?

**Answer:**  
No — it is unsafe (see Q34). During the base constructor's execution, the derived part of the object hasn't been constructed yet. Calling derived virtual methods leads to calling the base version or, worse, to undefined behavior.

```cpp
class Base {
public:
    Base() {
        // virtualMethod(); // UNSAFE: derived part not yet constructed
        nonVirtualMethod(); // Safe: no virtual dispatch
    }
    void nonVirtualMethod() { std::cout << "Base safe call\n"; }
    virtual void virtualMethod() { std::cout << "Base virtual\n"; }
};
```

---

### Q38. What is constructor chaining and how is it different from delegating constructors?

**Answer:**  
"Constructor chaining" informally means using one constructor to initialize an object in terms of another. In C++11+, **delegating constructors** are the formal mechanism — one constructor calls another of the **same class**. This is different from calling a base class constructor (which is base class initialization, not delegation).

```cpp
class Settings {
    int width, height;
    bool fullscreen;
public:
    Settings(int w, int h, bool fs)     // primary constructor
        : width(w), height(h), fullscreen(fs) {}

    Settings(int w, int h)              // delegates to primary
        : Settings(w, h, false) {}

    Settings()                          // delegates further
        : Settings(1920, 1080) {}
};

Settings s1;           // 1920 x 1080, not fullscreen
Settings s2(800, 600); // 800 x 600, not fullscreen
Settings s3(800, 600, true); // 800 x 600, fullscreen
```

---

## Special Member Functions & Rules

---

### Q39. What are the six special member functions in C++11?

**Answer:**  
C++ automatically generates up to six special member functions:
1. Default constructor
2. Copy constructor
3. Copy assignment operator
4. Destructor
5. Move constructor (C++11)
6. Move assignment operator (C++11)

The compiler only generates each if certain conditions are met and it's not suppressed by user declarations.

```cpp
class Auto {
    int x;
    std::string s;
    // All 6 special functions compiler-generated:
    // Auto() — default
    // Auto(const Auto&) — copy ctor
    // Auto& operator=(const Auto&) — copy assign
    // ~Auto() — destructor
    // Auto(Auto&&) — move ctor
    // Auto& operator=(Auto&&) — move assign
};
```

---

### Q40. When does declaring a destructor suppress move operations?

**Answer:**  
Declaring a user-defined destructor suppresses auto-generation of the move constructor and move assignment operator (C++11 rule). Copy operations are still generated (but deprecated behavior). Use `= default` to restore them explicitly.

```cpp
class Gotcha {
public:
    ~Gotcha() {}  // user destructor → move operations suppressed!
    // Copy operations still generated (deprecated)
    // Move constructor: NOT generated
    // Move assignment: NOT generated
};

class Fixed {
public:
    ~Fixed() = default;
    Fixed(Fixed&&) = default;            // explicitly restore
    Fixed& operator=(Fixed&&) = default; // explicitly restore
};
```

---

### Q41. What is `noexcept` on constructors and destructors?

**Answer:**  
`noexcept` tells the compiler the function won't throw. Destructors are implicitly `noexcept` since C++11. Move constructors should be `noexcept` — `std::vector` will use move instead of copy during reallocation only if the move constructor is `noexcept`.

```cpp
class SafeMove {
    int* data;
    size_t size;
public:
    SafeMove(size_t s) : size(s), data(new int[s]{}) {}

    SafeMove(SafeMove&& o) noexcept  // noexcept enables vector optimization
        : size(o.size), data(o.data)
        { o.data = nullptr; o.size = 0; }

    ~SafeMove() noexcept { delete[] data; } // implicitly noexcept anyway
};

std::vector<SafeMove> v;
v.push_back(SafeMove(10)); // uses move (noexcept confirmed)
```

---

### Q42. What is the impact of `explicit` on constructors with multiple parameters (C++11)?

**Answer:**  
From C++11, `explicit` can be applied to constructors with any number of parameters. C++20 allows `explicit(condition)` for conditional explicitness.

```cpp
class Pair {
public:
    int a, b;
    explicit Pair(int x, int y) : a(x), b(y) {}
};

// Pair p = {1, 2}; // ERROR: explicit blocks braced-init
Pair p(1, 2);       // OK: direct init
Pair q{1, 2};       // OK: direct braced-init

// C++20 conditional explicit:
template<typename T>
class Wrapper {
    T val;
public:
    template<typename U>
    explicit(!std::is_convertible_v<U, T>)
    Wrapper(U&& v) : val(std::forward<U>(v)) {}
};
```

---

### Q43. How does `std::initializer_list` interact with constructors?

**Answer:**  
If a class has a constructor taking `std::initializer_list<T>`, it takes priority over other constructors when braced-initialization is used. This can cause surprising behavior.

```cpp
#include <initializer_list>
class MyVec {
public:
    MyVec(int size, int val) {
        std::cout << "Size ctor: " << size << " vals of " << val << "\n";
    }
    MyVec(std::initializer_list<int> list) {
        std::cout << "List ctor: " << list.size() << " elements\n";
    }
};

MyVec a(5, 10);   // uses (int, int) constructor
MyVec b{5, 10};   // uses initializer_list! → "List ctor: 2 elements"
MyVec c{};        // uses initializer_list (empty)

// std::vector has the same behavior:
std::vector<int> v1(3, 0); // 3 zeros
std::vector<int> v2{3, 0}; // {3, 0} — two elements!
```

---

### Q44. What is the difference between direct initialization and copy initialization?

**Answer:**  
- **Direct initialization** — `Type obj(args)` or `Type obj{args}`: calls the constructor directly; `explicit` constructors are allowed.
- **Copy initialization** — `Type obj = value`: creates from a copy/conversion; `explicit` constructors are **NOT** allowed.

```cpp
class Strict {
public:
    explicit Strict(int x) { std::cout << "Created " << x; }
};

Strict a(5);      // Direct init — OK
Strict b{5};      // Direct init — OK
// Strict c = 5;  // Copy init — ERROR: explicit constructor
// Strict d = Strict(5); // OK: explicitly constructed, then move-init
```

---

### Q45. How does `constexpr` work with constructors?

**Answer:**  
A `constexpr` constructor allows objects of the class to be created at compile time (as `constexpr` objects). The constructor body must be evaluable at compile time.

```cpp
class Point {
    int x, y;
public:
    constexpr Point(int x, int y) : x(x), y(y) {}
    constexpr int getX() const { return x; }
    constexpr int getY() const { return y; }
    constexpr int distSquared() const { return x*x + y*y; }
};

constexpr Point origin(0, 0);
constexpr Point p(3, 4);
constexpr int d = p.distSquared(); // evaluated at compile time
static_assert(d == 25);            // verified at compile time
```

---

### Q46. What is aggregate initialization and when are constructors bypassed?

**Answer:**  
An **aggregate** (a class/struct with no user-declared constructors, no private/protected non-static data members, no base classes, and no virtual functions) can be initialized with a brace-enclosed list — no constructor needed.

```cpp
struct Point { int x; int y; int z; }; // aggregate

Point p = {1, 2, 3};  // aggregate init — no constructor called
Point q{4, 5, 6};     // also aggregate init

// C++20 designated initializers:
Point r{.x = 10, .y = 20, .z = 30};

// Adding a constructor makes it non-aggregate:
struct NotAggregate {
    int x;
    NotAggregate(int v) : x(v) {} // user constructor → no longer aggregate
};
```

---

## Advanced Topics

---

### Q47. What is placement new and how does it interact with constructors/destructors?

**Answer:**  
Placement new constructs an object at a pre-allocated memory location. The constructor is called on that memory. You must call the destructor **explicitly** — `delete` must not be used on placement-new objects.

```cpp
#include <new>
alignas(std::string) char buffer[sizeof(std::string)];

// Construct in pre-allocated buffer
std::string* sp = new(buffer) std::string("hello");
std::cout << *sp; // Output: hello

sp->~std::string(); // explicit destructor call — required!
// delete sp;        // WRONG: would try to free stack memory!
```

---

### Q48. What is a factory function pattern replacing constructors?

**Answer:**  
Factory functions (static methods returning objects) provide named construction, can return derived types, and allow `shared_ptr` construction with `enable_shared_from_this` patterns.

```cpp
class Connection {
    std::string host;
    int port;

    Connection(std::string h, int p) : host(std::move(h)), port(p) {}

public:
    static std::unique_ptr<Connection> create(std::string host, int port) {
        if (port <= 0 || port > 65535)
            throw std::invalid_argument("Invalid port");
        return std::unique_ptr<Connection>(new Connection(std::move(host), port));
    }

    static std::unique_ptr<Connection> createLocal(int port) {
        return create("localhost", port); // named variant
    }
};

auto conn = Connection::create("example.com", 8080);
auto local = Connection::createLocal(3000);
```

---

### Q49. What is the two-phase construction anti-pattern?

**Answer:**  
Two-phase construction splits initialization into a constructor and a separate `init()` method. This is an anti-pattern because objects can exist in an unusable (half-constructed) state. Prefer completing initialization in the constructor or using factory functions.

```cpp
// Anti-pattern:
class BadSocket {
    int fd = -1;
public:
    BadSocket() {} // phase 1 — not usable yet!
    bool init(const std::string& host) { // phase 2 — easy to forget
        fd = connect(host);
        return fd >= 0;
    }
    void send(const std::string& msg) {
        if (fd < 0) throw std::logic_error("Not initialized!"); // guard needed
    }
};

// Better: single-phase construction
class GoodSocket {
    int fd;
public:
    GoodSocket(const std::string& host) : fd(connect(host)) {
        if (fd < 0) throw std::runtime_error("Connection failed");
        // Object is always valid after construction
    }
    void send(const std::string& msg) { /* fd always valid */ }
};
```

---

### Q50. How do constructors and destructors interact with `std::vector` reallocation?

**Answer:**  
When a `vector` reallocates, it must move or copy all existing elements to the new buffer. It calls the move constructor if it is `noexcept`, otherwise falls back to the copy constructor. Destructors are called on the old elements after the move/copy.

```cpp
class Traced {
    int id;
public:
    Traced(int i) : id(i) { std::cout << "Ctor " << id << "\n"; }
    Traced(const Traced& o) : id(o.id) { std::cout << "Copy " << id << "\n"; }
    Traced(Traced&& o) noexcept : id(o.id) { std::cout << "Move " << id << "\n"; }
    ~Traced() { std::cout << "Dtor " << id << "\n"; }
};

std::vector<Traced> v;
v.reserve(1); // capacity = 1
v.emplace_back(1); // Ctor 1
v.emplace_back(2); // Reallocation! Move 1, then Ctor 2, Dtor old-1
// With noexcept move: Move is used
// Without noexcept: Copy would be used instead
```

---

## Quick Reference

```cpp
// --- Constructor types ---
class MyClass {
    int x; const int id; std::string name;
public:
    MyClass()                    // default
        : x(0), id(0), name("") {}
    explicit MyClass(int v)      // converting (explicit)
        : x(v), id(v), name("") {}
    MyClass(int v, std::string n) // parameterized
        : x(v), id(v), name(std::move(n)) {}
    MyClass(const MyClass& o)    // copy
        : x(o.x), id(o.id), name(o.name) {}
    MyClass(MyClass&& o) noexcept // move
        : x(o.x), id(o.id), name(std::move(o.name)) {}
    MyClass& operator=(const MyClass& o) { // copy assign
        if (this != &o) { x = o.x; name = o.name; }
        return *this;
    }
    MyClass& operator=(MyClass&& o) noexcept { // move assign
        if (this != &o) { x = o.x; name = std::move(o.name); }
        return *this;
    }
    ~MyClass() {}                // destructor
    MyClass(int v, int dummy)
        : MyClass(v, "") {}      // delegating constructor
};

// --- Key rules ---
// Rule of Zero:  manage no resources → declare none of the 5
// Rule of Three: destructor → also need copy ctor + copy assign
// Rule of Five:  Rule of Three + move ctor + move assign
// Always: noexcept on move ctor/assign and destructor
// Always: virtual destructor on polymorphic base classes
// Always: initializer list over constructor body assignment
// Never:  call virtual functions from constructors/destructors
// Never:  throw from destructors
```

---

*End of Top 50 C++ Constructor & Destructor Interview Questions*
