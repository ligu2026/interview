Ah, I see! You want **30 questions per subject** (total ~300 questions). That's a comprehensive request. I'll provide **30 tricky questions with answers and examples** for each of the 10 subjects.

Due to length, I'll start with the **first 2 subjects fully**, then offer to continue with the remaining 8.

---

# SUBJECT 1: Memory Management (30 Questions)

## Q1: What's the difference between stack and heap allocation?
**A:** Stack is automatic, fast, limited size. Heap is manual, slower, flexible.
```cpp
int stackVar = 5;           // Stack
int* heapVar = new int(5);  // Heap
delete heapVar;
```

## Q2: What's a memory leak? Give an example.
**A:** Memory allocated but never freed.
```cpp
void leak() {
    int* p = new int[100];
    // missing delete[] p;
}
```

## Q3: How does RAII solve memory leaks?
**A:** Resources acquired in constructor, released in destructor.
```cpp
class SmartArray {
    int* data;
public:
    SmartArray(size_t n) : data(new int[n]) {}
    ~SmartArray() { delete[] data; }
};
```

## Q4: What's a dangling pointer?
**A:** Pointer to already freed memory.
```cpp
int* p = new int(5);
delete p;
*p = 10; // Dangling pointer - UB
```

## Q5: Double-free bug - what is it?
**A:** Deleting same pointer twice.
```cpp
int* p = new int(5);
delete p;
delete p; // Double-free - UB
```

## Q6: How does `unique_ptr` prevent double-free?
**A:** Copy is deleted; only move transfers ownership.
```cpp
unique_ptr<int> p1(new int(5));
unique_ptr<int> p2 = std::move(p1); // OK, p1 now null
// unique_ptr<int> p3 = p1; // Compile error
```

## Q7: How to create a `unique_ptr` correctly?
**A:** Use `make_unique` (C++14) for exception safety.
```cpp
auto p = std::make_unique<int>(42); // Preferred
// auto p = std::unique_ptr<int>(new int(42)); // OK but not exception-safe
```

## Q8: What's `shared_ptr`'s control block?
**A:** Contains reference count, weak count, deleter, allocator.
```cpp
shared_ptr<int> sp1(new int(5)); // control block created
shared_ptr<int> sp2 = sp1;       // ref count = 2
```

## Q9: What's circular reference with `shared_ptr`? Example?
**A:** Objects referencing each other, never freed.
```cpp
struct Node {
    shared_ptr<Node> next;
};
auto a = make_shared<Node>();
auto b = make_shared<Node>();
a->next = b;
b->next = a; // memory leak
```

## Q10: How does `weak_ptr` break circular references?
**A:** `weak_ptr` doesn't increase ref count.
```cpp
struct Node {
    weak_ptr<Node> next; // break cycle
};
```

## Q11: Can you have `make_unique` for array? Syntax?
**A:** Yes, C++14:
```cpp
auto arr = make_unique<int[]>(10); // array of 10 ints
```

## Q12: How to create custom deleter with `unique_ptr`?
**A:** Second template parameter.
```cpp
auto deleter = [](FILE* f) { fclose(f); };
unique_ptr<FILE, decltype(deleter)> fp(fopen("t.txt","r"), deleter);
```

## Q13: When is `shared_ptr<T>(new T)` dangerous?
**A:** If constructor throws between new and shared_ptr creation.
```cpp
f(shared_ptr<int>(new int(5)), may_throw()); // leak risk
make_shared<int>(5); // safe
```

## Q14: What's aliasing constructor of `shared_ptr`?
**A:** Shares ownership of one object but points to another.
```cpp
struct Pair { int a, b; };
auto sp = make_shared<Pair>();
shared_ptr<int> ap(sp, &sp->a); // points to a, owns Pair
```

## Q15: Can `weak_ptr` be converted to `shared_ptr`? How?
**A:** Via `lock()` which returns `shared_ptr` or empty.
```cpp
weak_ptr<int> wp;
if (auto sp = wp.lock()) { // safe access
    cout << *sp;
}
```

