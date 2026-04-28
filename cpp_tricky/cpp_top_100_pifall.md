# Top 100 C++ Interview Pitfalls (with Answers & Examples)
This list covers **100 high-frequency C++ interview pitfalls** across memory management, OOP, templates, STL, concurrency, and modern C++ (C++11/14/17/20). Each entry includes a **problem code**, **root cause**, **correct fix**, and **key takeaway**—ideal for interview prep.

---

## 1. Memory Management (1–20)
### 1. `delete` vs `delete[]` Mismatch
**Problem**:
```cpp
int* arr = new int[10];
delete arr; // ❌
```
**Issue**: `delete` calls 1 destructor; `delete[]` calls N destructors. Mismatch → **memory leak + undefined behavior (UB)**.
**Fix**: `delete[] arr;`
**Takeaway**: Always pair `new[]` with `delete[]`.

### 2. Double Free / Use-After-Free
**Problem**:
```cpp
int* p = new int(42);
delete p;
delete p; // ❌ double free
```
**Issue**: Deleting a dangling pointer → UB (crash/corruption).
**Fix**: Set `p = nullptr;` after `delete`; use smart pointers.
**Takeaway**: Never delete a pointer twice; avoid raw pointers.

### 3. Raw Pointer → Multiple `shared_ptr`
**Problem**:
```cpp
int* p = new int(10);
std::shared_ptr<int> sp1(p);
std::shared_ptr<int> sp2(p); // ❌ two control blocks
```
**Issue**: Each `shared_ptr` creates an independent control block → **double free**.
**Fix**: Use `std::make_shared` or share ownership:
```cpp
auto sp1 = std::make_shared<int>(10);
auto sp2 = sp1; // ✅ same control block
```
**Takeaway**: Never initialize multiple `shared_ptr` from the same raw pointer.

### 4. `shared_ptr` Thread Safety Misconception
**Problem**:
```cpp
std::shared_ptr<int> sp = std::make_shared<int>(42);
// Thread 1: sp = std::make_shared<int>(100);
// Thread 2: sp = std::make_shared<int>(200); // ❌ race condition
```
**Issue**: `shared_ptr`’s **reference count is atomic**, but **assignment/reset of the `shared_ptr` object itself is not thread-safe**.
**Fix**: Protect with a mutex:
```cpp
std::mutex mtx;
{ std::lock_guard<std::mutex> lock(mtx); sp = ...; }
```
**Takeaway**: `shared_ptr` is not fully thread-safe—only refcount is atomic.

### 5. `weak_ptr` Lock Failure
**Problem**:
```cpp
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp(sp);
sp.reset();
auto p = wp.lock(); // p is null
*p = 100; // ❌ dereference null
```
**Issue**: `lock()` returns null if the object is destroyed.
**Fix**: Check validity:
```cpp
if (auto p = wp.lock()) { *p = 100; }
```
**Takeaway**: Always check `lock()` result before using `weak_ptr`.

### 6. `unique_ptr` Copy Attempt
**Problem**:
```cpp
std::unique_ptr<int> up1 = std::make_unique<int>(42);
std::unique_ptr<int> up2 = up1; // ❌ copy constructor deleted
```
**Issue**: `unique_ptr` is **move-only** (no copy semantics).
**Fix**: Move ownership:
```cpp
std::unique_ptr<int> up2 = std::move(up1);
```
**Takeaway**: `unique_ptr` enforces exclusive ownership—use `std::move`.

### 7. `auto_ptr` Deprecation & Ownership Transfer
**Problem**:
```cpp
std::auto_ptr<int> ap1(new int(42));
std::auto_ptr<int> ap2 = ap1; // ap1 becomes null
*ap1 = 100; // ❌ access violation
```
**Issue**: `auto_ptr` is deprecated (C++11) and transfers ownership on copy → UB.
**Fix**: Replace with `unique_ptr`/`shared_ptr`.
**Takeaway**: Never use `auto_ptr` in modern C++.

### 8. Memory Leak in Constructor Failure
**Problem**:
```cpp
class A {
    int* p;
public:
    A() : p(new int(42)) { throw std::runtime_error("oops"); }
    ~A() { delete p; }
};
A a; // ❌ leak: p is allocated but destructor never runs
```
**Issue**: If a constructor throws, the object is not fully constructed → **destructor not called** → leak.
**Fix**: Use RAII (smart pointers):
```cpp
class A { std::unique_ptr<int> p; public: A() : p(std::make_unique<int>(42)) {} };
```
**Takeaway**: Always use RAII for resources in constructors.

### 9. `new`/`malloc` Mixing
**Problem**:
```cpp
int* p = (int*)malloc(sizeof(int));
delete p; // ❌ UB
int* q = new int(42);
free(q); // ❌ UB
```
**Issue**: `new`/`delete` call constructors/destructors; `malloc`/`free` do not. Mixing → UB.
**Fix**: Pair `new` ↔ `delete`, `malloc` ↔ `free`.
**Takeaway**: Never mix allocation/deallocation functions.

### 10. Null Pointer Dereference
**Problem**:
```cpp
int* p = nullptr;
*p = 42; // ❌ crash
```
**Issue**: Dereferencing `nullptr` → UB.
**Fix**: Check for null before use:
```cpp
if (p) { *p = 42; }
```
**Takeaway**: Validate pointers before dereferencing.

### 11. Stack Overflow (Large Local Arrays)
**Problem**:
```cpp
void func() { int arr[1000000]; } // ❌ stack overflow
```
**Issue**: Stack size is limited (typically MBs); large arrays → overflow.
**Fix**: Use heap (`new[]`/`vector`) or static storage.
**Takeaway**: Avoid large objects on the stack.

