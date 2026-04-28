## Memory ManagementRaw pointers, smart pointers (unique_ptr, shared_ptr, weak_ptr), memory leaks, dangling pointers, double-free, RAII.
Here are **50 tricky C++ interview questions** focusing on **Memory Management** — covering raw pointers, `unique_ptr`, `shared_ptr`, `weak_ptr`, memory leaks, dangling pointers, double-free, RAII, and more — with answers and code examples.

---

## 1. What is RAII? Why is it fundamental in C++?

**Answer:**  
Resource Acquisition Is Initialization — resource lifetime tied to object lifetime. Constructor acquires, destructor releases.

```cpp
class File {
    FILE* f;
public:
    File(const char* name) : f(fopen(name, "r")) {}
    ~File() { if (f) fclose(f); }
}; // guaranteed release even on exception
```

---

## 2. What is a dangling pointer? How does it happen?

**Answer:**  
Pointer pointing to already deleted memory. Happens after `delete` without setting to `nullptr`.

```cpp
int* p = new int(5);
delete p;
// p is dangling
*p = 10; // UB
```

---

## 3. What is a double-free? Why dangerous?

**Answer:**  
Deleting same memory twice — heap corruption, UB, security vulnerability.

```cpp
int* p = new int(5);
delete p;
delete p; // double-free → crash or corruption
```

---

## 4. How do smart pointers prevent double-free?

**Answer:**  
Ownership semantics: `unique_ptr` move-only, `shared_ptr` ref-count.

```cpp
std::unique_ptr<int> p1(new int(5));
std::unique_ptr<int> p2 = std::move(p1); // p1 now null
// no double-free
```

---

## 5. What is a memory leak? How to detect?

**Answer:**  
Allocated memory never freed — process memory grows. Detect with Valgrind, ASan, leak sanitizer.

```cpp
void leak() { int* p = new int(10); } // lost memory
```

---

## 6. What is `std::unique_ptr`? When to use?

**Answer:**  
Exclusive ownership, no copy, lightweight.

```cpp
auto p = std::make_unique<int>(42);
// std::unique_ptr<int> p2 = p; // error
```

**Use:** Factory functions, PIMPL, polymorphic ownership.

---

## 7. What is `std::shared_ptr` overhead?

**Answer:**  
Two pointers (object + control block), atomic ref-count operations.

```cpp
sizeof(std::shared_ptr<int>) // 16 bytes on 64-bit (two pointers)
```

---

## 8. What is `std::weak_ptr`? Why not just raw pointer?

**Answer:**  
Non-owning observer of `shared_ptr` — avoids dangling, prevents circular references.

```cpp
std::shared_ptr<int> sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;
if (auto locked = wp.lock()) { /* use locked */ }
```

---

## 9. What is circular reference? Show example with `shared_ptr`.

**Answer:**  
Two objects hold each other's `shared_ptr` → never deleted.

```cpp
struct Node {
    std::shared_ptr<Node> next;
    ~Node() { cout << "deleted\n"; }
};
auto n1 = std::make_shared<Node>();
auto n2 = std::make_shared<Node>();
n1->next = n2;
n2->next = n1; // circular → leak
```

**Fix:** Use `weak_ptr` in one direction.

---

## 10. How to break circular reference with `shared_ptr`?

**Answer:**  
Use `weak_ptr` for back references.

```cpp
struct Node {
    std::weak_ptr<Node> next; // breaks cycle
};
```

---

## 11. What happens if you `delete` a raw pointer managed by `shared_ptr`?

**Answer:**  
Double-free — `shared_ptr` will delete again.

```cpp
int* raw = new int(5);
std::shared_ptr<int> sp(raw);
delete raw; // UB: second delete in sp destructor
```

---

## 12. Is this safe? `std::shared_ptr<T>(new T)` vs `std::make_shared<T>()`?

**Answer:**  
`make_shared` is safer (exception-safe) and faster (single allocation).

```cpp
f(std::shared_ptr<T>(new T), g()); // may leak if g() throws
f(std::make_shared<T>(), g()); // safe
```

---

## 13. Can two `shared_ptr` own same memory from two raw pointers?

**Answer:**  
No — two separate control blocks → double-free.

```cpp
int* raw = new int(5);
std::shared_ptr<int> sp1(raw);
std::shared_ptr<int> sp2(raw); // WRONG: double-free
```

**Fix:** `sp2 = sp1;`

---

## 14. What is `std::unique_ptr::release()`? When use?

**Answer:**  
Releases ownership, returns raw pointer, doesn't delete.

