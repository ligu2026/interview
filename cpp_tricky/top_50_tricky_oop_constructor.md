Here are **50 tricky C++ interview questions** focusing on **OOP & C++ Class Fundamentals** — specifically **constructors/destructors, copy/move semantics, polymorphism, virtual functions, vtable, abstract classes, and inheritance pitfalls** — with answers and code examples.

---

## 1. What is the output?

```cpp
class A { public: A() { cout << "A"; } };
class B : public A { public: B() { cout << "B"; } };
int main() { B b; }
```

**Answer:**  
`AB` — base constructor always called first.

---

## 2. What is the copy constructor? When is it called?

**Answer:**  
Constructs new object as copy. Called when passing by value, returning by value, or direct initialization.

```cpp
A(const A& other);
A a1; A a2 = a1; // copy ctor
```

---

## 3. What is the difference between copy constructor and copy assignment operator?

**Answer:**  
Copy ctor creates new object; copy assignment assigns to existing.

```cpp
A a1 = a2; // copy ctor (a1 didn't exist)
a1 = a3;   // copy assignment (a1 already exists)
```

---

## 4. What is the output?

```cpp
class A {
    int* p;
public:
    A() : p(new int(10)) {}
    ~A() { delete p; }
};
void f(A a) {}
int main() { A a; f(a); }
```

**Answer:**  
Double-free → crash or UB (default copy ctor does shallow copy).

**Fix:** Implement copy ctor or delete it.

---

## 5. What is the rule of three? Five? Zero?

**Answer:**  
- Rule of three: If you define destructor, copy ctor, or copy assign, define all three.
- Rule of five: Add move ctor and move assign.
- Rule of zero: Use smart pointers → no special functions.

---

## 6. What is move constructor? Why `noexcept`?

**Answer:**  
Transfers resources from temporary. `noexcept` allows containers to use move in strong exception guarantee.

```cpp
A(A&& other) noexcept : p(other.p) { other.p = nullptr; }
```

---

## 7. What is output of this move vs copy?

```cpp
vector<A> v;
v.push_back(A()); // moves if available
v.push_back(a);   // copies if a is lvalue
```

---

## 8. Can you default the move operations?

**Answer:**  
Yes: `A(A&&) = default;`

---

## 9. What is a virtual destructor? Why needed?

**Answer:**  
Ensures derived destructor called when deleting via base pointer.

```cpp
class Base { public: virtual ~Base() {} };
Base* p = new Derived();
delete p; // calls ~Derived then ~Base
```

---

## 10. What happens if base destructor is not virtual?

**Answer:**  
Only base destructor runs → resource leak, UB.

---

## 11. What is a pure virtual function? Abstract class?

**Answer:**  
`virtual void f() = 0;` Makes class abstract (cannot instantiate). Derived must override.

---

## 12. Can an abstract class have a constructor?

**Answer:**  
Yes — called when derived object constructed, but cannot create abstract object directly.

---

## 13. What is a vtable? When is it created?

**Answer:**  
Virtual table: array of function pointers per class. Created at compile-time, one per polymorphic class.

---

## 14. What is the vptr? When is it set?

**Answer:**  
Virtual pointer: hidden member pointing to vtable. Set during constructor (from base to derived).

---

## 15. Can you call virtual function from constructor?

**Answer:**  
Yes, but only base version is called (not overridden yet).

```cpp
class B { public: B() { f(); } virtual void f() { cout << "B"; } };
class D : public B { void f() override { cout << "D"; } };
D d; // outputs "B"
```

---

## 16. Can you call virtual function from destructor?

**Answer:**  
Yes, but only current class version (derived part already destroyed).

---

## 17. What is the output?

```cpp
class A { public: virtual void f() { cout << "A"; } };
class B : public A { void f() override { cout << "B"; } };
int main() {
    A* p = new B();
    p->f();
    delete p;
}
```

**Answer:**  
`B` (with UB if ~A not virtual). Add virtual destructor.

---

## 18. What is slicing? How to prevent?

**Answer:**  
Assigning derived to base copies only base part. Prevent using pointers/references.

```cpp
Derived d; Base b = d; // slices
Base& rb = d; // OK
```

---

## 19. What is object slicing with containers?

```cpp
vector<Base> v;
v.push_back(Derived()); // slices
```

**Fix:** `vector<unique_ptr<Base>>`

---

## 20. What is the output of this copy elision?

```cpp
A f() { A a; return a; }
int main() { A b = f(); }
```

**Answer:**  
Zero or one constructor call (copy elision / RVO).

---

## 21. When does copy elision NOT happen?

**Answer:**  
- Multiple return statements returning different named objects
- Compiler flag `-fno-elide-constructors`
- Returning function parameter

---

## 22. What is `std::move`? Does it move anything?