### 12. Uninitialized Pointer
**Problem**:
```cpp
int* p; // wild pointer
*p = 42; // ❌ UB
```
**Issue**: Uninitialized pointer points to random memory → UB.
**Fix**: Initialize to `nullptr` or a valid address.
**Takeaway**: Always initialize pointers.

### 13. Memory Leak in `std::exception`
**Problem**:
```cpp
void func() {
    int* p = new int(42);
    throw std::runtime_error("oops");
    delete p; // unreachable → leak
}
```
**Issue**: Exception skips cleanup code → leak.
**Fix**: RAII (smart pointers/`vector`).
**Takeaway**: Use RAII to guarantee cleanup during exceptions.

### 14. `shared_ptr::use_count()` for Logic
**Problem**:
```cpp
auto sp = std::make_shared<int>(42);
if (sp.use_count() == 1) { /* critical code */ } // ❌ unsafe in multi-thread
```
**Issue**: `use_count()` is approximate (not atomic in all cases) → race conditions.
**Fix**: Use `weak_ptr::lock()` or design to avoid counting.
**Takeaway**: Never use `use_count()` for business logic.

### 15. `unique_ptr` to Array
**Problem**:
```cpp
std::unique_ptr<int> up(new int[10]); // ❌ uses delete, not delete[]
```
**Issue**: Default `unique_ptr` deleter uses `delete` → leak for arrays.
**Fix**: Use array specialization:
```cpp
std::unique_ptr<int[]> up(new int[10]); // ✅ delete[]
```
**Takeaway**: Use `unique_ptr<T[]>` for dynamic arrays.

### 16. Dangling Reference to Local Variable
**Problem**:
```cpp
int& func() { int x = 42; return x; } // ❌ x is destroyed
int& ref = func();
ref = 100; // UB
```
**Issue**: Returning a reference to a local (stack) variable → dangling reference.
**Fix**: Return by value, or use static/heap allocation.
**Takeaway**: Never return references/pointers to local variables.

### 17. `malloc` for Non-POD Types
**Problem**:
```cpp
std::string* s = (std::string*)malloc(sizeof(std::string));
// ❌ constructor not called → invalid object
```
**Issue**: `malloc` only allocates memory; does not call constructors → invalid objects.
**Fix**: Use `new` (calls constructor) or placement `new`.
**Takeaway**: Use `new` for non-POD types; `malloc` only for POD.

### 18. Memory Leak in `std::vector` Resize
**Problem**:
```cpp
std::vector<int*> vec;
vec.push_back(new int(42));
vec.clear(); // ❌ pointers destroyed, but memory not freed
```
**Issue**: `vector` clears elements but does not delete pointed-to memory → leak.
**Fix**: Use `vector<unique_ptr<int>>`.
**Takeaway**: Store smart pointers in containers to avoid leaks.

### 19. `delete` on Stack Variable
**Problem**:
```cpp
int x = 42;
int* p = &x;
delete p; // ❌ UB (delete stack memory)
```
**Issue**: `delete` only for heap-allocated memory → UB.
**Fix**: Never delete stack pointers.
**Takeaway**: Only `delete` pointers from `new`/`malloc`.

### 20. Unfreed Memory in `main` Exit
**Problem**:
```cpp
int main() { int* p = new int(42); return 0; } // ❌ leak
```
**Issue**: OS reclaims memory on exit, but it’s a **logical leak** (bad practice).
**Fix**: Use smart pointers or `delete`.
**Takeaway**: Always clean up resources—even in `main`.

---

## 2. Object-Oriented Programming (21–40)
### 21. Non-Virtual Destructor in Base Class
**Problem**:
```cpp
class Base { public: ~Base() {} }; // ❌ non-virtual
class Derived : public Base { int* p; public: Derived() : p(new int) {} ~Derived() { delete p; } };
Base* b = new Derived();
delete b; // ❌ Derived destructor not called → leak
```
**Issue**: Deleting a derived object via base pointer requires a **virtual destructor**.
**Fix**: Make base destructor virtual:
```cpp
class Base { public: virtual ~Base() = default; };
```
**Takeaway**: Base classes with inheritance need virtual destructors.

### 22. Virtual Function in Constructor/Destructor
**Problem**:
```cpp
class Base {
public:
    Base() { func(); } // ❌ calls Base::func(), not Derived
    virtual void func() { std::cout << "Base\n"; }
};
class Derived : public Base {
public:
    void func() override { std::cout << "Derived\n"; }
};
Derived d; // prints "Base"
```
**Issue**: During base construction, the object is still a `Base`—**no dynamic dispatch**.
**Fix**: Avoid virtual calls in ctors/dtors; use factory functions.
**Takeaway**: Virtual functions in ctors/dtors do not work as expected.

### 23. Object Slicing
**Problem**:
```cpp
class Base { public: virtual void print() { std::cout << "Base\n"; } };
class Derived : public Base { int x; public: void print() override { std::cout << "Derived\n"; } };
Derived d;
Base b = d; // ❌ slicing: Derived parts are lost
b.print(); // calls Base::print()
```
**Issue**: Assigning a derived object to a base object **slices off derived members**.
**Fix**: Use pointers/references:
```cpp
Base& b = d; b.print(); // ✅ Derived::print()
```
**Takeaway**: Use pointers/references for polymorphic behavior.

### 24. Missing `override` Keyword
**Problem**:
```cpp
class Base { public: virtual void func(int) {} };
class Derived : public Base {
public:
    void func(double) {} // ❌ not overriding—new function
};
```
**Issue**: Signature mismatch → no override; silent bug.
**Fix**: Use `override` (C++11):
```cpp
void func(int) override {} // ✅ compiler error if mismatch
```
**Takeaway**: Always use `override` for virtual functions.