```cpp
auto p = std::make_unique<int>(10);
int* raw = p.release(); // p now null
delete raw; // manual cleanup
```

---

## 15. What is `std::unique_ptr::reset()` vs `release()`?

**Answer:**  
`reset()` deletes owned object (optionally takes new one). `release()` gives ownership without delete.

```cpp
p.reset(); // deletes
p.reset(new int(20)); // deletes old, takes new
```

---

## 16. Can you put `unique_ptr` in a container?

**Answer:**  
Yes — move-only.

```cpp
std::vector<std::unique_ptr<int>> vec;
vec.push_back(std::make_unique<int>(5));
vec.push_back(std::move(p));
```

---

## 17. How to pass `unique_ptr` to a function?

**Answer:**  
By value (move) or by reference (no transfer).

```cpp
void take(std::unique_ptr<int> p); // transfers ownership
void observe(const std::unique_ptr<int>& p); // no transfer
```

---

## 18. What is `std::make_unique` limitation vs `std::make_shared`?

**Answer:**  
`make_unique` cannot be used with custom deleters. `make_shared` has control block tied to object.

---

## 19. What is the "enable_shared_from_this" pattern?

**Answer:**  
Class that safely returns `shared_ptr` to `this`.

```cpp
class A : public std::enable_shared_from_this<A> {
public:
    std::shared_ptr<A> getptr() { return shared_from_this(); }
};
auto sp = std::make_shared<A>();
auto sp2 = sp->getptr(); // OK, same control block
```

---

## 20. What happens if you call `shared_from_this()` before object is owned by `shared_ptr`?

**Answer:**  
Throws `std::bad_weak_ptr`.

```cpp
A a;
auto sp = a.getptr(); // throws
```

---

## 21. Can `weak_ptr` expire? How to check?

**Answer:**  
Yes — when all `shared_ptr` destroyed. Use `expired()`, `lock()`, or `use_count()`.

```cpp
if (wp.expired()) cout << "gone";
```

---

## 22. What is the difference between `weak_ptr::lock()` and converting `shared_ptr`?

**Answer:**  
`lock()` creates new `shared_ptr` if object alive; converting from expired `weak_ptr` throws.

```cpp
std::shared_ptr<int> sp = wp.lock(); // returns null if expired
std::shared_ptr<int> sp2(wp); // throws bad_weak_ptr if expired
```

---

## 23. How to implement a custom deleter for `unique_ptr`?

**Answer:**  
Pass function/lambda as second template parameter.

```cpp
auto deleter = [](FILE* f) { if (f) fclose(f); };
std::unique_ptr<FILE, decltype(deleter)> fp(fopen("x.txt", "r"), deleter);
```

---

## 24. What is the performance cost of `shared_ptr` vs raw pointer?

**Answer:**  
+25–50% overhead: atomic increments/decrements on copy/destroy, double indirection.

---

## 25. What is `std::shared_ptr<T[]>?` (C++17)

**Answer:**  
Specialization for arrays — calls `delete[]`.

```cpp
std::shared_ptr<int[]> sp(new int[10]);
sp[0] = 5; // OK
```

---

## 26. Can you use `weak_ptr` with `unique_ptr`?

**Answer:**  
No — `weak_ptr` only works with `shared_ptr`.

---

## 27. What is the rule of zero/five?

**Answer:**  
Rule of zero: use smart pointers → no manual destructor/copy/move.  
Rule of five: if you define one, define all (destructor, copy/move ctor/assign).

```cpp
class RuleOfZero {
    std::unique_ptr<int> p; // automatic management
}; // no user-defined special members
```

---

## 28. What is a `std::auto_ptr`? Why deprecated?

**Answer:**  
Pre-C++11 smart pointer — copy actually moves (broken semantics).

```cpp
std::auto_ptr<int> p1(new int(5));
std::auto_ptr<int> p2 = p1; // p1 now null! (unexpected)
```

---

## 29. What is aliasing constructor of `shared_ptr`?

**Answer:**  
Creates `shared_ptr` sharing ownership but pointing to different object.

```cpp
struct Pair { int a, b; };
auto sp = std::make_shared<Pair>();
std::shared_ptr<int> asp(sp, &sp->a); // shares ownership, points to a
```

---

## 30. How to check if `unique_ptr` is null?

**Answer:**  
`if (!p)` or `if (p == nullptr)` or `if (p.get() == nullptr)`.

---

## 31. What is `std::owner_less` used for?

**Answer:**  
Compare `shared_ptr`/`weak_ptr` by ownership (not value) for associative containers.

