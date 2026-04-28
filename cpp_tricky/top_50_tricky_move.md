Here are **50 tricky C++ interview questions** focusing on **Move Semantics & Perfect Forwarding** — covering rvalue references (`&&`), `std::move`, `std::forward`, copy elision, why move is faster, and common pitfalls — with answers and code examples.

---

## 1. What is an rvalue? What is an lvalue?

**Answer:**  
- **lvalue**: has an address (e.g., variable name)  
- **rvalue**: temporary, no address (e.g., `5`, `x + y`, function return)

```cpp
int x = 5;     // x = lvalue, 5 = rvalue
int& y = x;    // OK: lvalue reference to lvalue
int&& z = 5;   // OK: rvalue reference to rvalue
```

---

## 2. What is the syntax of an rvalue reference?

**Answer:**  
`T&&` — binds to temporaries, enables move semantics.

```cpp
void f(int&& x) { cout << x; }
f(10);       // OK
int a = 5;
f(a);        // error: cannot bind rvalue ref to lvalue
```

---

## 3. What does `std::move` do?

**Answer:**  
`std::move` casts an lvalue to an rvalue reference. It doesn't move anything by itself.

```cpp
int x = 10;
int&& r = std::move(x); // casts x to rvalue reference
```

---

## 4. Does `std::move` actually move?