### 25. `final` Keyword Misuse
**Problem**:
```cpp
class Base final {};
class Derived : public Base {}; // ❌ compiler error
```
**Issue**: `final` prevents inheritance.
**Fix**: Remove `final` or avoid inheriting.
**Takeaway**: `final` blocks inheritance/overriding—use intentionally.

### 26. Copy Constructor/Assignment Operator Missing
**Problem**:
```cpp
class A { int* p; public: A() : p(new int(42)) {} ~A() { delete p; } };
A a1;
A a2 = a1; // ❌ shallow copy → double free
```
**Issue**: Default copy/assignment do **shallow copies** → duplicate pointers → double free.
**Fix**: Implement deep copy or disable copy:
```cpp
A(const A& other) : p(new int(*other.p)) {}
A& operator=(const A& other) { if (this != &other) { delete p; p = new int(*other.p); } return *this; }
// Or disable: A(const A&) = delete; A& operator=(const A&) = delete;
```
**Takeaway**: Follow the **Rule of Three/Five/Zero** for resource-owning classes.

### 27. Move Constructor/Assignment Missing
**Problem**:
```cpp
class A { int* p; public: A() : p(new int(42)) {} ~A() { delete p; } };
A a1;
A a2 = std::move(a1); // ❌ uses copy constructor → double free
```
**Issue**: No move semantics → copy is used for rvalues.
**Fix**: Implement move operations (Rule of Five):
```cpp
A(A&& other) noexcept : p(other.p) { other.p = nullptr; }
A& operator=(A&& other) noexcept { if (this != &other) { delete p; p = other.p; other.p = nullptr; } return *this; }
```
**Takeaway**: Implement move operations for performance and correctness.

### 28. Rule of Zero Violation
**Problem**:
```cpp
class A { std::string s; std::vector<int> v; public: ~A() {} }; // ❌ unnecessary dtor
```
**Issue**: Manual dtor disables default move operations → performance hit.
**Fix**: Follow Rule of Zero: let compiler generate special members.
**Takeaway**: Avoid manual special members if using RAII members.

### 29. Diamond Inheritance (No Virtual Inheritance)
**Problem**:
```cpp
class A { public: int x; };
class B : public A {};
class C : public A {};
class D : public B, public C {}; // ❌ two A subobjects
D d; d.x; // ambiguous
```
**Issue**: Non-virtual inheritance → duplicate base subobjects.
**Fix**: Virtual inheritance:
```cpp
class B : virtual public A {}; class C : virtual public A {};
```
**Takeaway**: Use virtual inheritance for diamond hierarchies.

### 30. `this` Pointer in Lambda Capture
**Problem**:
```cpp
class A {
public:
    void func() {
        auto lam = [this]() { std::cout << x; };
        std::thread t(lam);
    } // A destroyed before thread runs → UB
private:
    int x = 42;
};
```
**Issue**: Lambda captures `this`; if `A` is destroyed → dangling `this`.
**Fix**: Capture by value or use `shared_from_this`.
**Takeaway**: Be cautious with `this` in long-lived lambdas.

### 31. Static Member Initialization Order
**Problem**:
```cpp
// a.cpp
int x = y;
// b.cpp
int y = 42;
```
**Issue**: **Static initialization order fiasco**—`x` uses `y` before it’s initialized.
**Fix**: Use function-local statics (Meyers’ singleton):
```cpp
int& get_x() { static int x = get_y(); return x; }
int& get_y() { static int y = 42; return y; }
```
**Takeaway**: Avoid cross-file static dependencies.

### 32. Non-Const Member Function on Const Object
**Problem**:
```cpp
class A { public: void func() {} };
const A a;
a.func(); // ❌ compiler error
```
**Issue**: Const objects can only call const member functions.
**Fix**: Mark `func()` as `const`:
```cpp
void func() const {}
```
**Takeaway**: Mark non-modifying functions as `const`.

### 33. `mutable` Keyword Misuse
**Problem**:
```cpp
class A {
    int x;
public:
    void func() const { x = 42; } // ❌ error
};
```
**Issue**: Const functions cannot modify non-`mutable` members.
**Fix**: Use `mutable` for members that need modification in const functions:
```cpp
mutable int x;
```
**Takeaway**: Use `mutable` sparingly for logical constness.

### 34. Default Arguments in Virtual Functions
**Problem**:
```cpp
class Base { public: virtual void func(int x = 0) {} };
class Derived : public Base { public: void func(int x = 1) override {} };
Base* b = new Derived();
b->func(); // uses Base's default (0), not Derived's (1)
```
**Issue**: Default arguments are resolved **statically** (based on pointer type), not dynamically.
**Fix**: Avoid default args in virtual functions; use overloads.
**Takeaway**: Default arguments in virtual functions are dangerous.

### 35. Pure Virtual Function Definition
**Problem**:
```cpp
class Base { public: virtual void func() = 0; };
void Base::func() {} // allowed but often misused
class Derived : public Base {}; // ❌ no override → abstract
Derived d; // error
```
**Issue**: Pure virtual functions make a class abstract; derived classes must override.
**Fix**: Override in `Derived` or remove `= 0`.
**Takeaway**: Pure virtual functions enforce interface implementation.

### 36. `explicit` Keyword Missing
**Problem**:
```cpp
class A { public: A(int x) {} };
void func(A a) {}
func(42); // ❌ implicit conversion from int to A
```
**Issue**: Implicit conversions lead to unexpected behavior.
**Fix**: Use `explicit`:
```cpp
explicit A(int x) {}
```
**Takeaway**: Mark single-argument constructors `explicit` to prevent implicit conversions.

