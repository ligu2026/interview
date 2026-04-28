Here are **50 tricky C++ interview questions** focusing on **OOP & C++ Class Fundamentals** — covering inheritance, polymorphism, encapsulation, abstraction, access specifiers, method overriding, hiding, slicing, and more — with answers and code examples.

---

## 1. What is the difference between compile-time polymorphism and runtime polymorphism?

**Answer:**  
Compile-time: function overloading, templates. Runtime: virtual functions.

```cpp
class Base { public: virtual void f() { cout << "Base"; } };
class Derived : public Base { void f() override { cout << "Derived"; } };
Base* p = new Derived();
p->f(); // runtime: Derived
```

---

## 2. What is function overriding? How does it differ from overloading?

**Answer:**  
Overriding: redefining base virtual function in derived class. Overloading: same name, different parameters.

---

## 3. What is the output?

```cpp
class A { public: void f() { cout << "A"; } };
class B : public A { public: void f() { cout << "B"; } };
int main() {
    A* p = new B();
    p->f();
}
```

**Answer:**  
`A` — non-virtual, resolved at compile-time.

---

## 4. How to make overriding work in above?

**Answer:**  
Make `f()` virtual in base.

```cpp
class A { public: virtual void f() { cout << "A"; } };
```

---

## 5. What is a virtual destructor? Why needed?

**Answer:**  
Ensures derived destructor called when deleting base pointer.

```cpp
class Base { public: virtual ~Base() {} };
```

---

## 6. What is a pure virtual function? Abstract class?

**Answer:**  
`virtual void f() = 0;` makes class abstract (cannot instantiate).

---

## 7. Can an abstract class have a constructor?

**Answer:**  
Yes — called when derived object constructed.

---

## 8. What is the output?

```cpp
class Base { public: Base() { f(); } virtual void f() { cout << "B"; } };
class Derived : public Base { void f() override { cout << "D"; } };
Derived d;
```

**Answer:**  
`B` — virtual call in constructor goes to base version.

---

## 9. What is the slicing problem?

**Answer:**  
Assigning derived object to base object copies only base part.

```cpp
Derived d; Base b = d; // slices
```

**Fix:** Use pointers/references.

---

## 10. What is object slicing with containers?

**Answer:**
```cpp
vector<Base> v;
v.push_back(Derived()); // slices
```

**Fix:** Use `vector<unique_ptr<Base>>`.

---

## 11. What is the diamond problem in multiple inheritance?

**Answer:**  
A class inherits same base class twice.

```cpp
class A {};
class B : public A {};
class C : public A {};
class D : public B, public C {};
```

---

## 12. How to solve diamond problem?

**Answer:**  
Virtual inheritance.

```cpp
class B : virtual public A {};
class C : virtual public A {};
```

---

## 13. What is the order of constructor calls in multiple inheritance?

**Answer:**  
Base classes in declaration order, then members, then derived.

---

## 14. What is the order of destructor calls?

**Answer:**  
Reverse of constructor order.

---

## 15. Can you have private virtual functions?

**Answer:**  
Yes — useful for Template Method pattern.

```cpp
class Base { virtual void f() = 0; public: void call() { f(); } };
```

---

## 16. What is a `final` class?

**Answer:**  
`class A final {};` — cannot be inherited.

---

## 17. What is a `final` virtual function?

**Answer:**  
`virtual void f() final;` — cannot be overridden further.

---

## 18. What is function hiding? Example.

**Answer:**  
Derived class function hides base version even if signatures differ.

```cpp
class Base { public: void f(int); };
class Derived : public Base { public: void f(); };
Derived d; d.f(10); // error: hidden
```

**Fix:** `using Base::f;`

---

## 19. What is the difference between `public`, `protected`, `private` inheritance?

**Answer:**  
- `public`: public → public, protected → protected  
- `protected`: public → protected  
- `private`: everything → private  

---

## 20. Can you access base private members in derived?

**Answer:**  
No — private always inaccessible. Use `protected`.

---

## 21. What is a friend class? Friend function?

**Answer:**  
Non-member function or class accessing private members.

```cpp
class A { friend class B; };
```

---

## 22. Is friendship inherited?

**Answer:**  
No.

---

## 23. Is friendship transitive?

**Answer:**  
No.

---

## 24. What is the output?

```cpp
class Base { public: static void f() { cout << "B"; } };
class Derived : public Base { public: static void f() { cout << "D"; } };
Base* p = new Derived();
p->f();
```

**Answer:**  
`B` — static functions not virtual.

---

## 25. What is a covariant return type?

**Answer:**  
Overriding function returns pointer/reference to derived type.

```cpp
class Base { virtual Base* clone() { return new Base(); } };
class Derived : public Base { Derived* clone() override { return new Derived(); } };
```

---

## 26. What is the output of this evil code?

