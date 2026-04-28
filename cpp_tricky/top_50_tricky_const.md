Here are **50 tricky C++ interview questions** focusing on **Const Correctness & Type Safety** — covering `const` qualifiers, const member functions, const overloading, mutable, `const_cast`, type safety practices, `enum class`, strong typing, conversions, and more — with answers and code examples.

---

## 1. What does `const` mean in C++?

**Answer:**  
`const` means the value cannot be modified after initialization. It's a compile-time guarantee.

```cpp
const int x = 10;
x = 20; // error
```

---

## 2. What is the difference between `const int*`, `int const*`, and `int* const`?

**Answer:**  
- `const int*` / `int const*`: pointer to const int (int can't change via pointer)
- `int* const`: const pointer to int (pointer can't change)

```cpp
int x = 5, y = 10;
const int* p1 = &x; // OK, *p1 = 20; error, p1 = &y; OK
int* const p2 = &x; // *p2 = 20; OK, p2 = &y; error
```

---

## 3. What is a `const` member function?

**Answer:**  
Function that doesn't modify the object. Can be called on `const` objects.

```cpp
class A { int x; public: int get() const { return x; } };
```

---

## 4. What happens if you modify a member in a `const` member function?

**Answer:**  
Compilation error unless member is `mutable`.

---

## 5. What is `mutable` for?

**Answer:**  
Allows member modification in `const` functions (caching, logging, mutex).

```cpp
class A { mutable int cache; public: int get() const { cache++; return cache; } };
```

---

## 6. Can you call a non-const member function on a const object?

**Answer:**  
No — only const member functions.

```cpp
const A a;
a.get(); // OK if get() const
a.modify(); // error
```

---

## 7. What is the output?

```cpp
struct A { void f() const { cout << "const"; } void f() { cout << "non-const"; } };
int main() {
    A a; const A& ra = a;
    a.f(); ra.f();
}
```

**Answer:**  
`non-const const` — overload resolution picks based on constness of object.

---

## 8. What is `const` overloading?

**Answer:**  
Two functions same name, one const, one non-const. Used in iterators.

```cpp
T& operator[](int i);
const T& operator[](int i) const;
```

---

## 9. Can a const member function modify `*this`?

**Answer:**  
No — `this` is `const A*` inside const member function.

---

## 10. What is `const_cast` used for?

**Answer:**  
Removes constness — dangerous, usually for legacy API that should take const.

```cpp
void legacy(char* p) { *p = 'a'; }
const char* s = "hello";
legacy(const_cast<char*>(s)); // UB if string literal
```

---

## 11. When is `const_cast` safe?

**Answer:**  
Removing const from a variable that is not originally const.

```cpp
int x = 10; const int& r = x; const_cast<int&>(r) = 20; // OK
```

---

## 12. Is `const_cast` the only cast that removes const?

**Answer:**  
Yes — `static_cast`, `reinterpret_cast`, `dynamic_cast` cannot remove const.

---

## 13. What is the difference between `const` and `constexpr`?

**Answer:**  
`const` — runtime or compile-time constant; `constexpr` — guaranteed compile-time constant.

```cpp
const int a = 10; // may be runtime
constexpr int b = 20; // compile-time
```

---

## 14. Can a `constexpr` function modify parameters?

**Answer:**  
No — must be evaluatable at compile-time constraints (C++11), relaxed in C++14.

```cpp
constexpr int square(int x) { return x * x; } // OK
```

---

## 15. What is a constexpr constructor?

**Answer:**  
Constructor that can be evaluated at compile-time for constexpr objects.

```cpp
struct Point { int x,y; constexpr Point(int a, int b) : x(a), y(b) {} };
constexpr Point p(1,2);
```

---

## 16. What is type safety? Is C++ type-safe?

**Answer:**  
Type safety prevents operations on mismatched types. C++ is mostly but not fully type-safe (C-style casts, unions, `void*`).

---

## 17. What is a `strong typedef`? How to implement?

**Answer:**  
Create new distinct type with same representation. Use wrapper or `BOOST_STRONG_TYPEDEF`.

```cpp
struct Meters { double value; explicit Meters(double v) : value(v) {} };
struct Kilometers { double value; explicit Kilometers(double v) : value(v) {} };
```

---

## 18. Why prefer `enum class` over plain `enum`?

**Answer:**  
Scoped, strong type, no implicit conversion.

```cpp
enum Color { Red, Green }; // unscoped
enum class Status { Ok, Fail }; // scoped
Color c = Red; // OK
Status s = Status::Ok; // OK
int x = Red; // implicit conversion
int y = Status::Ok; // error
```

---

## 19. What is the output?

```cpp
const int x = 5;
int* p = const_cast<int*>(&x);
*p = 10;
cout << x;
```

**Answer:**  
Possible 5 (const折叠) or 10 or crash — modifying originally const variable is UB.

---

## 20. What is `const` correctness principle?

**Answer:**  
Mark everything `const` that can be — improves readability, safety, and optimization.

---

## 21. What is a `const` reference? Why useful?

**Answer:**  
Binds to temporary, extends lifetime, avoids copy.

```cpp
void f(const string& s);
f("hello"); // binds to temporary
```

---

## 22. Can you bind a temporary to non-const reference?

**Answer:**  
No — only `const` reference (or rvalue reference).

```cpp
int& r = 5; // error
const int& cr = 5; // OK
int&& rr = 5; // OK (C++11)
```

---

## 23. What is upcasting and downcasting with const?

**Answer:**  
Upcasting: derived* → base* (preserves const). Downcasting: base* → derived* (requires `const_cast` if const).

---

## 24. What is the meaning of `const` after a function declaration?

**Answer:**  
Function does not modify the object (const member function).

---

## 25. What is `const volatile` used for?

**Answer:**  
Memory-mapped I/O registers — not modified by program (const) but can change externally (volatile).

---

## 26. Does `const` member function guarantee thread-safety?

**Answer:**  
No — mutable members still need synchronization.

---

## 27. What is the difference between `const T*` and `T const*`?

**Answer:**  
No difference — both pointer to const T.

---

## 28. What is a `const` iterator vs `const_iterator`?

**Answer:**  
`const iterator` — iterator itself const (can't move).  
`const_iterator` — points to const value (can't modify element).

```cpp
vector<int> v;
const vector<int>::iterator it = v.begin(); // can't change it, but *it = 10 OK
vector<int>::const_iterator cit = v.begin(); // can change cit, *cit = 10 error
```

---

## 29. Can we have a `const` virtual function?

**Answer:**  
Yes — overrides must also be const.

```cpp
struct Base { virtual int get() const; };
struct Derived { int get() const override; };
```

---

## 30. What is a `const` member variable? How to initialize?

**Answer:**  
Must be initialized in member initializer list, cannot be assigned.

```cpp
class A { const int x; public: A(int v) : x(v) {} };
```

---

## 31. What is the return type of this function: `const A& f() const`?

**Answer:**  
Returns const reference to A; function doesn't modify object; caller can't modify returned object.

---

## 32. Why return `const` reference?

**Answer:**  
Avoid copy, prevent caller from modifying internal state.

```cpp
const vector<int>& getData() const { return data; }
```

---

## 33. What is a `const` lambda?

**Answer:**  
C++17 — mutable lambdas; `const` not directly, but capture by value then const.

---

## 34. What is the output?

```cpp
const char* msg = "hello";
char* p = const_cast<char*>(msg);
p[0] = 'j';
cout << msg;
```

**Answer:**  
Undefined behavior (string literal in read-only memory) — likely crash or unchanged.

---

## 35. How to make a function parameter `const`? Why?

**Answer:**  
`void f(const int& x)` — prevents modification, communicates intent.

---

## 36. What is the "const-correctness" of std::begin and std::cbegin?

**Answer:**  
`cbegin` returns const_iterator even if container non-const.

---

## 37. Can a constructor be `const`?

**Answer:**  
No — constructor modifies object.

---

## 38. Can a destructor be `const`?

**Answer:**  
No — destructor modifies object.

---

## 39. What is a `const` pointer to `const` data?

**Answer:**  
```cpp
const int* const p = &x; // neither pointer nor data can be changed
```

---

## 40. Does `const` affect storage duration?

**Answer:**  
No — only access restriction.

---

## 41. What is the purpose of `std::as_const` (C++17)?

**Answer:**  
Convenience function to add const.

```cpp
auto& constRef = std::as_const(obj);
```

---

## 42. Why prefer `const T&` over `T` for parameters?

**Answer:**  
Avoids copy, works with temporaries, const ensures no modification.

---

## 43. Can you overload on const vs non-const pointer?

**Answer:**  
Yes — different parameter types.

```cpp
void f(int* p);
void f(const int* p);
```

---

## 44. What is the "constness" of a pointer in a const member function?

**Answer:**  
`this` is `const A*` — member pointers become `T* const`, not `const T*`.

---

## 45. What is `std::add_const` type trait?

**Answer:**  
Adds const qualifier to type.

```cpp
std::add_const<int>::type x = 10; // const int
```

---

## 46. Is `const` necessary for move semantics?

**Answer:**  
No — `const` inhibits moving (move constructor usually non-const).

---

## 47. What is the "const inheritance" rule?

**Answer:**  
Constness isn't inherited — each class member function independently const.

---

## 48. Output of:

```cpp
class A { int x; public: void f() const { x++; } };
```

**Answer:**  
Error — modifying `x` in const function.

---

## 49. What is logical constness vs bitwise constness?

**Answer:**  
- Bitwise: no bits changed (C++ default)  
- Logical: observable state unchanged, but internal mutable allowed

```cpp
class Cache { mutable int cached; public: int get() const { cached = compute(); return cached; } };
```

---

## 50. How to enforce const correctness in a team?

**Answer:**  
- Use `-Wcast-qual` compiler flag  
- Use `const` by default, remove only when needed  
- Code review: mark methods const when possible  
- Use `clang-tidy` `readability-const-return-type` and `misc-const-correctness`

---

Let me know if you'd like the complete **300+ questions** across all 6 topics compiled, or a searchable document with examples.