### 37. Copy Elision Misunderstanding
**Problem**:
```cpp
A func() { A a; return a; }
A a = func(); // copy constructor not called (copy elision)
```
**Issue**: Compiler optimizes away copies (NRVO/RVO)—do not rely on side effects in copy ctors.
**Fix**: Avoid side effects in copy/move constructors.
**Takeaway**: Copy elision is allowed (C++17 mandatory for prvalues).

### 38. `friend` Keyword Overuse
**Problem**:
```cpp
class A { friend class B; }; // B accesses all A's private members
```
**Issue**: Overuse breaks encapsulation.
**Fix**: Minimize `friend`; use public interfaces or friend functions.
**Takeaway**: `friend` is a tool, not a default—use sparingly.

### 39. Inheritance Access Specifiers
**Problem**:
```cpp
class Base { public: int x; };
class Derived : private Base {};
Derived d; d.x; // ❌ error: Base is private
```
**Issue**: Private inheritance hides base members from outside.
**Fix**: Use public inheritance for interface:
```cpp
class Derived : public Base {};
```
**Takeaway**: Use public inheritance for "is-a" relationships.

### 40. `sizeof` for Polymorphic Objects
**Problem**:
```cpp
Base* b = new Derived();
std::cout << sizeof(*b); // prints sizeof(Base), not Derived
```
**Issue**: `sizeof` is resolved **statically**—does not account for dynamic type.
**Fix**: Use a virtual function to get size, or RTTI (`typeid`).
**Takeaway**: `sizeof` is static; use RTTI for dynamic type info.

---

## 3. Templates & Generic Programming (41–60)
### 41. Perfect Forwarding Failure
**Problem**:
```cpp
template <typename T>
void log(T&& param) { std::cout << std::forward<T>(param); }
log(std::vector<int>{}); // ❌ compile error
```
**Issue**: `std::vector`’s `operator<<` does not accept rvalues directly.
**Fix**: Use `std::move` or wrap in a forwarding helper:
```cpp
log(std::move(std::vector<int>{}));
```
**Takeaway**: Perfect forwarding requires care with rvalue-only operations.

### 42. `typename` vs `class` in Templates
**Problem**:
```cpp
template <template <typename> class Container> // OK
template <template <typename> typename Container> // OK (C++17)
```
**Issue**: No functional difference—`typename` is preferred for types.
**Takeaway**: Use `typename` for template parameters (C++17+).

### 43. Dependent Name Lookup
**Problem**:
```cpp
template <typename T>
class A {
    typename T::type x; // ❌ error without typename
};
```
**Issue**: `T::type` is a **dependent name**—compiler needs `typename` to know it’s a type.
**Fix**: Add `typename`:
```cpp
typename T::type x;
```
**Takeaway**: Use `typename` for dependent type names.

### 44. Template Specialization Order
**Problem**:
```cpp
template <typename T> void func(T) { std::cout << "generic\n"; }
template <> void func(int*) { std::cout << "int*\n"; }
template <typename T> void func(T*) { std::cout << "T*\n"; }
func((int*)nullptr); // ❌ ambiguous
```
**Issue**: Specialization order matters—more specific specializations come first.
**Fix**: Reorder:
```cpp
template <typename T> void func(T*) { ... }
template <> void func(int*) { ... }
```
**Takeaway**: Order specializations from most to least specific.

### 45. `constexpr` Function Limitations
**Problem**:
```cpp
constexpr int func(int x) { if (x > 0) return x; else return -x; } // OK (C++14)
constexpr int func(int x) { std::cout << x; return x; } // ❌ error: side effect
```
**Issue**: `constexpr` functions (pre-C++20) cannot have side effects (I/O, mutable state).
**Fix**: Remove side effects or use `consteval` (C++20) for immediate functions.
**Takeaway**: `constexpr` functions must be pure (no side effects).

### 46. `noexcept` Misuse
**Problem**:
```cpp
void func() noexcept { throw 42; } // ❌ UB: noexcept function throws
```
**Issue**: `noexcept` promises no exceptions—violation → `std::terminate()`.
**Fix**: Only mark functions `noexcept` if they truly never throw.
**Takeaway**: `noexcept` is a promise, not a hint—use carefully.

### 47. Template Instantiation in Header
**Problem**:
```cpp
// a.h
template <typename T> void func(T);
// a.cpp
template <typename T> void func(T) { ... }
// main.cpp
#include "a.h"; func(42); // ❌ linker error: no instantiation
```
**Issue**: Templates must be **defined in headers** (or explicit instantiation).
**Fix**: Move definition to header or use explicit instantiation:
```cpp
template void func<int>(int);
```
**Takeaway**: Template definitions belong in headers.

### 48. `auto` Type Deduction Surprises
**Problem**:
```cpp
const int x = 42;
auto y = x; // y is int (not const int)
int& z = x;
auto w = z; // w is int (not int&)
```
**Issue**: `auto` drops `const`/reference qualifiers by default.
**Fix**: Use `const auto`/`auto&`/`auto&&`:
```cpp
const auto y = x; auto& w = z;
```
**Takeaway**: `auto` deduces to value type—add qualifiers explicitly.

### 49. `decltype` vs `auto`
**Problem**:
```cpp
int x = 42;
decltype(x) y = x; // y is int
decltype((x)) z = x; // z is int& (parentheses matter)
```
**Issue**: `decltype` preserves `const`/reference; parentheses change meaning.
**Takeaway**: `decltype((x))` yields a reference type.

### 50. SFINAE Misapplication
**Problem**:
```cpp
template <typename T>
auto func(T t) -> decltype(t.func()) { return t.func(); }
void func(...) { std::cout << "fallback\n"; }
struct S { void func() {} };
func(S()); // OK
func(42); // OK (fallback)
```
**Issue**: SFINAE only works in **immediate context** (function signature).
**Takeaway**: Use SFINAE for overload resolution; avoid complex logic.