```cpp
std::set<std::weak_ptr<int>, std::owner_less<std::weak_ptr<int>>> s;
```

---

## 32. Can you have a `shared_ptr<void>`? Why?

**Answer:**  
Yes — type-erased ownership, but must cast back correctly.

```cpp
std::shared_ptr<void> sp = std::make_shared<int>(42);
int* p = static_cast<int*>(sp.get()); // ok
```

---

## 33. What is the `std::unique_ptr` default deleter?

**Answer:**  
Calls `delete` on pointer (or `delete[]` for array specialization).

---

## 34. What happens if you `reset()` a `shared_ptr` with multiple owners?

**Answer:**  
Decrements ref-count; only last owner deletes object.

```cpp
auto p1 = std::make_shared<int>(5);
auto p2 = p1;
p1.reset(); // ref-count: 2→1, no delete
p2.reset(); // ref-count: 1→0, deletes
```

---

## 35. How to create `shared_ptr` from `unique_ptr`?

**Answer:**  
Move construction.

```cpp
std::unique_ptr<int> up(new int(5));
std::shared_ptr<int> sp = std::move(up);
```

---

## 36. What is the cost of `shared_ptr` move vs copy?

**Answer:**  
Move is cheap (just pointers, no atomic ops). Copy needs atomic increment.

---

## 37. Is `shared_ptr` thread-safe?

**Answer:**  
Control block (ref-count) is thread-safe. Pointed object is not.

---

## 38. Can you call `delete` on `this`? Is it RAII-compliant?

**Answer:**  
Yes — but dangerous. Use only in `release()`/recycle patterns, not RAII.

```cpp
class A { public: void release() { delete this; } }; // anti-pattern
```

---

## 39. What is the "static initialization order fiasco" with smart pointers?

**Answer:**  
Global smart pointers may access uninitialized objects. Use `constinit` or Meyers singleton.

---

## 40. How to avoid memory leaks in exception-prone code with raw pointers?

**Answer:**  
Don't — use smart pointers or RAII wrappers.

```cpp
void risky() {
    int* p = new int(5);
    throw 1; // leak
    delete p;
}
```

**Fix:** `auto p = std::unique_ptr<int>(new int(5));`

---

## 41. What is the difference between `new` and `operator new`?

**Answer:**  
`new` = `operator new` (allocation) + constructor call. `operator new` only allocates memory.

```cpp
void* raw = ::operator new(sizeof(int)); // just allocates
int* p = new(raw) int(5); // placement new
```

---

## 42. Can you use `make_shared` with custom deleter?

**Answer:**  
No — use `shared_ptr` constructor directly.

```cpp
std::shared_ptr<int> sp(new int(5), [](int* p){ delete p; });
```

---

## 43. What is `std::allocate_shared`?

**Answer:**  
`make_shared` with custom allocator.

```cpp
std::allocate_shared<int>(std::allocator<int>(), 5);
```

---

## 44. What is a memory pool? How relates to smart pointers?

**Answer:**  
Pre-allocated block for fast allocations. Custom deleters return to pool.

```cpp
auto pool_deleter = [&pool](int* p) { pool.deallocate(p); };
std::unique_ptr<int, decltype(pool_deleter)> p(pool.allocate(), pool_deleter);
```

---

## 45. What is `std::out_of_range` vs memory leak from `vector::at()`?

**Answer:**  
`at()` throws, no leak if using RAII containers. Raw pointers may leak.

---

## 46. How to implement a simple intrusive pointer?

**Answer:**  
Reference count inside object — boost::intrusive_ptr.

---

## 47. What is a hazard pointer?

**Answer:**  
Lock-free memory reclamation technique for concurrent data structures.

---

## 48. What is `std::uninitialized_copy` danger?

**Answer:**  
Copies to uninitialized memory — must manually destroy if exception.

```cpp
std::uninitialized_copy(src.begin(), src.end(), dest_raw);
// if exception, must destroy constructed elements
```

---

## 49. Why prefer `std::make_unique` over raw `new`?

**Answer:**  
Exception safety, fewer characters, no duplicate type.

```cpp
f(std::make_unique<A>(), g()); // safe
f(std::unique_ptr<A>(new A()), g()); // may leak if g() throws
```

---

## 50. What is the final wisdom about raw pointers in modern C++?

**Answer:**  
Use raw pointers only for non-owning observations. Never `new`/`delete` manually. Prefer containers and smart pointers.

```cpp
// Owning: smart pointer or container
auto p = std::make_unique<Widget>();
// Observing: raw pointer or reference
void use(Widget* w); // w not owned
```

---
