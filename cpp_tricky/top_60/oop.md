Here is a curated list of **60 tricky C++ interview questions** focused on **OOP, Class Fundamentals, Constructor/Destructor, Copy/Move semantics, Polymorphism, vtable, and Abstract Classes**. Each includes a concise answer and a code example where relevant.

---

## **Constructors & Destructors**

### 1. Can a constructor be virtual?
**No.** Virtual mechanism relies on vtable, which is set up by the constructor. A virtual call inside a constructor uses the current class’s version, not the derived one.

### 2. Can a destructor be virtual? Why?
**Yes.** To ensure derived class destructor runs when deleting base pointer.
```cpp
Base* p = new Derived();
delete p; // if ~Base() not virtual, ~Derived() not called → leak
```

### 3. What is a pure virtual destructor? Must it have a body?
**Yes, it must have a body** (linker needs it for base destructor call).
```cpp
class Base { public: virtual ~Base() = 0; };
Base::~Base() {} // required
```

### 4. What order are constructors/destructors called?
**Construction:** Base → Derived. **Destruction:** Derived → Base.

### 5. What happens if an exception escapes a constructor?
The object is considered **not fully constructed**; destructor will **not** be called. Members constructed so far are destroyed automatically.

### 6. What happens if an exception escapes a destructor?
`std::terminate` is called (if the stack is already unwinding due to another exception). **Never throw from destructors.**

### 7. Can you overload a destructor?
**No.** Destructor has no parameters, only one per class.

### 8. Can a constructor be private? When useful?
**Yes.** For singleton, factory methods, or builder patterns.

### 9. What is a delegating constructor? (C++11)
One constructor calls another from the same class.
```cpp
class A {
public:
    A() : A(0) {}
    A(int x) : val(x) {}
};
```

### 10. If you define a copy constructor, must you define a destructor?
Not mandatory, but often **Rule of Three/Five** applies: manage resources.

---

## **Copy Constructor & Copy Assignment**

### 11. When is copy constructor called vs copy assignment?
- **Copy ctor:** initialization (`A b = a;` `A b(a);` passing by value).
- **Copy assignment:** existing object `b = a;`

### 12. Why must copy assignment return `A&`?
To allow chaining: `a = b = c;`

### 13. How to disable copying in C++03 vs C++11?
**03:** private, unimplemented copy ctor/assignment.  
**11+:** `A(const A&) = delete; A& operator=(const A&) = delete;`

### 14. What is the copy-and-swap idiom?
Provides strong exception safety and avoids duplication.
```cpp
A& operator=(A other) { swap(*this, other); return *this; }
```

### 15. Can copy constructor be templated?
Yes, but it’s not a **copy constructor** in the language’s eyes (the non-template one is still needed).

### 16. What happens if copy constructor takes value instead of const ref?
**Infinite recursion** – calling copy constructor to pass argument.

### 17. Is the compiler-generated copy constructor shallow or deep?
**Shallow** (memberwise copy). Deep copy must be user-defined.

### 18. Does `memcpy` work on non-trivial copyable objects?
**No.** Undefined behavior (violates lifetime rules, vptr issues).

### 19. When is copy constructor elided? (Copy Elision / RVO)
When returning a local variable or initializing from temporary. Guaranteed elision in C++17 for prvalues.

### 20. What’s the difference between `A a = b` and `A a(b)`?
No difference for copy construction. First form requires non-explicit constructor.

---

## **Move Semantics (C++11 and later)**

### 21. What’s the difference between move constructor and move assignment?
Move ctor creates new object from rvalue; move assignment replaces existing object’s state.

### 22. When is move constructor called?
From rvalues: `A a = std::move(b);` `A a(return_local());` `return local_var;`

### 23. Why mark move constructor `noexcept`?
Allows standard containers (vector) to use move instead of copy during reallocation (strong exception safety).

### 24. What’s the difference between `std::move` and `std::forward`?
`std::move` unconditionally casts to rvalue. `std::forward` preserves value category (perfect forwarding).

### 25. If a class has move constructor, is copy constructor deleted?
**No.** You must `= default` copy if needed. Move declared = compiler doesn’t generate copy.

### 26. Can you have both copy and move constructor?
Yes.

### 27. What is a stealing move constructor?
Transferring ownership of resources (e.g., pointer) and leaving source in valid but unspecified state.