### 51. `std::enable_if` Syntax Errors
**Problem**:
```cpp
template <typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void func(T) {} // OK
template <typename T>
std::enable_if_t<std::is_integral_v<T>> func(T) {} // OK
```
**Issue**: Common syntax mistakes lead to compile errors.
**Takeaway**: Master `enable_if` syntax for constrained templates.

### 52. Variadic Template Pack Expansion
**Problem**:
```cpp
template <typename... Args>
void func(Args... args) {
    print(args...); // OK
    print(args)...; // ❌ error
}
```
**Issue**: Pack expansion only works in specific contexts (function calls, initializers).
**Fix**: Use fold expressions (C++17):
```cpp
(print(args), ...);
```
**Takeaway**: Use fold expressions for pack operations (C++17+).

### 53. Template Friend Declarations
**Problem**:
```cpp
template <typename T> class A;
template <typename T> void func(A<T>);
template <typename T> class A { friend void func(A<T>); }; // ❌ not a template friend
```
**Issue**: Friend declaration must match the template.
**Fix**: Explicit template friend:
```cpp
template <typename U> friend void func(A<U>);
```
**Takeaway**: Template friends require explicit template parameters.

### 54. `constexpr` Constructor Limitations
**Problem**:
```cpp
struct A {
    int x;
    constexpr A(int x) : x(x) {} // OK
    constexpr A() { x = 42; } // OK (C++14)
};
constexpr A a; // OK
```
**Issue**: Pre-C++14, `constexpr` constructors must be empty and use member initializers.
**Takeaway**: C++14 relaxed `constexpr` constructor rules.

### 55. Template Metaprogramming (TMP) Infinite Recursion
**Problem**:
```cpp
template <int N> struct Fact { static constexpr int value = N * Fact<N-1>::value; };
template <> struct Fact<0> { static constexpr int value = 1; };
Fact<1000>::value; // ❌ compiler stack overflow
```
**Issue**: TMP recursion depth is limited (compiler-dependent).
**Fix**: Use iterative TMP or constexpr functions (C++14+).
**Takeaway**: Avoid deep TMP recursion; use constexpr for computation.

### 56. `std::is_same` vs `==`
**Problem**:
```cpp
template <typename T, typename U>
void func() {
    if (std::is_same<T, U>::value) { ... } // OK (compile-time)
    if (T == U) { ... } // ❌ error: compare types
}
```
**Issue**: Type comparisons use `std::is_same` (compile-time), not runtime `==`.
**Takeaway**: Use type traits for compile-time type checks.

### 57. `std::decay` Misuse
**Problem**:
```cpp
int arr[10];
using T = decltype(arr); // T = int[10]
using U = std::decay_t<T>; // U = int*
```
**Issue**: `decay` converts arrays/functions to pointers; use when you want value semantics.
**Takeaway**: `std::decay` mimics pass-by-value type conversion.

### 58. Template Argument Deduction (TAD) Failure
**Problem**:
```cpp
template <typename T> void func(std::vector<T>);
func({1,2,3}); // ❌ TAD fails (initializer list not vector)
```
**Issue**: TAD does not work for braced initializers.
**Fix**: Specify T explicitly or use `std::initializer_list`:
```cpp
func<int>({1,2,3});
```
**Takeaway**: Braced initializers require explicit template arguments.

### 59. `std::forward` on Lvalues
**Problem**:
```cpp
void func(int& x) { log(std::forward<int>(x)); } // ❌ forward as rvalue
```
**Issue**: `std::forward` preserves value category—incorrect use breaks forwarding.
**Fix**: Use `std::forward<T>` with forwarding references:
```cpp
template <typename T> void log(T&& x) { std::cout << std::forward<T>(x); }
```
**Takeaway**: `std::forward` only with `T&&` (forwarding references).

### 60. `constexpr` vs `const`
**Problem**:
```cpp
const int x = 42; // compile-time constant
constexpr int y = 42; // compile-time constant (enforced)
const int z = func(); // runtime constant (if func() is not constexpr)
```
**Issue**: `constexpr` guarantees compile-time evaluation; `const` does not.
**Takeaway**: Use `constexpr` for compile-time constants; `const` for runtime immutability.

---

## 4. STL & Containers (61–80)
### 61. `vector` Iterator Invalidation
**Problem**:
```cpp
std::vector<int> v = {1,2,3};
auto it = v.begin();
v.push_back(4); // ❌ invalidates it (reallocation)
*it = 5; // UB
```
**Issue**: `vector` reallocates on capacity change → iterators invalidated.
**Fix**: Use indices or re-acquire iterators after modification.
**Takeaway**: Know iterator invalidation rules for each container.

### 62. `vector::reserve` vs `resize`
**Problem**:
```cpp
std::vector<int> v;
v.reserve(10);
v[0] = 42; // ❌ UB: reserve changes capacity, not size
```
**Issue**: `reserve` allocates memory but does not construct elements; `resize` does both.
**Fix**: Use `resize` for element initialization:
```cpp
v.resize(10); v[0] = 42;
```
**Takeaway**: `reserve` → capacity; `resize` → size + elements.

### 63. `std::list` Random Access
**Problem**:
```cpp
std::list<int> l = {1,2,3};
l[0]; // ❌ error: list has no operator[]
```
**Issue**: `list` is a doubly linked list—**no random access** (O(n) traversal).
**Fix**: Use iterators or `std::advance`.
**Takeaway**: Use `vector` for random access; `list` for frequent insert/erase.