**Answer:**  
`std::move` casts to rvalue. Move constructor does actual moving.

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1); // s1 now empty
```

---

## 23. What is a moved-from state?

**Answer:**  
Valid but unspecified. Only safe to destroy or assign new value.

---

## 24. Can you use moved-from object?

**Answer:**  
Yes, but only reset it or destroy it.

```cpp
std::vector<int> v1{1,2,3};
std::vector<int> v2 = std::move(v1);
v1.clear(); // OK
v1.push_back(5); // OK
```

---

## 25. What is output?

```cpp
class A {
    int x;
public:
    A() : x(0) {}
    A(int v) : x(v) {}
    A(const A&) { cout << "copy "; }
    A(A&&) { cout << "move "; }
};
A f() { A a(10); return a; }
int main() { A b = f(); }
```

**Answer:**  
`(nothing)` — copy elision (RVO). With `-fno-elide-constructors`: `move`

---

## 26. What is the difference between `virtual` and `override`?

**Answer:**  
`virtual` declares function can be overridden. `override` checks correct overriding at compile-time.

```cpp
virtual void f();
void f() override; // checks base signature matches
```

---

## 27. What is `final` for virtual functions?

**Answer:**  
Prevents further overriding.

```cpp
virtual void f() final;
```

---

## 28. What is a covariant return type?

**Answer:**  
Overriding function returns pointer/reference to more derived type.

```cpp
class Base { virtual Base* clone(); };
class Derived : public Base { Derived* clone() override; };
```

---

## 29. Can you have a virtual copy constructor?

**Answer:**  
No — copyctors can't be virtual, but virtual `clone()` method works.

```cpp
virtual Base* clone() const { return new Base(*this); }
```

---

## 30. What is the output of this diamond inheritance?

```cpp
class A { public: A() { cout << "A"; } };
class B : virtual public A { public: B() { cout << "B"; } };
class C : virtual public A { public: C() { cout << "C"; } };
class D : public B, public C { public: D() { cout << "D"; } };
D d;
```

**Answer:**  
`A B C D` — virtual base initialized once by most derived.

---

## 31. What is the difference between `dynamic_cast` and `static_cast`?

**Answer:**  
`dynamic_cast` checks at runtime (needs RTTI, polymorphic base). `static_cast` compile-time, faster but unsafe.

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b); // OK
Derived* d2 = static_cast<Derived*>(b); // OK but unchecked
```

---

## 32. What happens if `dynamic_cast` fails for pointers vs references?

**Answer:**  
Pointer → `nullptr`. Reference → throws `std::bad_cast`.

---

## 33. What is the cost of `dynamic_cast`?

**Answer:**  
String comparison or traversal of inheritance tree — slower than `static_cast`.

---

## 34. What is the `typeid` operator?

**Answer:**  
Returns `std::type_info` for type. Requires RTTI for polymorphic types.

```cpp
if (typeid(*p) == typeid(Derived)) {}
```

---

## 35. What is the CRTP (Curiously Recurring Template Pattern)?

**Answer:**  
Static polymorphism (no vtable overhead).

```cpp
template<typename Derived> class Base {
    void f() { static_cast<Derived*>(this)->f(); }
};
class Derived : public Base<Derived> {};
```

---

## 36. Why is CRTP faster than virtual functions?

**Answer:**  
No vtable, no indirection, can inline.

---

## 37. What is the output of multiple inheritance same function name?

```cpp
class A { public: void f() { cout << "A"; } };
class B { public: void f() { cout << "B"; } };
class C : public A, public B {};
C c; c.f(); // error: ambiguous
```

**Fix:** `c.A::f();` or `c.B::f();`

---

## 38. What is a delegating constructor (C++11)?

**Answer:**  
Constructor calls another constructor in same class.

```cpp
A() : A(0) {}
A(int v) : x(v) {}
```

---

## 39. What is an explicit constructor? Why used?

**Answer:**  
Prevents implicit conversions.

```cpp
class A { explicit A(int) {} };
void f(A) {}
f(10); // error
f(A(10)); // OK
```

---

## 40. Can you have a private constructor? Copy constructor private?

**Answer:**  
Yes — singleton pattern, prevent copying.

```cpp
class Singleton {
    Singleton() {}
    Singleton(const Singleton&) = delete;
};
```

---

## 41. What is the output of this polymorphic array deletion?

```cpp
class Base { public: virtual ~Base() {} };
class Derived : public Base {};
Base* arr = new Derived[10];
delete[] arr; // UB
```

**Answer:**  
Undefined behavior — array delete needs exact type.

**Fix:** Use `vector<unique_ptr<Base>>`.

---

## 42. What is the difference between `delete` and `delete[]`?

**Answer:**  
`delete` calls one destructor; `delete[]` calls N destructors. Mixing is UB.

---

## 43. Can constructors be virtual?

**Answer:**  
No — object type unknown at construction time.

---

## 44. Can destructors be pure virtual?

**Answer:**  
Yes — but must have body.

```cpp
virtual ~A() = 0;
A::~A() {}
```

---

## 45. What is the order of member initialization in initializer list?

**Answer:**  
Order of declaration in class, NOT order in initializer list.

```cpp
class A { int x, y; public: A() : y(10), x(y) {} };
// x initialized first (garbage), then y=10
```

---

## 46. What is the output of `static` member initialization?

```cpp
class A { static int x; };
// without definition: linker error
int A::x = 0; // definition
```

---

## 47. Can you have a `const` member function modify `mutable` members?

**Answer:**  
Yes — `mutable` bypasses constness for caching, mutex.

```cpp
class A { mutable int cache; int get() const { cache++; return cache; } };
```

---

## 48. What is the difference between `struct` and `class` for inheritance?

**Answer:**  
`struct`: default public inheritance; `class`: default private.

---

## 49. What is the output of virtual inheritance with multiple paths?

```cpp
class A { public: int x; A() : x(10) {} };
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};
D d; cout << d.x; // works: x from single shared A
```

**Answer:**  
`10`

---

## 50. Final tricky: What is the output?

```cpp
class A {
public:
    virtual void f() final { cout << "A"; }
};
class B : public A {
    void f() override { cout << "B"; } // error: cannot override final
};
```

**Answer:**  
Compilation error — `final` function cannot be overridden.

---

Let me know if you want the **complete 500+ questions** across all 10 topics compiled into a master interview guide or searchable document.