### 28. Is `std::vector<A>` efficient if `A` has no move ctor?
Falls back to copy if no move and no noexcept copy, but slower.

### 29. Can you `=default` a move constructor in a class with raw pointer?
It will shallow copy pointer (dangerous). You must define custom move.

### 30. How to force move instead of copy?
`A a(std::move(b));`

---

## **Polymorphism & Virtual Functions**

### 31. What’s the difference between static and dynamic polymorphism?
Static: templates, overloads (compile-time). Dynamic: virtual functions (runtime).

### 32. What is a vtable?
Per-class table of function pointers for virtual functions. Each object has a hidden vptr pointing to its class’s vtable.

### 33. When is the vptr set?
During **constructor** – after base class construction completes, before constructor body runs for that level.

### 34. Can you call a virtual function from constructor safely?
**Call is resolved to current class (not derived).** No polymorphic behavior.

### 35. What is a pure virtual function? Effect on class?
`virtual void f() = 0;` Makes class abstract. Cannot instantiate.

### 36. Can an abstract class have a constructor?
Yes. It initializes base part of derived objects.

### 37. What’s a virtual final function? (C++11)
`virtual void f() final;` Prevents overriding in derived classes.

### 38. What happens if you override a non-virtual function?
Hides base function (not polymorphic). Call through base pointer uses base version.

### 39. Can static functions be virtual?
**No.** Static functions are not bound to objects (no `this`, no vptr).

### 40. What is the cost of virtual functions?
Extra memory (vptr per object, vtable per class) and indirect call overhead (branch prediction may fail).

---

## **Inheritance & Object Slicing**

### 41. What is object slicing?
Assigning derived object to base object by value – only base part copied.
```cpp
Derived d; Base b = d; // sliced
```

### 42. How to prevent slicing?
Use pointers or references. Delete base copy operations (if polymorphic).

### 43. What is the diamond problem? How resolved in C++?
Multiple inheritance from two classes sharing a common base. Resolved by **virtual inheritance** (`class B : virtual public A`).

### 44. What’s the order of construction with virtual inheritance?
Most virtual base classes first (deepest leftmost first), then normal bases, then derived.

### 45. Can you downcast safely without RTTI?
Only if you know the type – `static_cast` is unsafe. `dynamic_cast` checks (requires at least one virtual function).

### 46. What is `dynamic_cast` vs `static_cast`?
`dynamic_cast` runtime-checked (for polymorphic types). `static_cast` compile-time, faster but dangerous.

### 47. What is `typeid`? When can it be used?
Returns `std::type_info`. Works on polymorphic types with dynamic type info; else static type.

### 48. Can you have private virtual functions?
Yes – overriding allowed in derived, but base class controls call (Template Method pattern).

### 49. What is the “non-virtual interface” idiom (NVI)?
Public non-virtual function calls private virtual function (template method pattern).

### 50. Can a derived class redefine a virtual function’s default argument?
**Yes, but dangerous** – default arguments are statically bound. Use base’s defaults.

---

## **OOP & Class Design Tricky Questions**

### 51. What is an “abstract base class”?
Class with at least one pure virtual function.

### 52. Can a pure virtual function have a definition?
Yes. Can provide common implementation – derived must still override or remain abstract.

### 53. How to make a class non-inheritable in C++11?
`final` keyword: `class A final {};`

### 54. C++03 trick to make class non-inheritable?
Private constructor + static factory, or template + private destructor trick.

### 55. What is “empty base optimization” (EBO)?
Compiler can give size 0 to a base class with no non-static data members (no vptr). Used in `std::vector` for allocators.

### 56. Can a class have multiple vptrs?
Yes – if inherits from multiple classes with virtual functions.

### 57. What’s an “expensive” hidden cost of virtual inheritance?
Each virtual base access adds an extra indirection (offset in vtable).

### 58. What is the “curiously recurring template pattern” (CRTP)?
Base template taking derived as parameter – static polymorphism.
```cpp
template <typename D> class Base { void f() { static_cast<D*>(this)->impl(); } };
```

### 59. How to define a “final” virtual function?
`virtual void f() final;`

### 60. What’s wrong with calling `delete this`?
Valid **only** if object allocated with `new` and you ensure no member/destructor accesses `this` after. Often error-prone.

---

Let me know if you’d like a deep-dive explanation (with full runnable examples) for any of the top 10 hardest ones.