### 64. `std::map` `operator[]` Insertion
**Problem**:
```cpp
std::map<int, int> m;
int x = m[42]; // ❌ inserts {42, 0} if not present
```
**Issue**: `map::operator[]` default-constructs a value if the key is missing.
**Fix**: Use `find()` to check existence:
```cpp
auto it = m.find(42); if (it != m.end()) { x = it->second; }
```
**Takeaway**: `map::operator[]` modifies the map—use `find()` for read-only checks.

### 65. `std::unordered_map` Hash Function
**Problem**:
```cpp
struct S { int x; };
std::unordered_map<S, int> m; // ❌ no hash function for S
```
**Issue**: Custom types need a hash specialization for `unordered_map`.
**Fix**: Provide a hash function:
```cpp
template <> struct std::hash<S> { size_t operator()(const S& s) const { return std::hash<int>()(s.x); } };
```
**Takeaway**: Custom types in unordered containers require hash functions.

### 66. `std::set` Iterator Constness
**Problem**:
```cpp
std::set<int> s = {1,2,3};
auto it = s.begin();
*it = 4; // ❌ error: set elements are const
```
**Issue**: `set` elements are immutable (to maintain ordering).
**Fix**: Erase and reinsert:
```cpp
s.erase(it); s.insert(4);
```
**Takeaway**: `set`/`map` iterators point to const elements.

### 67. `std::string` `c_str()` Lifetime
**Problem**:
```cpp
const char* func() {
    std::string s = "hello";
    return s.c_str(); // ❌ s destroyed → dangling pointer
}
```
**Issue**: `c_str()` returns a pointer to internal data—valid only while the string exists.
**Fix**: Return the string by value or copy the C-string.
**Takeaway**: Never return `c_str()` of a local string.

### 68. `std::string` `find` Return Value
**Problem**:
```cpp
std::string s = "hello";
if (s.find('x')) { ... } // ❌ wrong: find returns std::string::npos (not 0)
```
**Issue**: `find` returns `npos` (size_t, typically `-1`) if not found—non-zero is true.
**Fix**: Compare to `npos`:
```cpp
if (s.find('x') != std::string::npos) { ... }
```
**Takeaway**: Always check `find` result against `npos`.

### 69. `std::sort` Comparator Strict Weak Ordering
**Problem**:
```cpp
std::vector<int> v = {3,1,2};
std::sort(v.begin(), v.end(), [](int a, int b) { return a <= b; }); // ❌ UB
```
**Issue**: Comparator must enforce **strict weak ordering** (no `<=`—use `<`).
**Fix**: Use `<`:
```cpp
std::sort(v.begin(), v.end(), [](int a, int b) { return a < b; });
```
**Takeaway**: Sort comparators must be strict weak orderings.

### 70. `std::unique` Requires Sorted Range
**Problem**:
```cpp
std::vector<int> v = {1,2,1,3};
auto last = std::unique(v.begin(), v.end()); // ❌ no duplicates removed (unsorted)
```
**Issue**: `unique` only removes **consecutive** duplicates—requires sorted input.
**Fix**: Sort first:
```cpp
std::sort(v.begin(), v.end());
auto last = std::unique(v.begin(), v.end());
v.erase(last, v.end());
```
**Takeaway**: `std::unique` works only on sorted ranges.

### 71. `std::remove` Erase-Idiom
**Problem**:
```cpp
std::vector<int> v = {1,2,3,2};
std::remove(v.begin(), v.end(), 2); // ❌ elements not erased
```
**Issue**: `remove` shifts elements but does not resize the container—use erase-remove idiom.
**Fix**:
```cpp
v.erase(std::remove(v.begin(), v.end(), 2), v.end());
```
**Takeaway**: Always use erase-remove for container element removal.

### 72. `std::stack`/`std::queue` Underlying Container
**Problem**:
```cpp
std::stack<int, std::vector<int>> s; // OK (vector as underlying)
```
**Issue**: Default is `deque`—`vector` works but no random access.
**Takeaway**: Adaptors use `deque` by default; can override with `vector`/`list`.

### 73. `std::priority_queue` Top() Modification
**Problem**:
```cpp
std::priority_queue<int> pq;
pq.push(3); pq.push(1);
auto& top = pq.top();
top = 5; // ❌ UB: heap property violated
```
**Issue**: Modifying `top()` breaks the heap invariant.
**Fix**: Pop, modify, push:
```cpp
int val = pq.top(); pq.pop(); val = 5; pq.push(val);
```
**Takeaway**: Never modify `priority_queue::top()` directly.

### 74. `std::array` Size Compile-Time
**Problem**:
```cpp
int n = 10;
std::array<int, n> arr; // ❌ error: size must be compile-time constant
```
**Issue**: `std::array` size is a template parameter—must be known at compile time.
**Fix**: Use `vector` for dynamic sizes.
**Takeaway**: `std::array` is fixed-size (compile-time); `vector` is dynamic.

### 75. `std::bitset` Size Compile-Time
**Problem**:
```cpp
int n = 10;
std::bitset<n> bs; // ❌ error: size must be compile-time constant
```
**Issue**: Same as `array`—`bitset` size is a template parameter.
**Fix**: Use `std::vector<bool>` or `boost::dynamic_bitset`.
**Takeaway**: `std::bitset` is fixed-size; use `vector<bool>` for dynamic bitsets.

### 76. `std::vector<bool>` Specialization Issues
**Problem**:
```cpp
std::vector<bool> v = {true, false};
bool* p = &v[0]; // ❌ error: vector<bool> returns proxy objects
```
**Issue**: `vector<bool>` is a bit-optimized specialization—does not store actual `bool`s.
**Fix**: Use `vector<char>` or `std::array<bool, N>`.
**Takeaway**: Avoid `vector<bool>`—use `vector<char>` instead.