## Q16: What's `std::enable_shared_from_this`?
**A:** Allows class to return `shared_ptr` to itself.
```cpp
class Can : public enable_shared_from_this<Can> {
public:
    shared_ptr<Can> getptr() { return shared_from_this(); }
};
auto p1 = make_shared<Can>();
auto p2 = p1->getptr(); // same ownership
```

## Q17: What's wrong with `return shared_ptr<T>(this)`?
**A:** Creates separate control blocks → double-free.
```cpp
class Bad {
public:
    shared_ptr<Bad> get() { return shared_ptr<Bad>(this); }
};
auto b1 = make_shared<Bad>();
auto b2 = b1->get(); // double control blocks → disaster
```

## Q18: What's `std::allocate_shared`?
**A:** Creates `shared_ptr` with custom allocator.
```cpp
auto sp = std::allocate_shared<int>(std::allocator<int>(), 42);
```

## Q19: Performance: `make_shared` vs `new` + `shared_ptr`?
**A:** `make_shared` allocates object + control block in one allocation (faster, less memory fragmentation).
```cpp
auto p1 = make_shared<int>(5);        // One allocation
shared_ptr<int> p2(new int(5));       // Two allocations
```

## Q20: When can't you use `make_shared`?
**A:** Custom deleters, private constructors, aligned allocations.
```cpp
shared_ptr<int> p(new int[10], [](int* p){ delete[] p; }); // Can't use make_shared
```

## Q21: What's `std::owner_less`?
**A:** Compares `shared_ptr`/`weak_ptr` by ownership (not value).
```cpp
set<weak_ptr<int>, owner_less<weak_ptr<int>>> s; // needed for weak_ptr in set
```

## Q22: How to check if `unique_ptr` is null?
**A:** Implicit conversion to bool.
```cpp
unique_ptr<int> p;
if (!p) cout << "null";
if (p == nullptr) cout << "null";
```

## Q23: Can you have `shared_ptr<void>`? Use case?
**A:** Yes - type-erased ownership.
```cpp
shared_ptr<void> p(new int(5));
// later: static_cast<int*>(p.get())
```

## Q24: What's `std::default_delete`?
**A:** Default deleter for `unique_ptr`.
```cpp
template<typename T>
struct default_delete {
    void operator()(T* p) const { delete p; }
};
```

## Q25: How `unique_ptr` works with incomplete types?
**A:** Deleter must be defined before destructor instantiation (PIMPL idiom).
```cpp
// In header
class Impl;
struct Deleter { void operator()(Impl*) const; };
unique_ptr<Impl, Deleter> p;
```

## Q26: What's the size of `unique_ptr` and `shared_ptr`?
**A:** `unique_ptr` = raw pointer size (if no custom deleter with state). `shared_ptr` = 2 raw pointers (object + control block).
```cpp
cout << sizeof(unique_ptr<int>);   // 8 (on 64-bit)
cout << sizeof(shared_ptr<int>);   // 16
```

## Q27: What's `std::atomic_shared_ptr` (C++20)?
**A:** Atomic operations on `shared_ptr` (lock-free optional).
```cpp
atomic<shared_ptr<int>> asp;
shared_ptr<int> sp = make_shared<int>(5);
asp.store(sp);
```

## Q28: What's memory corruption from buffer overflow?
**A:** Writing beyond allocated memory.
```cpp
int* p = new int[5];
p[10] = 42; // Corruption - UB
delete[] p;
```

## Q29: Tool to detect memory leaks?
**A:** Valgrind, AddressSanitizer (ASan), Dr. Memory.
```bash
g++ -fsanitize=address -g program.cpp
```

## Q30: What's placement new? Memory leak risk?
**A:** Constructs object in pre-allocated memory.
```cpp
char buffer[sizeof(MyClass)];
MyClass* p = new(buffer) MyClass(); // placement new
p->~MyClass(); // Must call destructor manually - leak if forgotten
```

---

# SUBJECT 2: OOP & C++ Class Fundamentals (30 Questions)

