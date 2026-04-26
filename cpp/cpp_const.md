# Top 20 C++ Interview Questions
## The `const` Keyword

---

## Table of Contents
1. [Fundamentals (Q1–Q7)](#fundamentals)
2. [const with Pointers & References (Q8–Q12)](#const-with-pointers--references)
3. [const in Classes (Q13–Q17)](#const-in-classes)
4. [constexpr & Advanced const (Q18–Q20)](#constexpr--advanced-const)

---

## Fundamentals

---

### Q1. What does `const` mean in C++?

**Answer:**  
`const` declares that a variable's value cannot be modified after initialization. It is a compile-time contract enforced by the type system. Attempting to modify a `const` variable is a compile error.

```cpp
const int x = 10;
// x = 20; // ERROR: assignment of read-only variable

const double PI = 3.14159;
std::cout << PI; // Output: 3.14159
```

---

### Q2. What is the difference between `const` and `#define` for constants?

**Answer:**  

| Feature          | `const`                   | `#define`               |
|------------------|---------------------------|-------------------------|
| Type safety      | ✅ Typed                  | ❌ No type              |
| Scope            | ✅ Respects scope         | ❌ Global text replace  |
| Debuggable       | ✅ Visible in debugger    | ❌ Replaced before compile |
| Evaluated        | At compile or runtime     | Pure text substitution  |

```cpp
#define MAX_SIZE 100       // no type, no scope
const int maxSize = 100;   // typed, scoped, debuggable

// #define danger:
#define SQUARE(x) x*x
int r = SQUARE(2+3); // expands to 2+3*2+3 = 11, not 25!

// const is safe:
const int side = 2 + 3;   // evaluated once, correctly
```

---

### Q3. What is a `const` local variable vs. a `const` global variable?

**Answer:**  
A `const` local variable has block scope and is typically stack-allocated. A `const` global has file scope and static storage. In C++, `const` globals at namespace scope have **internal linkage** by default (unlike non-const globals).

```cpp
const int GLOBAL_MAX = 500;  // internal linkage by default

void func() {
    const int LOCAL_LIMIT = 100; // only visible inside func
    std::cout << LOCAL_LIMIT;
}

// extern const int GLOBAL_MAX; // needed to share across files
```

---

### Q4. Can a `const` variable be initialized at runtime?

**Answer:**  
Yes. `const` only means the variable cannot be changed **after** initialization — the initial value itself can come from a runtime expression.

```cpp
int getUserInput() { return 42; } // runtime value

const int limit = getUserInput(); // OK: initialized once at runtime
// limit = 99; // ERROR: cannot modify after initialization

std::cout << limit; // Output: 42
```

---

### Q5. What is `const` correctness?

**Answer:**  
`const` correctness is the discipline of marking every variable, parameter, and member function `const` wherever mutation is not needed. It prevents accidental modification and signals intent clearly to readers and compilers.

```cpp
// Bad: accepts mutable ref, but doesn't need to modify
void printLength(std::string& s) { std::cout << s.size(); }

// Good: communicates read-only intent
void printLength(const std::string& s) { std::cout << s.size(); }

// Caller benefit: can now pass temporaries and const objects
const std::string msg = "hello";
printLength(msg); // works only with const& version
```

---

### Q6. What is the `const` qualifier on a function parameter?

**Answer:**  
Marking a parameter `const` prevents the function body from modifying it. For value parameters, it only affects the local copy. For reference/pointer parameters, it prevents modification of the original object.

```cpp
void byValue(const int x) {
    // x = 10; // ERROR: can't modify local copy
    std::cout << x;
}

void byRef(const std::string& s) {
    // s += "!"; // ERROR: can't modify original
    std::cout << s;
}

byValue(5);
byRef("hello");
```

---

### Q7. What is top-level vs. low-level `const`?

**Answer:**  
- **Top-level const**: the object itself is `const` (e.g., `const int x`).
- **Low-level const**: the object pointed/referred to is `const` (e.g., `const int* p`).

Top-level `const` is ignored in function signatures for overloading; low-level `const` is not.

```cpp
const int x = 5;         // top-level: x itself is const
int* const p = &someInt; // top-level: the pointer is const
const int* q = &x;       // low-level: the int pointed to is const
const int* const r = &x; // both: pointer and pointee are const
```

---

## const with Pointers & References

---

### Q8. What is the difference between `const int*`, `int* const`, and `const int* const`?

**Answer:**  
Read right-to-left from the `*`:
- `const int*` — pointer to const int (can't change the value; **can** move the pointer)
- `int* const` — const pointer to int (can change the value; **can't** move the pointer)
- `const int* const` — const pointer to const int (neither the value nor the pointer can change)

```cpp
int a = 1, b = 2;

const int* p1 = &a;   // pointer-to-const
// *p1 = 10;          // ERROR: can't change value
p1 = &b;              // OK: can move pointer

int* const p2 = &a;   // const-pointer
*p2 = 10;             // OK: can change value
// p2 = &b;           // ERROR: can't move pointer

const int* const p3 = &a; // const-pointer-to-const
// *p3 = 10;          // ERROR
// p3 = &b;           // ERROR
```

---

### Q9. Why should you prefer `const` references for function parameters?

**Answer:**  
Passing by `const&` avoids copying (efficient for large objects) while preventing modification of the original. It also allows binding to temporaries and `const` objects — something a non-const reference cannot do.

```cpp
struct BigData { std::vector<int> data; };

void process(const BigData& d) { // no copy, no modification
    std::cout << d.data.size();
}

// Can accept:
BigData bd;
process(bd);             // named object
process(BigData{});      // temporary — works with const&
// process(BigData{}) would fail with BigData& (non-const ref)
```

---

### Q10. Can you convert a `const` pointer to a non-const pointer?

**Answer:**  
Not implicitly. You must use `const_cast`, which removes constness. This is **only safe** if the underlying object is truly non-const; casting away const on a genuinely const object and then modifying it is undefined behavior.

```cpp
const int x = 10;
const int* cp = &x;

int* p = const_cast<int*>(cp); // removes const
// *p = 20; // UNDEFINED BEHAVIOR: x is genuinely const

// Safe use of const_cast (original object is non-const):
int y = 42;
const int* cy = &y;
int* py = const_cast<int*>(cy);
*py = 99; // OK: y was never truly const
std::cout << y; // Output: 99
```

---

### Q11. What is a `const` reference to a temporary (lifetime extension)?

**Answer:**  
Binding a `const` reference to a temporary extends the temporary's lifetime to match the reference. This does NOT work with non-const references.

```cpp
const std::string& ref = std::string("hello"); // lifetime extended!
std::cout << ref; // Output: hello — temporary still alive

// std::string& bad = std::string("hi"); // ERROR: non-const ref to temp
```

---

### Q12. Can a `const` reference bind to a different type?

**Answer:**  
Yes — if an implicit conversion exists. The compiler creates a temporary of the target type and the `const` reference binds to it (with lifetime extension).

```cpp
double d = 3.14;
const int& r = d; // implicit conversion: temporary int created
std::cout << r;   // Output: 3 (truncated)
// r = 5;         // ERROR: const reference
```

---

## const in Classes

---

### Q13. What is a `const` member function?

**Answer:**  
A member function declared `const` promises not to modify any non-mutable member of the object. `const` objects can only call `const` member functions. The `const` qualifier is part of the function signature.

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}

    double area() const {          // const: won't modify object
        return 3.14159 * radius * radius;
    }

    void setRadius(double r) {     // non-const: modifies object
        radius = r;
    }
};

const Circle c(5.0);
std::cout << c.area();    // OK: const function
// c.setRadius(3.0);      // ERROR: non-const on const object
```

---

### Q14. What is `mutable` and how does it interact with `const`?

**Answer:**  
`mutable` allows a specific member variable to be modified even inside a `const` member function. Used for caching, lazy evaluation, and logging that doesn't logically change the object's observable state.

```cpp
class ExpensiveCalc {
    mutable double cachedResult = -1;
    mutable bool computed = false;
    double rawData;
public:
    ExpensiveCalc(double d) : rawData(d) {}

    double result() const {
        if (!computed) {
            cachedResult = rawData * rawData; // mutable: OK in const
            computed = true;
        }
        return cachedResult;
    }
};

const ExpensiveCalc ec(7.0);
std::cout << ec.result(); // Output: 49 — computed and cached
std::cout << ec.result(); // Output: 49 — returned from cache
```

---

### Q15. What is a `const` data member and how must it be initialized?

**Answer:**  
A `const` data member cannot be assigned after construction. It **must** be initialized via the member initializer list in the constructor. It cannot be initialized in the constructor body.

```cpp
class Config {
    const int maxConnections;
    const std::string host;
public:
    Config(int max, std::string h)
        : maxConnections(max), host(std::move(h)) {} // initializer list required

    void show() const {
        std::cout << host << ":" << maxConnections;
    }
};

Config cfg(10, "localhost");
cfg.show(); // Output: localhost:10
```

---

### Q16. What is overloading on `const`?

**Answer:**  
A class can have two versions of a member function — one `const` and one non-const. The compiler selects the correct version based on whether the object is `const`.

```cpp
class Container {
    std::vector<int> data = {1, 2, 3};
public:
    int& operator[](int i)       { return data[i]; } // non-const: mutable ref
    const int& operator[](int i) const { return data[i]; } // const: read-only ref
};

Container c;
c[0] = 99;              // calls non-const version

const Container cc;
std::cout << cc[0];     // calls const version
// cc[0] = 5;           // ERROR: returns const ref
```

---

### Q17. What is `const` propagation through member access?

**Answer:**  
When an object is `const`, all its members become effectively `const` through it. Accessing a pointer member through a `const` object means the **pointer** is const, but not necessarily what it points to (shallow const).

```cpp
class Wrapper {
public:
    int* ptr;
    Wrapper(int* p) : ptr(p) {}
};

int val = 42;
const Wrapper w(&val);
// w.ptr = nullptr; // ERROR: ptr itself is const (top-level)
*w.ptr = 99;        // OK! The int pointed to is NOT const (shallow)
std::cout << val;   // Output: 99
```

---

## constexpr & Advanced const

---

### Q18. What is `constexpr` and how does it differ from `const`?

**Answer:**  
- `const` — value cannot change after initialization; may be evaluated at runtime.
- `constexpr` — value **must** be evaluable at **compile time**; implies `const`.

`constexpr` enables use in template parameters, array sizes, and other compile-time contexts.

```cpp
const int a = 5;           // may be runtime
constexpr int b = 10;      // must be compile-time

int arr1[a];               // may not compile (compiler-dependent)
int arr2[b];               // always OK: compile-time constant

constexpr int square(int x) { return x * x; }
constexpr int s = square(7); // evaluated at compile time
static_assert(s == 49);      // verified at compile time
```

---

### Q19. What is `consteval` (C++20)?

**Answer:**  
`consteval` declares an **immediate function** — it must be evaluated at compile time. Any call that cannot be evaluated at compile time is a compile error (stricter than `constexpr`).

```cpp
consteval int compileSquare(int x) { return x * x; }

constexpr int result = compileSquare(6); // OK: compile-time
std::cout << result; // Output: 36

// int n = 5;
// int bad = compileSquare(n); // ERROR: n is not constexpr
```

---

### Q20. What is `constinit` (C++20)?

**Answer:**  
`constinit` ensures a variable is **initialized at compile time** (static initialization), preventing the **static initialization order fiasco**. Unlike `constexpr`, `constinit` variables can be modified after initialization.

```cpp
constinit int globalCounter = 0; // guaranteed compile-time init
// constinit int bad = someRuntimeFunc(); // ERROR: not constexpr-init

void increment() {
    globalCounter++; // OK: constinit doesn't mean const
}

int main() {
    increment();
    std::cout << globalCounter; // Output: 1
}
```

---

## Quick Reference

```cpp
// --- const variables ---
const int x = 5;              // cannot change
constexpr int y = 10;         // compile-time constant
constinit int z = 0;          // compile-time init, runtime-mutable (C++20)

// --- const pointers (read right-to-left) ---
const int* p1 = &x;           // ptr to const int   (value locked)
int* const p2 = &someInt;     // const ptr to int   (pointer locked)
const int* const p3 = &x;     // both locked

// --- const parameters ---
void f(const int n);          // local copy is const
void g(const int& r);         // reference to const — preferred for objects
void h(const int* p);         // pointer to const int

// --- const member functions ---
class Foo {
    mutable int cache = 0;
public:
    int get() const { return cache; }         // won't modify object
    void set(int v) { cache = v; }            // modifies object
    int& operator[](int i);                   // non-const overload
    const int& operator[](int i) const;       // const overload
};

// --- const member variables ---
class Bar {
    const int id;
public:
    Bar(int i) : id(i) {}  // must use initializer list
};
```

---

*End of Top 20 C++ `const` Interview Questions*