### 77. `std::equal_range` for Non-Sorted Ranges
**Problem**:
```cpp
std::vector<int> v = {3,1,2};
auto [first, last] = std::equal_range(v.begin(), v.end(), 2); // ❌ UB (unsorted)
```
**Issue**: Binary search algorithms (`equal_range`, `lower_bound`, `upper_bound`) require sorted ranges.
**Fix**: Sort first.
**Takeaway**: Binary search algorithms need sorted input.

### 78. `std::for_each` vs Range-For
**Problem**:
```cpp
std::vector<int> v = {1,2,3};
std::for_each(v.begin(), v.end(), [](int& x) { x *= 2; }); // OK
for (int& x : v) { x *= 2; } // OK (preferred)
```
**Issue**: Range-for is more readable; `for_each` is useful for generic code.
**Takeaway**: Prefer range-for for simple loops; `for_each` for complex/generic cases.

### 79. `std::move` on Containers
**Problem**:
```cpp
std::vector<int> v1 = {1,2,3};
std::vector<int> v2 = std::move(v1); // v1 is in valid but unspecified state
v1.push_back(4); // OK (valid)
```
**Issue**: Moved-from containers are **valid but unspecified**—can be reused but not assumed empty.
**Fix**: Do not rely on moved-from container state; reinitialize if needed.
**Takeaway**: Moved-from objects are valid but unspecified.

### 80. `std::inserter` vs Back/Front Inserter
**Problem**:
```cpp
std::vector<int> v;
std::copy(src.begin(), src.end(), std::back_inserter(v)); // OK
std::copy(src.begin(), src.end(), v.begin()); // ❌ UB (no space)
```
**Issue**: Raw iterators require preallocated space; inserters handle insertion.
**Fix**: Use inserters for empty/growing containers.
**Takeaway**: Use `back_inserter`/`front_inserter`/`inserter` for dynamic insertion.

---

## 5. Concurrency & Modern C++ (81–100)
### 81. Data Race on Non-Atomic Variables
**Problem**:
```cpp
int x = 0;
// Thread 1: x++;
// Thread 2: x++;
std::cout << x; // ❌ UB (data race)
```
**Issue**: Non-atomic operations are not thread-safe—race conditions.
**Fix**: Use `std::atomic<int>`:
```cpp
std::atomic<int> x = 0;
```
**Takeaway**: Shared non-atomic variables need synchronization (mutex/atomic).

### 82. `std::mutex` Unlock Failure
**Problem**:
```cpp
std::mutex mtx;
mtx.lock();
// ... code that throws ...
mtx.unlock(); // ❌ unreachable → deadlock
```
**Issue**: Exceptions skip `unlock()` → mutex remains locked.
**Fix**: Use RAII locks (`std::lock_guard`, `std::unique_lock`):
```cpp
std::lock_guard<std::mutex> lock(mtx);
```
**Takeaway**: Always use RAII for mutex locking.

### 83. `std::unique_lock` vs `std::lock_guard`
**Problem**:
```cpp
std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
lock.lock(); // OK
lock.unlock(); // OK
```
**Issue**: `lock_guard` is strict (lock on construction, unlock on destruction); `unique_lock` is flexible (defer, try-lock, unlock).
**Takeaway**: Use `lock_guard` for simple scoped locks; `unique_lock` for flexibility.

### 84. `std::condition_variable` Spurious Wakeups
**Problem**:
```cpp
std::condition_variable cv;
std::mutex mtx;
bool ready = false;
// Waiter:
std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock); // ❌ UB: spurious wakeup
```
**Issue**: `condition_variable` can wake up without a signal—**spurious wakeups**.
**Fix**: Use a predicate loop:
```cpp
cv.wait(lock, [&ready]() { return ready; });
```
**Takeaway**: Always use a predicate with `condition_variable::wait()`.

### 85. `std::thread` Detachment Issues
**Problem**:
```cpp
void func() { std::cout << "hello\n"; }
std::thread t(func);
t.detach(); // ❌ main exits before t runs → UB
```
**Issue**: Detached threads are not joined—main may exit first → UB.
**Fix**: Use `t.join()` or ensure thread lifetime.
**Takeaway**: Prefer `join()` over `detach()`; manage thread lifetimes carefully.

### 86. `std::async` Launch Policy
**Problem**:
```cpp
auto fut = std::async(func); // ❌ implementation-defined (may be deferred)
fut.get(); // may run synchronously
```
**Issue**: Default `std::async` launch policy is `std::launch::any` (deferred or async).
**Fix**: Specify policy:
```cpp
auto fut = std::async(std::launch::async, func); // force async
```
**Takeaway**: Always specify `std::launch::async` for true parallelism.

### 87. `std::future` Shared State
**Problem**:
```cpp
std::promise<int> p;
auto fut = p.get_future();
p.set_value(42);
p.set_value(100); // ❌ UB: set_value called twice
```
**Issue**: `promise` can only be fulfilled once—multiple calls → UB.
**Fix**: Set value once; use `std::shared_future` for multiple waits.
**Takeaway**: `promise` is single-use; `shared_future` for multiple consumers.

### 88. `std::atomic` Load/Store Ordering
**Problem**:
```cpp
std::atomic<int> x = 0;
// Thread 1: x.store(42, std::memory_order_relaxed);
// Thread 2: int y = x.load(std::memory_order_relaxed);
```
**Issue**: `relaxed` ordering provides no synchronization—use for independent operations.
**Fix**: Use `std::memory_order_acquire`/`std::memory_order_release` for synchronization.
**Takeaway**: Choose memory orders carefully—`seq_cst` is safest but slowest.