## Q1: What's the order of constructor calls in inheritance?
**A:** Base → Derived. Virtual bases before direct bases, left-to-right in declaration order.
```cpp
struct A { A() { cout << "A "; } };
struct B : A { B() { cout << "B "; } };
B b; // Output: A B
```

## Q2: Destructor order in inheritance?
**A:** Derived → Base.
```cpp
struct A { ~A() { cout << "A "; } };
struct B : A { ~B() { cout << "B "; } };
B b; // Output: B A (when destroyed)
```

## Q3: Virtual destructor - why needed?
**A:** To ensure derived destructor called when deleting through base pointer.
```cpp
struct Base { virtual ~Base() {} };
struct Derived : Base { ~Derived() { cout << "cleanup\n"; } };
Base* p = new Derived();
delete p; // Calls Derived destructor (virtual)
```

## Q4: Pure virtual function - syntax and effect?
**A:** `virtual void f() = 0;` - makes class abstract.
```cpp
class Shape { public: virtual double area() = 0; };
// Shape s; // Error: abstract class
class Circle : public Shape { double area() override { return 3.14; } };
```

## Q5: Can pure virtual function have a body?
**A:** Yes, but still makes class abstract.
```cpp
class Base {
public:
    virtual void f() = 0;
};
void Base::f() { cout << "default\n"; } // Implementation
class Derived : Base { void f() override { Base::f(); } };
```

## Q6: What's the "Rule of Three"?
**A:** If class needs custom destructor, copy constructor, or copy assignment, it needs all three.
```cpp
class String {
    char* data;
public:
    ~String() { delete[] data; }
    String(const String& other);  // copy ctor
    String& operator=(const String& other); // copy assign
};
```

## Q7: Rule of Five (C++11)?
**A:** Rule of Three + move constructor + move assignment.
```cpp
class String {
    char* data;
public:
    String(String&& other) noexcept; // move ctor
    String& operator=(String&& other) noexcept; // move assign
};
```

## Q8: What's copy elision? When is it mandatory?
**A:** Compiler optimization to skip copy/move. In C++17, prvalues guarantee no copy.
```cpp
vector<int> make_vec() { return {1,2,3}; }
vector<int> v = make_vec(); // No copy (guaranteed in C++17)
```

## Q9: What's object slicing?
**A:** Assigning derived object to base object by value cuts off derived part.
```cpp
struct Base { int a; };
struct Derived : Base { int b; };
Derived d;
Base b = d; // b lost d.b (slicing)
```

## Q10: How to prevent slicing?
**A:** Use pointers/references, or disable copy.
```cpp
Base& b = d; // OK - no slice
Base* p = &d; // OK
```

## Q11: What's the vtable? When is it created?
**A:** Virtual table of function pointers. One per class, created at compile time.
```cpp
class Base { virtual void f() {} }; // compiler creates vtable for Base
```

## Q12: vptr - what and when initialized?
**A:** Virtual pointer pointing to vtable. Set in constructor of each class (base to derived).
```cpp
class B { virtual void f() {} };
class D : B { virtual void f() {} };
// D's vptr points to D's vtable after D's ctor runs
```

## Q13: Multiple inheritance - diamond problem?
**A:** Two paths to same base class → ambiguity.
```cpp
struct A { int x; };
struct B : A {};
struct C : A {};
struct D : B, C {}; // D has two A::x copies
```

## Q14: Virtual inheritance solves diamond problem how?
**A:** Only one instance of virtual base shared.
```cpp
struct A { int x; };
struct B : virtual A {};
struct C : virtual A {};
struct D : B, C {}; // single A subobject
```

## Q15: Cost of virtual inheritance?
**A:** Extra indirection (vptr for virtual base offset), larger objects, slower access.
```cpp
D d;
d.x; // works, but has overhead to find A's location
```

## Q16: Can a constructor be virtual? Why not?
**A:** No. vptr not set until constructor runs. Virtual mechanism depends on vptr.

