Here are **60 tricky C++ interview questions and answers** focused on **Memory Management**, categorized by topic, with examples.

---

## **Raw Pointers & Basic Memory Management**

### 1. **What is the difference between `malloc`/`free` and `new`/`delete`?**
`malloc`/`free` are C functions – no constructors/destructors. `new`/`delete` call constructors/destructors.  
**Trick:** `new` returns exact type; `malloc` returns `void*`.

```cpp
int* p1 = (int*)malloc(sizeof(int)); // No constructor
int* p2 = new int(42);               // Constructor called
free(p1);
delete p2;
```

### 2. **What happens if you `delete` the same pointer twice?**
**Undefined behavior** – usually crash (double-free).  
```cpp
int* p = new int(5);
delete p;
delete p; // BOOM: double free
```

### 3. **What is a dangling pointer? Example.**
Pointer to memory that has been freed.  
```cpp
int* p = new int(10);
delete p;
*p = 20; // Dangling pointer – undefined behavior
```

### 4. **What is a memory leak?**
Lost reference to allocated memory – cannot delete.  
```cpp
void leaky() {
    int* p = new int(100);
    // no delete – memory leak when function exits
}
```

### 5. **Can you use `free()` to delete memory allocated by `new`? Why?**
No – undefined behavior. `new` might use different allocator (operator new vs malloc).  
```cpp
int* p = new int;
free(p); // WRONG – disaster potential
```

### 6. **What is `operator new` vs `new` operator?**  
`new` operator = language feature (allocates + constructs). `operator new` = function (allocates raw memory).  
Similarly `delete` operator vs `operator delete`.

### 7. **Why should you never mix `new[]` and `delete` (without `[]`)?**  
Undefined behavior – wrong number of destructors called.  
```cpp
int* arr = new int[10];
delete arr; // Only destructor for first element? No – UB.
// Correct: delete[] arr;
```

### 8. **What does `placement new` do?**  
Constructs object in pre-allocated memory buffer (no allocation).  
```cpp
char buffer[sizeof(MyClass)];
MyClass* obj = new(buffer) MyClass(); // placement new
obj->~MyClass(); // Manual destructor call needed
```

### 9. **Is it safe to return a pointer to a local variable?**  
No – local variable destroyed on function exit → dangling pointer.  
```cpp
int* bad() {
    int x = 5;
    return &x; // Dangling immediately
}
```

### 10. **What’s wrong with this code?**  
```cpp
int* p = new int(10);
p = new int(20);
delete p;
```
**Answer:** Memory leak of first `int(10)`.

---

## **RAII & Rule of Three/Five**

### 11. **What is RAII?**  
Resource Acquisition Is Initialization – resources (memory, file handles) tied to object lifetime. Constructor acquires, destructor releases.

### 12. **Why RAII prevents memory leaks?**  
If exception thrown, destructors run automatically → guaranteed cleanup.

### 13. **What is Rule of Three?**  
If you define destructor, copy constructor, or copy assignment – define all three (for deep copy of raw pointer members).

### 14. **What is Rule of Five?**  
Rule of Three + move constructor + move assignment (C++11).

### 15. **Example of violating Rule of Three leading to double-free.**  
```cpp
class Bad {
    int* p;
public:
    Bad() : p(new int(10)) {}
    ~Bad() { delete p; }
    // No copy constructor – shallow copy → double delete
};
Bad a;
Bad b = a; // Crash on destruction
```

### 16. **How to fix above?**  
Delete copy operations or implement deep copy (or use `unique_ptr`).

---

## **`unique_ptr`**

### 17. **Why `unique_ptr` is “unique”?**  
Non-copyable – only one owner at a time. Ownership transfer via move.

### 18. **What happens when `unique_ptr` goes out of scope?**  
Destructor calls `delete` on owned pointer automatically.

### 19. **How to transfer ownership of `unique_ptr`?**  
`std::move`.  
```cpp
std::unique_ptr<int> a = std::make_unique<int>(5);
std::unique_ptr<int> b = std::move(a); // a now null
```