```cpp
class B { public: virtual void f() { cout << "B"; } };
class D : public B { public: void f() override { cout << "D"; } };
int main() {
    B* p = new D();
    p->f();
    delete p;
}
```

**Answer:**  
`D` (and leak if ~B non-virtual). Add virtual destructor.

---

## 27. Can a constructor be `private`? Why?

**Answer:**  
Yes — singleton, factory, builder patterns.

---

## 28. Can a destructor be `private`?

**Answer:**  
Yes — prevents stack allocation, forces heap + `friend` or static method for deletion.

---

## 29. What is the `CRTP` (Curiously Recurring Template Pattern)?

**Answer:**  
Static polymorphism using templates.

```cpp
template<typename Derived> class Base { void f() { static_cast<Derived*>(this)->f(); } };
class Derived : public Base<Derived> {};
```

---

## 30. What is the output of virtual inheritance construction?

```cpp
class A { public: A(int) { cout << "A"; } };
class B : virtual public A { public: B() : A(1) { cout << "B"; } };
class C : virtual public A { public: C() : A(2) { cout << "C"; } };
class D : public B, public C { public: D() : A(3) { cout << "D"; } };
D d;
```

**Answer:**  
`A3 B C D` — virtual base initialized by most derived class.

---

## 31. What is the "most derived" class rule in virtual inheritance?

**Answer:**  
Most derived class initializes virtual base; others' initializers ignored.

---

## 32. What is a `delegating constructor`?

**Answer:**  
Calls another constructor in same class.

```cpp
A() : A(0) {}
```

---

## 33. What is an `explicit` constructor? Prevent what?

**Answer:**  
Prevents implicit conversions.

```cpp
class A { explicit A(int) {} };
void f(A) {}
f(10); // error
```

---

## 34. What is `override` specifier?

**Answer:**  
Tells compiler function overrides base virtual. Catches mismatches.

---

## 35. What is `final` specifier for virtual functions?

**Answer:**  
`virtual void f() final;` — prevents further overriding.

---

## 36. What is the difference between `struct` and `class`?

**Answer:**  
`struct` members public by default; `class` private.

---

## 37. Can `struct` inherit from `class`? Vice versa?

**Answer:**  
Yes. Default access: struct → public, class → private.

---

## 38. What is the Law of Demeter (principle of least knowledge)?

**Answer:**  
Object should only call methods of itself, its members, arguments, or newly created objects.

---

## 39. What is a virtual base class vtable cost?

**Answer:**  
Each virtual base adds pointer overhead; virtual inheritance slower.

---

## 40. Can you have a virtual friend function?

**Answer:**  
No — `friend` not member, can't be virtual.

---

## 41. What is an interface in C++?

**Answer:**  
Abstract class with only pure virtual functions.

```cpp
class IShape { public: virtual double area() = 0; virtual ~IShape() {} };
```

---

## 42. What is the difference between early binding and late binding?

**Answer:**  
Early: resolved at compile-time (non-virtual). Late: virtual table at runtime.

---

## 43. What is the output?

```cpp
class B { public: virtual ~B() { cout << "B"; } };
class D : public B { public: ~D() { cout << "D"; } };
B* p = new D(); delete p;
```

**Answer:**  
`DB` — derived then base.

---

## 44. Can static functions be virtual?

**Answer:**  
No — virtual depends on `this` pointer.

---

## 45. What is name mangling? Why needed for overloading?

**Answer:**  
Compiler encodes function name + parameters to differentiate overloads.

---

## 46. What is the difference between `dynamic_cast` and `static_cast` with polymorphic types?

**Answer:**  
`dynamic_cast` checks at runtime (requires RTTI); `static_cast` compile-time.

```cpp
Base* b = new Derived();
Derived* d = dynamic_cast<Derived*>(b); // OK
Derived* d2 = static_cast<Derived*>(b); // OK but unsafe if wrong
```

---

## 47. What is `typeid`? When does it give wrong answer?

**Answer:**  
`typeid(*ptr)` with non-polymorphic base returns base type even if derived object.

---

## 48. What is a `virtual` table (vtable)?

**Answer:**  
Per-class array of function pointers for dynamic dispatch.

---

## 49. Can you have a virtual constructor?

**Answer:**  
No — object type unknown at construction time.

---

## 50. Output of this polymorphic array trap?

```cpp
class B { public: int x; virtual ~B() {} };
class D : public B { public: int y; };
void f(B* arr, int n) { for (int i = 0; i < n; i++) arr[i].x = 1; }
int main() {
    D d[10];
    f(d, 10);
}
```

**Answer:**  
Undefined behavior — pointer arithmetic on array of derived treats as base size.

---

Let me know if you’d like the full set of **250+ questions** (all 5 parts) compiled into a master document or a quiz generator script.:wq