## Q17: Can a destructor be pure virtual?
**A:** Yes, but must provide a body (because derived destructors call it).
```cpp
class Base {
public:
    virtual ~Base() = 0;
};
Base::~Base() {} // Required
```

## Q18: What's the "Rule of Zero"?
**A:** Avoid custom copy/move/dtor; use RAII types (smart pointers, containers).
```cpp
class Person {
    string name; // string manages memory
    unique_ptr<Address> addr; // smart pointer manages Address
    // No custom copy/move/dtor needed
};
```

## Q19: Override keyword - purpose?
A:** Checks that function actually overrides virtual function.
```cpp
class Base { virtual void f(int); };
class Derived : Base {
    void f(double) override; // Error: doesn't override
};
```

## Q20: Final keyword with classes and functions?
A:** Prevents further overriding/inheritance.
```cpp
class Base final {};        // Cannot be inherited
class Derived {
    virtual void f() final; // Cannot be overridden
};
```

## Q21: What's a delegating constructor?
**A:** Constructor calls another constructor in same class (C++11).
```cpp
class Point {
    int x, y;
public:
    Point() : Point(0, 0) {} // delegates
    Point(int a, int b) : x(a), y(b) {}
};
```

## Q22: Inheriting constructors (C++11)?
**A:** Brings base constructors into derived scope.
```cpp
struct B {
    B(int) {}
    B(double) {}
};
struct D : B {
    using B::B; // Inherit both constructors
};
```

## Q23: Can you call virtual function in constructor?
**A:** Yes, but only current class version called (not final override).
```cpp
class Base {
public:
    Base() { f(); } // calls Base::f, not Derived::f
    virtual void f() { cout << "Base"; }
};
```

## Q24: What's a covariant return type?
**A:** Override can return pointer/reference to derived class.
```cpp
struct Base { virtual Base* clone() const; };
struct Derived : Base { Derived* clone() const override; }; // allowed
```

## Q25: Difference between struct and class?
**A:** struct: public inheritance/members by default; class: private.
```cpp
struct S { int a; };        // a is public
class C { int a; };         // a is private
struct D : S {};            // public inheritance
class E : S {};             // private inheritance
```

## Q26: What's the "Curiously Recurring Template Pattern" (CRTP)?
**A:** Derived class inherits from template instantiated with itself.
```cpp
template<typename T>
class Base {
    void interface() { static_cast<T*>(this)->impl(); }
};
class Derived : public Base<Derived> {
    void impl() { cout << "CRTP"; }
}; // Static polymorphism - no vtable
```

## Q27: Can you have static virtual function?
**A:** No. Virtual dispatch needs `this` pointer (object instance).

## Q28: What's the "most vexing parse"?
**A:** Ambiguity between function declaration and object definition.
```cpp
vector<int> v(istream_iterator<int>(cin), istream_iterator<int>());
// Parsed as function declaration! Fix:
vector<int> v{istream_iterator<int>(cin), istream_iterator<int>()};
```

## Q29: What's object lifetime? When does it start/end?
**A:** Starts after constructor completes. Ends when destructor starts.
```cpp
class X {
public:
    X() { cout << "ctor start"; } // object not alive yet
    // after ctor ends: object alive
    ~X() { cout << "dtor start"; } // object still alive
    // after dtor ends: object dead
};
```

## Q30: What's `std::launder` (C++17)? When needed?
**A:** Tells compiler that object changed address. Needed with placement new in const objects or unions.
```cpp
struct X { const int i; };
X* p = new X{42};
new (p) X{100};           // p->i may still be 42 due to optimization
cout << std::launder(p)->i; // 100 guaranteed
```

---

I've provided **60 questions (30 + 30)** covering **Memory Management** and **OOP & Class Fundamentals**.

**Shall I continue with the remaining 8 subjects?** Each will have 30 questions with examples. Total would be 300 questions. I can provide them in:
1. **One massive reply** (but may hit length limits)
2. **8 separate replies** (you choose topic order)
3. **Downloadable document format**

Which approach would you prefer? Also, if you want me to adjust difficulty level or focus more on certain subtopics within each subject, let me know!