**Answer:**  
No — only casts. The move constructor or move assignment does the actual work.

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1); // move constructor transfers ownership
// s1 is now valid but unspecified (typically empty)
```

---

## 5. Why is move faster than copy?

**Answer:**  
Move transfers resources (pointers, handles) instead of duplicating data.

```cpp
class Buffer {
    int* data;
public:
    // Copy: O(n) allocation + copy
    Buffer(const Buffer& other) : data(new int[1000]) {
        std::copy(other.data, other.data + 1000, data);
    }
    // Move: O(1) pointer swap
    Buffer(Buffer&& other) noexcept : data(other.data) {
        other.data = nullptr;
    }
};
```

---

## 6. When is move constructor called?

**Answer:**  
When initializing from rvalue (temporary, `std::move`, function returning local).

```cpp
A a1;
A a2 = std::move(a1); // move ctor
A a3 = A();           // move ctor (or elided)
```

---

## 7. What is a move constructor signature?

**Answer:**  
`A(A&& other) noexcept;`

```cpp
class A {
public:
    A(A&& other) noexcept : data(other.data) {
        other.data = nullptr;
    }
};
```

---

## 8. Why `noexcept` on move constructor?

**Answer:**  
Containers (`std::vector`) use move only if `noexcept` (strong exception guarantee).

```cpp
static_assert(std::is_nothrow_move_constructible_v<A>);
```

---

## 9. What is the difference between move constructor and copy constructor?

**Answer:**  
Move steals resources (O(1), destructive). Copy duplicates (O(n), preserves original).

---

## 10. What is a move assignment operator?

**Answer:**  
Transfers resources from rvalue to existing object.

```cpp
A& operator=(A&& other) noexcept {
    if (this != &other) {
        delete data;
        data = other.data;
        other.data = nullptr;
    }
    return *this;
}
```

---

## 11. Output of this move vs copy?

```cpp
struct A {
    A() = default;
    A(const A&) { cout << "copy "; }
    A(A&&) { cout << "move "; }
};
A f() { A a; return a; }
int main() { A a = f(); }
```

**Answer:**  
`(nothing)` — copy elision. With `-fno-elide-constructors`: `move`

---

## 12. What is copy elision? How does it interact with move?

**Answer:**  
Compiler omits copy/move entirely. C++17 guarantees elision for prvalues.

```cpp
A make() { return A(); } // no copy/move, constructs directly
```

---

## 13. What is RVO vs NRVO?

**Answer:**  
- **RVO**: Return Value Optimization — returns unnamed temporary.
- **NRVO**: Named RVO — returns named local variable (optional).

```cpp
A f1() { return A(); } // RVO guaranteed since C++17
A f2() { A a; return a; } // NRVO (implementation-dependent)
```

---

## 14. When does copy elision NOT happen?

**Answer:**  
- Multiple return paths with different named objects
- Returning function parameter
- With `-fno-elide-constructors` flag

```cpp
A f(bool b) {
    A a1, a2;
    return b ? a1 : a2; // no elision (must copy/move)
}
```

---

## 15. What is a `std::forward`? Why needed?

**Answer:**  
`std::forward` preserves value category (lvalue/rvalue) for perfect forwarding.

```cpp
template<typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg)); // forwards as lvalue or rvalue
}
```

---

## 16. Why not just `std::move` in forwarding?

**Answer:**  
`std::move` always casts to rvalue; `std::forward` preserves lvalue-ness.

```cpp
void target(int&) { cout << "lvalue "; }
void target(int&&) { cout << "rvalue "; }
template<typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg)); // preserves
    target(std::move(arg));       // always rvalue
}
int x = 5;
wrapper(x); // lvalue rvalue (first preserves, second moves)
```

---

## 17. What is perfect forwarding?

**Answer:**  
Template passes arguments to another function preserving type and value category.

```cpp
template<typename F, typename... Args>
auto invoke(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

---

## 18. What is a forwarding reference (universal reference)?

**Answer:**  
`T&&` in template type deduction context (not `std::vector<T>&&` or concrete type).

```cpp
template<typename T>
void f(T&& param); // forwarding reference

auto&& x = 5; // also forwarding reference
```

---

## 19. What's wrong with overloading on rvalue/lvalue?

**Answer:**  
Proliferates overloads. Better to use forwarding reference for wrappers.

```cpp
void f(const A&);
void f(A&&); // works, but need many overloads
```

---

## 20. Can you move from `const` object?

**Answer:**  
No — move constructor takes non-`const` rvalue reference. `const` forces copy.

```cpp
const std::string s = "hello";
std::string s2 = std::move(s); // copies (const prevents move)
```

---

## 21. What is the state of a moved-from object?

**Answer:**  
Valid but unspecified. Only safe to destroy or assign new value.

```cpp
std::vector<int> v1{1,2,3};
std::vector<int> v2 = std::move(v1);
v1.clear();          // OK
v1.push_back(5);     // OK
// cout << v1[0];    // unsafe (unspecified)
```

---

## 22. Can you use a moved-from object after move?

**Answer:**  
Yes, but only reset it (assign new state) or destroy it.

---

## 23. What is the output?

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);
cout << s1.size();
```

**Answer:**  
`0` (strings typically become empty after move)

---

## 24. Is `std::move` expensive?

**Answer:**  
No — it's just a cast (zero runtime overhead).

---

## 25. What is `std::move_if_noexcept`?

**Answer:**  
Returns rvalue if move is `noexcept`, otherwise lvalue (used by containers).

```cpp
struct A { A(A&&) {} }; // not noexcept
std::move_if_noexcept(a); // returns lvalue (copies)
```

---

## 26. How to force move instead of copy?

**Answer:**  
Wrap with `std::move` or use temporary.

```cpp
std::vector<std::string> v;
std::string s = "hello";
v.push_back(std::move(s)); // moves, s becomes empty
```

---

## 27. What is the output of this move constructor mistake?

```cpp
class A {
    int* p;
public:
    A() : p(new int(10)) {}
    A(A&& other) : p(other.p) {} // forgot to null other.p
    ~A() { delete p; }
};
A a1; A a2 = std::move(a1); // double-free!
```

**Answer:**  
Double-free → crash or UB. Fix: `other.p = nullptr;`

---

## 28. Why should move operations leave object in destructible state?

**Answer:**  
Moved-from object's destructor still runs; must be safe.

```cpp
~A() { delete p; } // p must be valid (nullptr OK)
```

---

## 29. Can you have virtual move constructor?

**Answer:**  
No — constructors cannot be virtual.

---

## 30. What is the difference between `std::move` and `std::forward`?

**Answer:**  
- `std::move`: unconditionally casts to rvalue
- `std::forward`: conditionally casts (based on template argument)

```cpp
std::move(x); // always T&&
std::forward<T>(x); // T&& if T is not lvalue reference
```

---

## 31. When should you use `std::forward` vs `std::move`?

**Answer:**  
- Use `std::move` when you know you have an rvalue
- Use `std::forward` in templates forwarding to preserve value category

```cpp
template<typename T>
void wrapper(T&& arg) {
    // Move: always steals (bad for lvalues)
    // Forward: respects original category
    target(std::forward<T>(arg));
}
```

---

## 32. What is a `std::reference_wrapper`? How relates to move?

**Answer:**  
Copyable wrapper for references. Move doesn't transfer reference.

```cpp
int x = 5;
auto r = std::ref(x);
auto r2 = std::move(r); // still references x
```

---

## 33. What is the output of this perfect forwarding pitfall?

```cpp
void f(const int&) { cout << "lvalue "; }
void f(int&&) { cout << "rvalue "; }
template<typename T>
void wrapper(T&& arg) {
    f(std::move(arg)); // always rvalue
}
int x = 5;
wrapper(x);      // outputs "rvalue" (wrong)
```

**Answer:**  
`rvalue` — `std::move` lost lvalue-ness. Fix: use `std::forward`.

---

## 34. Can you move from array?

**Answer:**  
No — arrays don't have move constructors.

```cpp
int arr1[10];
int arr2[10] = std::move(arr1); // error: copies!
```

**Fix:** Use `std::array` or `std::vector`.

---

## 35. What is move-only type? Give example.

**Answer:**  
Type that can be moved but not copied (e.g., `std::unique_ptr`, `std::thread`).

```cpp
std::unique_ptr<int> p1(new int(5));
std::unique_ptr<int> p2 = std::move(p1);
// std::unique_ptr<int> p3 = p1; // error: copy deleted
```

---

## 36. How to implement move-only class?

**Answer:**  
Delete copy operations, default move operations.

```cpp
class MoveOnly {
public:
    MoveOnly() = default;
    MoveOnly(const MoveOnly&) = delete;
    MoveOnly& operator=(const MoveOnly&) = delete;
    MoveOnly(MoveOnly&&) = default;
    MoveOnly& operator=(MoveOnly&&) = default;
};
```

---

## 37. What is the cost of `std::move` on `shared_ptr`?

**Answer:**  
Cheap — just pointers, no atomic ref-count change.

```cpp
auto sp1 = std::make_shared<int>(5);
auto sp2 = std::move(sp1); // sp1 now null, no atomic ops
```

---

## 38. What is copy-and-swap idiom? How relates to move?

**Answer:**  
Uses copy/move to implement assignment with strong exception guarantee.

```cpp
A& operator=(A other) { // copy or move
    swap(*this, other);
    return *this;
}
```

---

## 39. What is the output of this move with inheritance?

```cpp
class Base { public: Base(Base&&) { cout << "Base move "; } };
class Derived : public Base { public: Derived(Derived&&) { cout << "Derived move "; } };
Derived d1; Derived d2 = std::move(d1);
```

**Answer:**  
`Derived move` only — base not moved automatically. Fix: call base move in initializer list.

```cpp
Derived(Derived&& other) : Base(std::move(other)) {}
```

---

## 40. Why can't you move `const` unique_ptr?

**Answer:**  
`const` prevents modification; moving would nullify source.

```cpp
const std::unique_ptr<int> p(new int(5));
auto p2 = std::move(p); // error: cannot move from const
```

---

## 41. What is a `std::decay`? Used in perfect forwarding?

**Answer:**  
Removes references and cv-qualifiers, converts array to pointer. Used in some forwarding contexts.

```cpp
std::decay_t<const int&> // int
```

---

## 42. What is guaranteed copy elision (C++17)?

**Answer:**  
prvalues (e.g., `A()`) construct directly into destination, no temporary.

```cpp
A f() { return A(); } // no copy/move, guaranteed
```

---

## 43. What is a materialized temporary? (C++17)

**Answer:**  
Temporary created only when needed. prvalue is not an object until "materialized".

---

## 44. Can `std::move` be dangerous?

**Answer:**  
Yes — moving from object and then using it without resetting.

```cpp
std::string s = "hello";
auto s2 = std::move(s);
cout << s; // unspecified, may crash
```

---

## 45. What is the difference between returning `std::move(local)` and returning local?

**Answer:**  
Returning local allows NRVO (elision). Returning `std::move(local)` prevents elision (forces move).

```cpp
A f() { A a; return a; } // NRVO possible
A g() { A a; return std::move(a); } // forces move, blocks NRVO (bad)
```

---

## 46. What is the performance gain of move with `std::vector`?

**Answer:**  
Resizing: move string elements (O(1) each) instead of copy (O(n) each).

---

## 47. How to detect if type is move-constructible?

**Answer:**  
`std::is_move_constructible_v<T>`

---

## 48. What is a `deleted` move constructor effect?

**Answer:**  
Type cannot be moved; forces copy.

```cpp
class A { A(A&&) = delete; };
A a1; A a2 = std::move(a1); // error: move deleted
```

---

## 49. Can lambda capture by move?

**Answer:**  
C++14 init-capture: `[p = std::move(ptr)]`

```cpp
auto p = std::make_unique<int>(5);
auto f = [p = std::move(p)] { return *p; };
```

---

## 50. Final tricky: Output of this perfect forwarding?

```cpp
void g(int&) { cout << "lvalue "; }
void g(int&&) { cout << "rvalue "; }
template<typename T>
void f(T&& x) {
    g(x);                      // lvalue
    g(std::move(x));           // rvalue
    g(std::forward<T>(x));     // depends on T
}
int main() {
    int a = 5;
    f(a);  // T = int&
    f(10); // T = int
}
```

**Answer:**  
For `f(a)`: `lvalue rvalue lvalue` (forward preserves lvalue)  
For `f(10)`: `lvalue rvalue rvalue` (forward becomes rvalue)

---