### 20. **Can you put `unique_ptr` in a `vector`?**  
Yes, but can’t copy – must `std::move`.  
```cpp
std::vector<std::unique_ptr<int>> vec;
vec.push_back(std::make_unique<int>(10)); // OK
```

### 21. **What is `make_unique` advantage over `new`?**  
Exception safety (prevents leak if other allocation throws), fewer `new`/`delete` in user code, cleaner.

```cpp
// Less safe (if bar() throws, leak)
foo(std::unique_ptr<int>(new int(5)), bar()); 
// Safe
foo(std::make_unique<int>(5), bar());
```

### 22. **How to use custom deleter with `unique_ptr`?**  
```cpp
auto deleter = [](FILE* f) { fclose(f); };
std::unique_ptr<FILE, decltype(deleter)> fptr(fopen("x.txt","r"), deleter);
```

### 23. **What is `unique_ptr` size compared to raw pointer?**  
Same size (one pointer) if custom deleter is stateless (lambda without capture). With function pointer deleter, size increases.

---

## **`shared_ptr`**

### 24. **How does `shared_ptr` know when to delete?**  
Reference counting – control block tracks strong refs.

### 25. **What is control block in `shared_ptr`?**  
Separate heap-allocated object storing: strong count, weak count, custom deleter, allocator.

### 26. **Is `shared_ptr` thread-safe?**  
Control block updates are atomic (reference count operations thread-safe). *Pointed object is not automatically thread-safe.*

### 27. **What is cyclic reference problem with `shared_ptr`?**  
Two objects hold `shared_ptr` to each other → never delete (memory leak).

```cpp
struct Node {
    std::shared_ptr<Node> next;
};
auto a = std::make_shared<Node>();
auto b = std::make_shared<Node>();
a->next = b;
b->next = a; // Cycle – never freed
```

### 28. **How to break the cycle?**  
Use `weak_ptr` for one direction.

### 29. **What is `make_shared` advantage over `new` + `shared_ptr`?**  
One allocation (object + control block) instead of two → better cache locality, less fragmentation.

### 30. **What is `shared_ptr` overhead?**  
Control block (usually 2 words for strong+weak count) plus pointer (total ~3x size of raw pointer).

### 31. **When is `shared_ptr` dangerous?**  
Cycles, unintended prolonging of lifetime, performance hit from atomic increments.

### 32. **Can `shared_ptr<T>` point to `T[]`?**  
Yes in C++17 (calls `delete[]`), or custom deleter. Prefer `std::shared_ptr<std::vector<T>>`.

---

## **`weak_ptr`**

### 33. **What is `weak_ptr` used for?**  
Observe object owned by `shared_ptr` without extending lifetime. Break cycles, implement caches.

### 34. **How to access object from `weak_ptr`?**  
`.lock()` returns `shared_ptr` (null if expired).  
```cpp
std::weak_ptr<int> wp;
{
    auto sp = std::make_shared<int>(42);
    wp = sp;
    auto sp2 = wp.lock(); // valid
}
auto sp3 = wp.lock(); // null
```

### 35. **What is `expired()` method?**  
Checks if object already deleted.

### 36. **Can `weak_ptr` cause memory leak?**  
No – weak count doesn’t prevent object deletion. Control block lives while `weak_ptr` exists though (small overhead).

### 37. **Implement thread-safe cache with `weak_ptr`.**  
Store `weak_ptr`s; when requested, try `.lock()` – if null, reload.

---

## **Memory Leaks & Detection**

### 38. **How to detect memory leaks in C++?**  
Valgrind, AddressSanitizer (ASan), Visual Studio CRT debug heap, custom allocator tracking.

### 39. **Can an exception cause memory leak?**  
Yes – if `new` succeeds but later constructor throws before pointer stored in smart pointer.  
```cpp
void risky() {
    int* p = new int(5);
    throw std::runtime_error("oops");
    delete p; // never reached → leak
}
// Fix: use unique_ptr
```

### 40. **What is “leak at exit” in C++?**  
Memory not freed when program ends – OS reclaims, but indicates bad design.