### 89. `std::lock` for Multiple Mutexes
**Problem**:
```cpp
std::mutex m1, m2;
// Thread 1: m1.lock(); m2.lock();
// Thread 2: m2.lock(); m1.lock(); // ❌ deadlock
```
**Issue**: Lock order inversion → deadlock.
**Fix**: Use `std::lock` to lock multiple mutexes atomically:
```cpp
std::lock(m1, m2);
std::lock_guard<std::mutex> lock1(m1, std::adopt_lock);
std::lock_guard<std::mutex> lock2(m2, std::adopt_lock);
```
**Takeaway**: Use `std::lock` for multiple mutexes to avoid deadlock.

### 90. `std::jthread` (C++20) RAII Joining
**Problem**:
```cpp
std::jthread t(func); // ✅ joins automatically on destruction
```
**Issue**: `std::thread` requires manual `join()`/`detach()`; `jthread` joins by default.
**Takeaway**: Prefer `std::jthread` (C++20) for automatic joining.

### 91. Lambda Capture by Reference in Threads
**Problem**:
```cpp
void func() {
    int x = 42;
    std::thread t([&x]() { std::cout << x; }); // ❌ x destroyed → UB
    t.detach();
}
```
**Issue**: Lambda captures local variable by reference—variable destroyed before thread runs.
**Fix**: Capture by value or use `std::move`.
**Takeaway**: Never capture local variables by reference in detached threads.

### 92. `std::chrono` Duration Casting
**Problem**:
```cpp
std::chrono::milliseconds ms(100);
std::chrono::seconds s = ms; // ❌ error: narrowing conversion
```
**Issue**: Duration casts require explicit conversion to avoid narrowing.
**Fix**: Use `std::chrono::duration_cast`:
```cpp
auto s = std::chrono::duration_cast<std::chrono::seconds>(ms);
```
**Takeaway**: Use `duration_cast` for cross-duration conversions.

### 93. `std::chrono::system_clock` vs `steady_clock`
**Problem**:
```cpp
auto start = std::chrono::system_clock::now();
// ... code ...
auto end = std::chrono::system_clock::now(); // ❌ system clock may be adjusted
```
**Issue**: `system_clock` is not monotonic (can be set back); `steady_clock` is monotonic.
**Fix**: Use `steady_clock` for timing:
```cpp
auto start = std::chrono::steady_clock::now();
```
**Takeaway**: Use `steady_clock` for elapsed time measurements.

### 94. `std::filesystem` Path Handling
**Problem**:
```cpp
std::filesystem::path p = "folder/file.txt";
p /= "subfolder"; // OK (appends correctly)
```
**Issue**: Path concatenation uses `/=` operator—handles OS-specific separators.
**Takeaway**: Use `std::filesystem::path` for cross-platform path handling.

### 95. `std::variant` Visitor Pattern
**Problem**:
```cpp
std::variant<int, std::string> v = 42;
std::visit([](auto&& x) { std::cout << x; }, v); // OK
```
**Issue**: `std::visit` requires a generic lambda to handle all variant types.
**Takeaway**: Use `std::visit` for type-safe variant access.

### 96. `std::optional` Value Check
**Problem**:
```cpp
std::optional<int> opt;
int x = *opt; // ❌ UB: no value
```
**Issue**: Dereferencing empty `optional` → UB.
**Fix**: Check `has_value()` or use `value()` (throws):
```cpp
if (opt.has_value()) { int x = *opt; }
// Or: int x = opt.value(); // throws if empty
```
**Takeaway**: Always check `optional` before dereferencing.

### 97. `std::any` Type Safety
**Problem**:
```cpp
std::any a = 42;
int x = std::any_cast<int>(a); // OK
std::string s = std::any_cast<std::string>(a); // ❌ throws std::bad_any_cast
```
**Issue**: `any_cast` throws if type mismatch—use `has_value()` and `type()` to check.
**Takeaway**: `std::any` is type-safe but requires explicit type checks.

### 98. C++17 Structured Bindings
**Problem**:
```cpp
std::map<int, std::string> m = {{1, "one"}};
for (auto& [k, v] : m) { v = "ONE"; } // OK (C++17)
```
**Issue**: Structured bindings simplify tuple/pair/map access—use `&` for modification.
**Takeaway**: Use structured bindings for clean container iteration.

### 99. C++20 Concepts
**Problem**:
```cpp
template <std::integral T>
T func(T x) { return x * 2; } // OK (constrained template)
```
**Issue**: Concepts replace SFINAE for cleaner constrained templates.
**Takeaway**: Use C++20 concepts for type-safe generic code.

### 100. Undefined Behavior (UB) General Pitfall
**Problem**: Any code that relies on UB (e.g., signed integer overflow, null dereference, out-of-bounds access).
**Issue**: UB means the compiler can do anything—no guarantees.
**Fix**: Avoid UB at all costs; use sanitizers (AddressSanitizer, UndefinedBehaviorSanitizer) to detect issues.
**Takeaway**: UB is the #1 enemy—write code with well-defined behavior.

---

## Final Takeaways for Interviews
1. **Master the Basics**: Memory management, OOP, STL, and concurrency are non-negotiable.
2. **Know Modern C++**: C++11/14/17/20 features (smart pointers, move semantics, lambdas, concepts) are frequently tested.
3. **Avoid UB**: Interviewers love to test UB scenarios—identify and fix them quickly.
4. **Practice**: Write code, use sanitizers, and review common pitfalls.

This list covers the **top 100 C++ interview pitfalls** you’re likely to encounter. Review each entry, understand the root cause, and practice the fixes—you’ll be well-prepared for any C++ interview.
