# C++ Interview Questions: Smart Pointers

> **50 curated questions with answers and code examples** — covering `unique_ptr`, `shared_ptr`, `weak_ptr`, custom deleters, internals, and best practices.

---

## Table of Contents

1. [Fundamentals (Q1–Q8)](#fundamentals)
2. [unique_ptr (Q9–Q18)](#unique_ptr)
3. [shared_ptr (Q19–Q30)](#shared_ptr)
4. [weak_ptr (Q31–Q38)](#weak_ptr)
5. [Custom Deleters & Advanced Features (Q39–Q44)](#custom-deleters--advanced-features)
6. [Best Practices & Pitfalls (Q45–Q50)](#best-practices--pitfalls)

---

## Fundamentals

---

### Q1. What is a smart pointer and why do we need it?

**Answer:**  
A smart pointer is a class template that wraps a raw pointer and manages the lifetime of the object it points to. It uses RAII — the resource is released automatically when the smart pointer goes out of scope, preventing memory leaks and dangling pointers.

```cpp
#include <memory>

// Raw pointer: manual, error-prone
void rawExample() {
    int* p = new int(42);
    // ... if exception thrown here, p is never deleted!
    delete p;
}

// Smart pointer: automatic, safe
void smartExample() {
    auto p = std::make_unique<int>(42);
    std::cout << *p; // 42
} // p destroyed here automatically — no delete needed
```

---

### Q2. What are the three standard smart pointer types in C++11?

**Answer:**

| Smart Pointer | Ownership | Use Case |
|---|---|---|
| `std::unique_ptr<T>` | Sole/exclusive | Single owner, zero overhead |
| `std::shared_ptr<T>` | Shared/reference-counted | Multiple owners |
| `std::weak_ptr<T>` | Non-owning observer | Break cycles, optional access |

```cpp
#include <memory>

// unique_ptr: one owner
auto u = std::make_unique<int>(1);

// shared_ptr: multiple owners
auto s1 = std::make_shared<int>(2);
auto s2 = s1; // ref count = 2

// weak_ptr: non-owning
std::weak_ptr<int> w = s1; // ref count still 2
```

---

### Q3. What is RAII and how do smart pointers implement it?

**Answer:**  
RAII (Resource Acquisition Is Initialization) ties a resource's lifetime to a stack object. The constructor acquires the resource; the destructor releases it. Smart pointers implement this: the pointer is acquired at construction, and `delete` (or a custom deleter) is called in the destructor.

```cpp
class RawSocket {
    int fd;
public:
    RawSocket(int f) : fd(f) { std::cout << "Socket opened: " << fd << "\n"; }
    ~RawSocket() { std::cout << "Socket closed: " << fd << "\n"; close(fd); }
};

void process() {
    auto sock = std::make_unique<RawSocket>(42);
    // ... work ...
    throw std::runtime_error("oops");
    // ~RawSocket() is STILL called — no leak!
}
```

---

### Q4. What is the difference between a smart pointer and a raw pointer?

**Answer:**

| Feature | Raw Pointer | Smart Pointer |
|---|---|---|
| Memory management | Manual (`delete`) | Automatic |
| Null safety | No protection | `get()` returns `nullptr` safely |
| Ownership semantics | Unclear | Explicit (`unique`/`shared`/`weak`) |
| Exception safety | Unsafe | Safe (RAII) |
| Overhead | None | `unique_ptr`: zero; `shared_ptr`: ref count |

```cpp
// Raw: ownership unclear, leak-prone
int* raw = new int(10);
// forgot delete raw; → memory leak

// Smart: ownership explicit, automatic cleanup
auto smart = std::make_unique<int>(10);
// no delete needed; ownership is clear
```

---

### Q5. What header do smart pointers live in?

**Answer:**  
All standard smart pointers are in `<memory>`.

```cpp
#include <memory>   // unique_ptr, shared_ptr, weak_ptr
                    // make_unique, make_shared
                    // enable_shared_from_this

auto u = std::make_unique<std::string>("hello");
auto s = std::make_shared<std::vector<int>>(10, 0);
```

---

### Q6. How do you dereference a smart pointer?

**Answer:**  
Smart pointers overload `operator*` (dereference) and `operator->` (member access), just like raw pointers. Use `get()` to obtain the underlying raw pointer when needed.

```cpp
struct Point { int x, y; };

auto p = std::make_unique<Point>(Point{3, 4});

std::cout << (*p).x;   // dereference: 3
std::cout << p->y;     // arrow:       4
std::cout << p.get();  // raw pointer address

// nullptr check (same as raw pointer)
if (p) std::cout << "valid";
```

---

### Q7. What is `std::make_unique` and `std::make_shared` and why prefer them over `new`?

**Answer:**  
Factory functions that construct the managed object and the smart pointer in one step.

Advantages over `new`:
- **Exception safety:** No partial construction leak.
- **`make_shared` efficiency:** Allocates control block + object in one allocation.
- **Clarity:** No explicit `new`/`delete`.

```cpp
// Prefer this:
auto u = std::make_unique<int>(42);
auto s = std::make_shared<std::string>("hello");

// Over this:
std::unique_ptr<int> u2(new int(42));     // two allocations of thought
std::shared_ptr<std::string> s2(new std::string("hello")); // exception risk

// Exception safety issue with raw new:
void foo(std::shared_ptr<int> a, std::shared_ptr<int> b);
// foo(shared_ptr<int>(new int(1)), shared_ptr<int>(new int(2)));
// ^ if second new succeeds but shared_ptr ctor throws → leak!
// foo(make_shared<int>(1), make_shared<int>(2)); // safe
```

---

### Q8. Can smart pointers be stored in containers?

**Answer:**  
Yes. `unique_ptr` can be stored in containers (as move-only elements); `shared_ptr` can be copied freely into containers.

```cpp
#include <vector>
#include <memory>

// vector of unique_ptr (move-only)
std::vector<std::unique_ptr<int>> uVec;
uVec.push_back(std::make_unique<int>(1));
uVec.push_back(std::make_unique<int>(2));
uVec.emplace_back(std::make_unique<int>(3));

// vector of shared_ptr (copyable)
std::vector<std::shared_ptr<std::string>> sVec;
auto s = std::make_shared<std::string>("hello");
sVec.push_back(s);       // ref count increases
sVec.push_back(s);       // ref count = 3
```

---

## unique_ptr

---

### Q9. What is `std::unique_ptr`?

**Answer:**  
`unique_ptr` is a smart pointer with **exclusive ownership** — exactly one `unique_ptr` owns the managed object at any time. It has **zero overhead** compared to a raw pointer (no reference counting). The object is destroyed when the `unique_ptr` is destroyed or reset.

```cpp
auto p = std::make_unique<std::string>("C++");

std::cout << *p;           // C++
std::cout << p->length();  // 3

// When p goes out of scope, string is deleted automatically
```

---

### Q10. Why is `unique_ptr` non-copyable?

**Answer:**  
Copying a `unique_ptr` would create two owners of the same object — violating exclusive ownership. The copy constructor and copy assignment are `delete`d. Only **move** transfers ownership.

```cpp
auto p1 = std::make_unique<int>(42);

// auto p2 = p1;            // Error: copy constructor deleted
// unique_ptr<int> p3 = p1; // Error: copy assignment deleted

// Move is allowed: ownership transfers
auto p2 = std::move(p1);    // p1 is now nullptr
std::cout << (p1 == nullptr); // true
std::cout << *p2;             // 42
```

---

### Q11. How do you transfer ownership with `unique_ptr`?

**Answer:**  
Use `std::move()`. After the move, the source `unique_ptr` becomes `nullptr`.

```cpp
std::unique_ptr<int> owner = std::make_unique<int>(100);

// Transfer to another unique_ptr
std::unique_ptr<int> newOwner = std::move(owner);
std::cout << (owner == nullptr); // true
std::cout << *newOwner;          // 100

// Transfer into a function
void take(std::unique_ptr<int> p) {
    std::cout << *p;
} // p destroyed here

take(std::move(newOwner)); // 100
// newOwner is now nullptr
```

---

### Q12. How do you return a `unique_ptr` from a function?

**Answer:**  
Return by value — the compiler applies NRVO or move semantics automatically. No explicit `std::move` needed on a local variable return.

```cpp
std::unique_ptr<std::vector<int>> makeData(int n) {
    auto v = std::make_unique<std::vector<int>>(n, 0);
    for (int i = 0; i < n; ++i) (*v)[i] = i * i;
    return v; // NRVO or implicit move — no std::move needed
}

auto data = makeData(5);
for (int x : *data) std::cout << x << " "; // 0 1 4 9 16
```

---

### Q13. What is `unique_ptr::release()`?

**Answer:**  
`release()` gives up ownership: it returns the raw pointer and sets the `unique_ptr` to `nullptr`. The caller is now responsible for deleting the returned pointer. Use sparingly.

```cpp
auto p = std::make_unique<int>(55);

int* raw = p.release();       // p is now nullptr
std::cout << (p == nullptr);  // true
std::cout << *raw;            // 55

delete raw; // caller must delete manually!
```

---

### Q14. What is `unique_ptr::reset()`?

**Answer:**  
`reset()` destroys the currently managed object and optionally takes ownership of a new raw pointer (or sets to `nullptr`).

```cpp
auto p = std::make_unique<int>(10);
std::cout << *p; // 10

p.reset(new int(20)); // old int(10) deleted, now owns int(20)
std::cout << *p; // 20

p.reset();            // deletes int(20), p is nullptr
std::cout << (p == nullptr); // true
```

---

### Q15. How does `unique_ptr` work with arrays?

**Answer:**  
`unique_ptr<T[]>` manages dynamic arrays and calls `delete[]` (not `delete`). Use `operator[]` to access elements.

```cpp
// Array form
auto arr = std::make_unique<int[]>(5);
for (int i = 0; i < 5; ++i)
    arr[i] = i * 10;

std::cout << arr[0]; // 0
std::cout << arr[3]; // 30
// delete[] called automatically

// Prefer std::vector or std::array in practice
auto vec = std::make_unique<std::vector<int>>(5, 0);
```

---

### Q16. Can you use `unique_ptr` with a custom deleter?

**Answer:**  
Yes. The deleter type is part of the `unique_ptr` type signature. Useful for resources that need cleanup other than `delete`.

```cpp
// Custom deleter as lambda
auto fileDeleter = [](FILE* f) {
    if (f) { std::cout << "Closing file\n"; fclose(f); }
};

std::unique_ptr<FILE, decltype(fileDeleter)>
    file(fopen("data.txt", "r"), fileDeleter);

if (file) {
    // use file.get() for C FILE* functions
}
// fclose called automatically

// Custom deleter as function
void freeBuffer(uint8_t* buf) { free(buf); }

std::unique_ptr<uint8_t, decltype(&freeBuffer)>
    buf(static_cast<uint8_t*>(malloc(256)), freeBuffer);
```

---

### Q17. How do you convert a `unique_ptr` to a `shared_ptr`?

**Answer:**  
Implicit conversion from `unique_ptr` to `shared_ptr` is supported — it moves the `unique_ptr` into the `shared_ptr`.

```cpp
auto u = std::make_unique<int>(99);

// Implicit conversion: u is moved, becomes nullptr
std::shared_ptr<int> s = std::move(u);

std::cout << (u == nullptr); // true
std::cout << *s;             // 99
std::cout << s.use_count();  // 1
```

---

### Q18. What is the size of a `unique_ptr`?

**Answer:**  
With a default deleter, `unique_ptr` is the **same size as a raw pointer** (zero overhead). With a stateful custom deleter, it grows to accommodate the deleter.

```cpp
std::cout << sizeof(std::unique_ptr<int>);         // 8 (on 64-bit)
std::cout << sizeof(int*);                          // 8

// Stateful deleter adds size
auto del = [x = 42](int* p) { delete p; };         // captures x
std::unique_ptr<int, decltype(del)> p(new int(1), del);
std::cout << sizeof(p);                             // > 8
```

---

## shared_ptr

---

### Q19. What is `std::shared_ptr`?

**Answer:**  
`shared_ptr` provides **shared ownership** through reference counting. Multiple `shared_ptr` instances can point to the same object. The object is destroyed when the last `shared_ptr` owning it is destroyed or reset.

```cpp
auto s1 = std::make_shared<std::string>("shared");
std::cout << s1.use_count(); // 1

{
    auto s2 = s1;            // ref count = 2
    auto s3 = s1;            // ref count = 3
    std::cout << s1.use_count(); // 3
} // s2, s3 destroyed → ref count = 1

std::cout << s1.use_count(); // 1
// s1 destroyed → ref count = 0 → string deleted
```

---

### Q20. What is the control block in `shared_ptr`?

**Answer:**  
The control block is a heap-allocated structure that holds the reference count, weak reference count, and the custom deleter. All `shared_ptr` instances sharing an object point to the same control block.

```cpp
// make_shared: ONE allocation (object + control block together)
auto s1 = std::make_shared<int>(42);

// shared_ptr(new T): TWO allocations (object + separate control block)
std::shared_ptr<int> s2(new int(42));

// make_shared is preferred: fewer allocations, better cache locality
// Diagram:
// make_shared:  [ref_count | weak_count | int(42)]  ← single block
// new + wrap:   [int(42)]  +  [ref_count | weak_count | ptr→int]
```

---

### Q21. What is `use_count()`?

**Answer:**  
`use_count()` returns the number of `shared_ptr` instances sharing ownership. Used for debugging; do not use for synchronization logic.

```cpp
auto a = std::make_shared<int>(1);
std::cout << a.use_count(); // 1

auto b = a;
std::cout << a.use_count(); // 2

auto c = b;
std::cout << a.use_count(); // 3

b.reset();
std::cout << a.use_count(); // 2

c = nullptr;
std::cout << a.use_count(); // 1
```

---

### Q22. Is `shared_ptr` thread-safe?

**Answer:**  
The **reference count** operations are atomic — safe to copy/destroy `shared_ptr` from multiple threads. However, **access to the managed object itself** is NOT automatically thread-safe; you must synchronize access to the object separately.

```cpp
auto shared = std::make_shared<int>(0);

// Safe: copying shared_ptr across threads (ref count is atomic)
std::thread t1([s = shared]() { /* read *s */ });
std::thread t2([s = shared]() { /* read *s */ });

// NOT safe: multiple threads writing to the managed object
// Need mutex for: *shared = 42; (data race!)
std::mutex mtx;
std::thread t3([&]() {
    std::lock_guard<std::mutex> lock(mtx);
    *shared = 42;
});

t1.join(); t2.join(); t3.join();
```

---

### Q23. What is `shared_ptr::unique()`? (Deprecated in C++17, removed C++20)

**Answer:**  
`unique()` returned `true` if `use_count() == 1`. It was deprecated because it's inherently racy in multi-threaded code. Use `use_count() == 1` for single-threaded diagnostic purposes.

```cpp
auto s = std::make_shared<int>(5);
// C++14: s.unique() → true
std::cout << (s.use_count() == 1); // true

auto s2 = s;
std::cout << (s.use_count() == 1); // false (use_count is 2)
```

---

### Q24. Can two separate `shared_ptr` objects manage the same raw pointer?

**Answer:**  
You should **never** create two independent `shared_ptr` instances from the same raw pointer — each will have its own control block and will try to `delete` the object, causing a double-free.

```cpp
int* raw = new int(42);

std::shared_ptr<int> s1(raw);  // control block A, ref count = 1
std::shared_ptr<int> s2(raw);  // control block B, ref count = 1
// DISASTER: when s1 and s2 are both destroyed,
// delete raw is called TWICE → undefined behavior!

// CORRECT: share ownership via copy
std::shared_ptr<int> s3 = std::make_shared<int>(42);
std::shared_ptr<int> s4 = s3; // shares control block A
```

---

### Q25. What is a cyclic reference with `shared_ptr` and why is it a problem?

**Answer:**  
When two objects hold `shared_ptr` to each other, their reference counts never reach zero — memory leak! Use `weak_ptr` to break the cycle.

```cpp
struct Node {
    std::shared_ptr<Node> next; // creates cycle!
    int val;
    Node(int v) : val(v) {}
    ~Node() { std::cout << "~Node(" << val << ")\n"; }
};

{
    auto a = std::make_shared<Node>(1);
    auto b = std::make_shared<Node>(2);
    a->next = b;
    b->next = a; // cycle: a→b→a
} // neither destructor called! Memory leak.
```

---

### Q26. How does `shared_ptr` handle inheritance and polymorphism?

**Answer:**  
`shared_ptr` supports implicit upcasting (derived to base). `dynamic_pointer_cast` safely downcasts within the smart pointer framework.

```cpp
struct Base {
    virtual std::string who() const { return "Base"; }
    virtual ~Base() = default;
};
struct Derived : Base {
    std::string who() const override { return "Derived"; }
};

std::shared_ptr<Derived> d = std::make_shared<Derived>();
std::shared_ptr<Base>    b = d;  // upcast: implicit, safe

std::cout << b->who();           // "Derived" — virtual dispatch works
std::cout << b.use_count();      // 2 (both share the same control block)

// Downcast
auto d2 = std::dynamic_pointer_cast<Derived>(b);
if (d2) std::cout << d2->who(); // "Derived"
```

---

### Q27. What are `static_pointer_cast`, `dynamic_pointer_cast`, and `const_pointer_cast`?

**Answer:**  
Smart pointer equivalents of the standard casts, preserving shared ownership.

```cpp
struct A { virtual ~A() = default; };
struct B : A { void hello() { std::cout << "B::hello\n"; } };

auto b = std::make_shared<B>();
std::shared_ptr<A> a = b;                          // implicit upcast

// dynamic: safe downcast (returns nullptr on failure)
auto b2 = std::dynamic_pointer_cast<B>(a);
if (b2) b2->hello(); // B::hello

// static: fast downcast (no runtime check — use when certain)
auto b3 = std::static_pointer_cast<B>(a);
b3->hello(); // B::hello

// const: add/remove const
std::shared_ptr<const B> cb = b;
auto mb = std::const_pointer_cast<B>(cb); // remove const
```

---

### Q28. What is aliasing constructor of `shared_ptr`?

**Answer:**  
The aliasing constructor creates a `shared_ptr` that shares ownership with another `shared_ptr` but points to a different (related) object — for example, a member of the managed object.

```cpp
struct Config {
    int timeout = 30;
    std::string host = "localhost";
};

auto cfg = std::make_shared<Config>();

// Aliasing: shares ownership of cfg but points to cfg->timeout
std::shared_ptr<int> timeoutPtr(cfg, &cfg->timeout);

std::cout << *timeoutPtr;        // 30
std::cout << cfg.use_count();    // 2 (both cfg and timeoutPtr share control block)

cfg.reset();
// cfg is gone, but timeoutPtr still keeps Config alive!
std::cout << *timeoutPtr;        // 30 — still valid
```

---

### Q29. How do you store `shared_ptr` in a class to express shared ownership?

**Answer:**  
Store `shared_ptr<T>` as a member when the class shares ownership of the resource (it co-owns it with others).

```cpp
class Renderer {
    std::shared_ptr<Texture> texture; // shared ownership
public:
    explicit Renderer(std::shared_ptr<Texture> tex)
        : texture(std::move(tex)) {}

    void draw() {
        // use texture
    }
};

auto tex = std::make_shared<Texture>("sprite.png");
Renderer r1(tex); // ref count = 2
Renderer r2(tex); // ref count = 3
// Texture lives as long as any Renderer or tex exists
```

---

### Q30. What is `std::enable_shared_from_this`?

**Answer:**  
A base class that allows an object to create a `shared_ptr` to itself from within a member function, without constructing a new control block. **Never** use `shared_ptr<T>(this)` directly inside a class — it creates a second independent control block (double-free).

```cpp
class Session : public std::enable_shared_from_this<Session> {
public:
    std::shared_ptr<Session> getShared() {
        return shared_from_this(); // safe: uses existing control block
    }

    void asyncWork() {
        auto self = shared_from_this(); // keep alive during async op
        // pass self to callback, safe from dangling
    }
};

auto s = std::make_shared<Session>();
auto s2 = s->getShared();
std::cout << s.use_count(); // 2

// WRONG (without enable_shared_from_this):
// std::shared_ptr<Session> bad(this); // double-free disaster!
```

---

## weak_ptr

---

### Q31. What is `std::weak_ptr`?

**Answer:**  
`weak_ptr` is a non-owning smart pointer that observes a `shared_ptr`-managed object without affecting its reference count. It does not prevent the object from being destroyed. You must `lock()` it to access the object safely.

```cpp
auto shared = std::make_shared<int>(42);
std::weak_ptr<int> weak = shared;

std::cout << shared.use_count(); // 1 (weak doesn't count)

if (auto locked = weak.lock()) { // temporarily shared_ptr
    std::cout << *locked;        // 42
}

shared.reset();                  // object destroyed

if (weak.expired()) {
    std::cout << "Object is gone\n"; // prints this
}
```

---

### Q32. How do you safely access the object through a `weak_ptr`?

**Answer:**  
Use `lock()` to obtain a `shared_ptr`. If the object still exists, `lock()` returns a valid `shared_ptr`; otherwise it returns `nullptr`. This is the **only** safe way to access a `weak_ptr`'s object.

```cpp
std::weak_ptr<std::string> wp;

{
    auto sp = std::make_shared<std::string>("hello");
    wp = sp;

    // Inside scope: lock succeeds
    if (auto p = wp.lock()) {
        std::cout << *p;          // "hello"
        std::cout << p.use_count(); // 2 (sp + p)
    }
} // sp destroyed

// Outside scope: lock returns nullptr
if (auto p = wp.lock()) {
    std::cout << "alive";
} else {
    std::cout << "expired";        // prints this
}
```

---

### Q33. What is `weak_ptr::expired()`?

**Answer:**  
`expired()` returns `true` if the managed object has been destroyed (use count == 0). It's a cheap check but **not sufficient for safe access** — use `lock()` instead to atomically check and access.

```cpp
auto sp = std::make_shared<int>(10);
std::weak_ptr<int> wp = sp;

std::cout << wp.expired(); // false

sp.reset();                // object destroyed

std::cout << wp.expired(); // true

// Don't do this (race condition in multi-threaded code):
// if (!wp.expired()) { auto p = wp.lock(); *p; }  // TOCTOU bug

// Do this (atomic check + access):
if (auto p = wp.lock()) { *p; } // safe
```

---

### Q34. How does `weak_ptr` solve the cyclic reference problem?

**Answer:**  
Replace one of the `shared_ptr` cycle links with a `weak_ptr`. The weak link doesn't contribute to the reference count, allowing the cycle to be broken.

```cpp
struct Node {
    int val;
    std::shared_ptr<Node> next;  // strong link forward
    std::weak_ptr<Node>   prev;  // weak link back — breaks cycle!

    Node(int v) : val(v) {}
    ~Node() { std::cout << "~Node(" << val << ")\n"; }
};

auto a = std::make_shared<Node>(1);
auto b = std::make_shared<Node>(2);

a->next = b;     // a → b (strong)
b->prev = a;     // b → a (weak, no ref count increase)

// When scope ends:
// a use_count = 1 → 0 → ~Node(1)
// b use_count = 1 → 0 → ~Node(2)
// Both destroyed correctly!
```

---

### Q35. What is the weak reference count in the control block?

**Answer:**  
The control block holds two counts:
- **Strong count (use_count):** Number of `shared_ptr` owners. When it hits 0, the object is destroyed.
- **Weak count:** Number of `weak_ptr` observers + 1 (if strong count > 0). When it hits 0, the **control block itself** is freed.

```cpp
auto sp = std::make_shared<int>(5);  // strong=1, weak=1(internal)
std::weak_ptr<int> wp = sp;          // strong=1, weak=2

sp.reset();   // strong=0 → int(5) DESTROYED; weak=1 → control block lives
              // wp can still check expired() via the control block

wp.reset();   // weak=0 → control block FREED
```

---

### Q36. Can you create a `weak_ptr` from a `unique_ptr`?

**Answer:**  
No. `weak_ptr` can only be created from a `shared_ptr` (or another `weak_ptr`) because it needs access to the shared control block. `unique_ptr` has no control block.

```cpp
auto u = std::make_unique<int>(10);
// std::weak_ptr<int> w = u; // Error: no viable conversion

// Must first convert to shared_ptr:
std::shared_ptr<int> s = std::move(u);
std::weak_ptr<int> w = s;  // OK
std::cout << w.expired();  // false
```

---

### Q37. How do you implement an observer pattern with `weak_ptr`?

**Answer:**  
Observers hold `weak_ptr` to the subject. If the subject is destroyed, observers detect this via `expired()` or `lock()` returning `nullptr`.

```cpp
class EventSource {
    std::vector<std::weak_ptr<std::function<void()>>> listeners;
public:
    void subscribe(std::shared_ptr<std::function<void()>> cb) {
        listeners.emplace_back(cb);
    }

    void emit() {
        listeners.erase(
            std::remove_if(listeners.begin(), listeners.end(),
                [](auto& w) { return w.expired(); }),
            listeners.end());

        for (auto& w : listeners)
            if (auto cb = w.lock()) (*cb)();
    }
};

EventSource src;
{
    auto cb = std::make_shared<std::function<void()>>(
        []() { std::cout << "Event!\n"; });
    src.subscribe(cb);
    src.emit(); // "Event!"
} // cb destroyed

src.emit(); // nothing — listener cleaned up
```

---

### Q38. What is the difference between `weak_ptr::lock()` and `weak_ptr::expired()`?

**Answer:**

| Method | Returns | Thread-safe for access? | Use case |
|---|---|---|---|
| `expired()` | `bool` | No (TOCTOU) | Quick check only |
| `lock()` | `shared_ptr<T>` (null if expired) | Yes (atomic) | Safe access |

```cpp
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;

// WRONG: check then use (race condition between expired() and lock())
// if (!wp.expired()) {
//     auto p = wp.lock(); // object might be gone between these two!
// }

// CORRECT: lock atomically checks and returns shared ownership
if (auto p = wp.lock()) {
    std::cout << *p; // guaranteed valid for lifetime of p
}
```

---

## Custom Deleters & Advanced Features

---

### Q39. What is a custom deleter and when do you use it?

**Answer:**  
A custom deleter is a callable invoked instead of `delete` when the smart pointer releases ownership. Useful for non-memory resources (files, sockets, mutexes, C library handles).

```cpp
// Using lambda
auto sqlClose = [](sqlite3* db) {
    std::cout << "Closing DB\n";
    sqlite3_close(db);
};
std::unique_ptr<sqlite3, decltype(sqlClose)> db(openDB(), sqlClose);

// Using function pointer
std::shared_ptr<FILE> file(fopen("log.txt", "w"), fclose);
if (file) fputs("hello\n", file.get());
// fclose called automatically

// Using std::default_delete explicitly
std::unique_ptr<int[], std::default_delete<int[]>> arr(new int[10]);
```

---

### Q40. How does a custom deleter affect the type of `unique_ptr` vs `shared_ptr`?

**Answer:**  
For `unique_ptr`, the deleter type is part of the template type (affects ABI/size). For `shared_ptr`, the deleter is stored in the control block via type erasure — it does **not** appear in the type signature.

```cpp
// unique_ptr: deleter IS part of the type
auto del = [](int* p) { delete p; };
std::unique_ptr<int, decltype(del)> u(new int(1), del);
// Type: unique_ptr<int, lambda>

// shared_ptr: deleter is type-erased (not in type)
std::shared_ptr<int> s(new int(2), [](int* p) { delete p; });
// Type: shared_ptr<int>  ← no deleter in type!

// This means shared_ptr with different deleters can be stored together:
std::vector<std::shared_ptr<int>> vec;
vec.push_back(std::shared_ptr<int>(new int(1), [](int* p){ delete p; }));
vec.push_back(std::shared_ptr<int>(new int(2), [](int* p){ free(p); }));
```

---

### Q41. How do you use smart pointers with C library resources?

**Answer:**  
Wrap C library handles in smart pointers with custom deleters to get RAII management.

```cpp
#include <cstdio>
#include <memory>

// FILE* wrapper
auto openFile(const char* path) {
    return std::unique_ptr<FILE, decltype(&fclose)>(
        fopen(path, "r"), &fclose);
}

// malloc/free wrapper
auto makeCBuffer(size_t n) {
    return std::unique_ptr<char, decltype(&free)>(
        static_cast<char*>(malloc(n)), &free);
}

// OpenSSL EVP_MD_CTX example
// auto ctx = std::unique_ptr<EVP_MD_CTX, decltype(&EVP_MD_CTX_free)>(
//     EVP_MD_CTX_new(), &EVP_MD_CTX_free);

auto f = openFile("data.txt");
if (f) std::cout << "File opened\n";
// fclose called automatically
```

---

### Q42. What is `std::shared_ptr` with `void` (type-erased smart pointer)?

**Answer:**  
`shared_ptr<void>` can hold any type. Because the deleter is stored in the control block at construction time, it will correctly destroy the original object even through `void*`.

```cpp
std::shared_ptr<void> any;

{
    auto s = std::make_shared<std::string>("type-erased");
    any = s; // upcast to void — deleter preserved in control block
    std::cout << any.use_count(); // 2
}

// When any is destroyed, ~string() is called correctly!
// This works because make_shared captured the deleter for string
any.reset(); // "type-erased" string properly destroyed
```

---

### Q43. How do smart pointers interact with exceptions?

**Answer:**  
Smart pointers are exception-safe: if an exception is thrown while a smart pointer is in scope, its destructor still runs (stack unwinding), ensuring no leaks.

```cpp
class Resource {
public:
    Resource()  { std::cout << "Acquired\n"; }
    ~Resource() { std::cout << "Released\n"; }
};

void riskyOperation() {
    auto r = std::make_unique<Resource>(); // Acquired
    // ... work ...
    throw std::runtime_error("Something failed");
    // ~Resource() still called during stack unwinding!
}

try {
    riskyOperation();
} catch (const std::exception& e) {
    std::cout << e.what(); // "Something failed"
}
// Output: Acquired → Released → Something failed
```

---

### Q44. What is `std::owner_less` and when is it used?

**Answer:**  
`owner_less` is a comparator for `shared_ptr`/`weak_ptr` based on ownership (control block address) rather than the stored pointer value. Useful for storing them in ordered containers (`std::map`, `std::set`).

```cpp
#include <map>
#include <memory>

std::map<std::weak_ptr<int>, std::string,
         std::owner_less<std::weak_ptr<int>>> weakMap;

auto s1 = std::make_shared<int>(1);
auto s2 = std::make_shared<int>(2);

weakMap[s1] = "first";
weakMap[s2] = "second";

std::cout << weakMap[s1]; // "first"
std::cout << weakMap[s2]; // "second"

// Also works with shared_ptr as key
std::set<std::shared_ptr<int>, std::owner_less<std::shared_ptr<int>>> pSet;
pSet.insert(s1);
pSet.insert(s2);
```

---

## Best Practices & Pitfalls

---

### Q45. What are the golden rules for choosing the right smart pointer?

**Answer:**

```
Ask: Who owns this resource?
│
├── One owner → unique_ptr
│     └── Transfer ownership? → std::move
│
├── Multiple owners → shared_ptr
│     ├── Need to observe without owning? → weak_ptr
│     └── Callback that outlives caller? → shared_ptr by capture
│
└── No ownership (just use) → raw pointer or reference
      (caller guarantees lifetime)
```

```cpp
// unique_ptr: factory, sole ownership
std::unique_ptr<Widget> createWidget() {
    return std::make_unique<Widget>();
}

// shared_ptr: shared graph node
struct GraphNode {
    std::vector<std::shared_ptr<GraphNode>> neighbors;
};

// weak_ptr: cache that doesn't prevent eviction
std::map<int, std::weak_ptr<Texture>> textureCache;

// Raw pointer/ref: non-owning use
void render(const Widget* w) { /* just uses w, doesn't own */ }
```

---

### Q46. What are the most common smart pointer mistakes?

**Answer:**

```cpp
// MISTAKE 1: Creating shared_ptr from raw 'this'
struct Bad : std::enable_shared_from_this<Bad> {
    std::shared_ptr<Bad> getPtr() {
        // return std::shared_ptr<Bad>(this); // WRONG: double-free!
        return shared_from_this();            // CORRECT
    }
};

// MISTAKE 2: Two independent shared_ptrs from same raw pointer
int* raw = new int(5);
std::shared_ptr<int> p1(raw);
// std::shared_ptr<int> p2(raw); // WRONG: double-free!

// MISTAKE 3: Storing raw pointer from unique_ptr after release
auto u = std::make_unique<int>(10);
int* r = u.release(); // u is now nullptr — YOU must delete r!
// delete r;          // Don't forget!

// MISTAKE 4: Using weak_ptr without locking
std::weak_ptr<int> wp = std::make_shared<int>(1);
// if (!wp.expired()) { *wp.lock() = 2; } // TOCTOU race condition
if (auto p = wp.lock()) { *p = 2; }        // CORRECT

// MISTAKE 5: Cyclic shared_ptr
struct Node { std::shared_ptr<Node> next; }; // memory leak if cycle
// Fix: use weak_ptr for back-links
```

---

### Q47. When should you NOT use smart pointers?

**Answer:**  
Smart pointers are not always the right tool.

```cpp
// 1. Non-owning observation: use raw pointer or reference
void printValue(const int* p) { std::cout << *p; } // just observes

// 2. Stack-allocated objects: no need for heap management
void stackExample() {
    int x = 42;           // plain value, no pointer needed
    std::string s = "hi"; // RAII already built in
}

// 3. C API boundaries: pass raw pointer from get()
void cFunction(int* ptr);
auto sp = std::make_shared<int>(5);
cFunction(sp.get()); // raw pointer, sp still owns it

// 4. Performance-critical hot paths: unique_ptr is fine (zero overhead);
//    shared_ptr has atomic ref count — avoid in tight loops
// 5. Intrusive reference counting: prefer boost::intrusive_ptr or custom
//    when you need the ref count inside the object
```

---

### Q48. How do smart pointers interact with `std::vector` and move semantics?

**Answer:**  
`unique_ptr` elements in vectors must use move operations. `shared_ptr` elements can be copied. Both work correctly with `emplace_back`, `std::move`, and reallocation.

```cpp
std::vector<std::unique_ptr<int>> uVec;

uVec.push_back(std::make_unique<int>(1));
uVec.emplace_back(std::make_unique<int>(2));

auto u = std::make_unique<int>(3);
uVec.push_back(std::move(u)); // u is now nullptr

// Move the whole vector (cheap)
auto moved = std::move(uVec); // uVec is now empty
std::cout << moved.size();    // 3

// Sort by dereferenced value
std::sort(moved.begin(), moved.end(),
    [](const auto& a, const auto& b) { return *a < *b; });
```

---

### Q49. What happens when you pass a smart pointer to a function? What are the conventions?

**Answer:**

| Parameter Type | Meaning |
|---|---|
| `unique_ptr<T>` by value | Sink: function takes ownership |
| `unique_ptr<T>&` | Function may reset/reseat the pointer |
| `const unique_ptr<T>&` | Awkward — prefer `T*` or `T&` |
| `shared_ptr<T>` by value | Function shares ownership |
| `shared_ptr<T>&` | Function may reseat the shared_ptr |
| `T*` or `T&` | Non-owning use (caller owns) |

```cpp
// Sink: takes ownership
void sink(std::unique_ptr<Widget> w) { /* owns w */ }
sink(std::make_unique<Widget>()); // OK
// sink(ptr); // must move: sink(std::move(ptr));

// Non-owning use — prefer this for most function params
void use(Widget& w) { w.doWork(); }
void use(Widget* w) { if(w) w->doWork(); }

auto p = std::make_unique<Widget>();
use(*p);    // dereference
use(p.get()); // raw ptr

// Share ownership
void share(std::shared_ptr<Widget> w) { /* co-owns */ }
auto sp = std::make_shared<Widget>();
share(sp); // ref count incremented
```

---

### Q50. How do smart pointers support the Pimpl idiom?

**Answer:**  
The Pimpl (Pointer to IMPLementation) idiom hides implementation details in a forward-declared class. `unique_ptr` is the modern way to manage the Pimpl, giving automatic cleanup while keeping the destructor out of the header.

```cpp
// widget.h — public header
class Widget {
public:
    Widget();
    ~Widget();               // declared but defined in .cpp
    void doWork();

private:
    struct Impl;             // forward declaration
    std::unique_ptr<Impl> pImpl; // unique_ptr to incomplete type
};

// widget.cpp — implementation
struct Widget::Impl {
    std::string name = "Widget";
    int value = 0;
    void work() { std::cout << name << " working\n"; }
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() = default;   // defined here: Impl is complete
void Widget::doWork() { pImpl->work(); }

// Usage:
Widget w;
w.doWork(); // "Widget working"
// ~Widget() → ~Impl() called automatically
```

---

## Quick Reference Summary

| Smart Pointer | Ownership | Copyable | Moveable | Overhead | Best For |
|---|---|---|---|---|---|
| `unique_ptr<T>` | Exclusive | ✗ | ✓ | Zero | Default choice, factory returns |
| `shared_ptr<T>` | Shared | ✓ | ✓ | Ref count (atomic) | Shared graphs, callbacks |
| `weak_ptr<T>` | None | ✓ | ✓ | Weak count only | Break cycles, caches, observers |

### Key Functions at a Glance

| Function | Description |
|---|---|
| `make_unique<T>(args...)` | Create `unique_ptr` (C++14) |
| `make_shared<T>(args...)` | Create `shared_ptr` (one allocation) |
| `ptr.get()` | Raw pointer (no ownership transfer) |
| `ptr.reset()` | Release and optionally replace |
| `ptr.release()` | Relinquish ownership (unique_ptr only) |
| `ptr.use_count()` | Ref count (shared_ptr) |
| `wp.lock()` | Try to get shared_ptr from weak_ptr |
| `wp.expired()` | Check if managed object is gone |
| `std::move(ptr)` | Transfer ownership of unique_ptr |
| `shared_from_this()` | Safe self shared_ptr (enable_shared_from_this) |

---

*Master smart pointers and you master modern C++ resource management!*