### 41. **What is memory fragmentation?**  
Free memory split into small unusable chunks. Frequent small allocations/deletions cause it.

### 42. **How does `std::allocator` help?**  
Splits allocation from construction – useful for containers to manage memory pools.

---

## **Dangling Pointers & Double-Free**

### 43. **How to avoid dangling pointers?**  
Set pointer to `nullptr` after `delete`, or use smart pointers.

### 44. **What happens if you dereference `nullptr`?**  
Segmentation fault (undefined behavior, but usually crash).

### 45. **What is double-free detection?**  
AddressSanitizer catches double-free. Otherwise silent memory corruption.

### 46. **Can `shared_ptr` have double-free?**  
No – reference counting prevents. But two independent `shared_ptr`s created from same raw pointer cause double-free.  
```cpp
int* raw = new int(10);
std::shared_ptr<int> sp1(raw);
std::shared_ptr<int> sp2(raw); // Both destroy → double-free
```

### 47. **What’s wrong with `return *this` assignment chain in allocation?**  
Not directly, but raw pointer class without rule-of-three is dangerous.

---

## **Advanced / Tricky Scenarios**

### 48. **What is `std::enable_shared_from_this`?**  
Allows object to get a `shared_ptr` to itself from inside member function.  
```cpp
class A : public std::enable_shared_from_this<A> {
public:
    std::shared_ptr<A> getptr() { return shared_from_this(); }
};
auto a = std::make_shared<A>();
auto b = a->getptr(); // OK – both share ownership
```

### 49. **What if you call `shared_from_this()` on stack object?**  
Throws `std::bad_weak_ptr`.

### 50. **What is copy-on-write (COW) and its memory management?**  
Shared immutable state until write → needs custom reference counting (rare now due to C++11 move semantics).

### 51. **How to allocate aligned memory in C++?**  
`std::aligned_alloc` or `operator new` with align argument (C++17).

### 52. **What is `std::launder` for?**  
After placement new into same storage, prevents UB when accessing original pointer. Rare, but needed in some constexpr or union cases.

### 53. **What is the “memory barrier” problem with `shared_ptr`?**  
`shared_ptr` destructor uses atomic decrement with release barrier; copy uses acquire. Correct but not lock-free in all implementations.

### 54. **Can you replace global `operator new`?**  
Yes – link with custom implementation. Dangerous – must be compliant.

### 55. **Why is `delete this;` valid?**  
Possible only if object allocated on heap, and no access to members after.  
```cpp
class SelfDestruct {
public:
    void die() { delete this; }
};
```

### 56. **What is “static initialization order fiasco” related to memory?**  
Static objects in different translation units constructed in undefined order → accessing one from another’s constructor may crash.

### 57. **How does `std::pmr::monotonic_buffer_resource` work?**  
Allocates from a buffer, no individual frees – reset whole buffer at once. Fast for short-lived allocations.

### 58. **Difference between `std::unique_ptr` with custom deleter and `std::shared_ptr` with custom deleter?**  
`unique_ptr` deleter is part of type (important for size). `shared_ptr` deleter erased (type-erased) – same `shared_ptr<T>` regardless of deleter type.

### 59. **Can you convert `shared_ptr<T>` to `shared_ptr<const T>`?**  
Yes – implicit conversion.  
```cpp
std::shared_ptr<int> sp = std::make_shared<int>(5);
std::shared_ptr<const int> csp = sp; // OK
```

### 60. **What is the “indestructible base” problem and how to solve with `shared_ptr`?**  
If base has private/protected destructor, can still use `shared_ptr` with custom deleter or `shared_ptr` allows no-op destroy if `enable_shared_from_this` and appropriate.

```cpp
struct NonDeletable {
    ~NonDeletable() = default; // private in real case
    static void destroy(NonDeletable* p) { /* custom delete */ }
};
auto p = std::shared_ptr<NonDeletable>(new NonDeletable(), &NonDeletable::destroy);
```

---

These 60 questions cover **everything** from basic raw pointer pitfalls to advanced `weak_ptr` cycles and `pmr` allocators. Memorize the examples – they are the “trick” part that interviewers love.