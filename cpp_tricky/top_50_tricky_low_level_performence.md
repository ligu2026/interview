Here are **50 tricky C++ interview questions** focusing on **Low-Level / System & Performance** — covering cache efficiency, false sharing, alignment, inlining, virtual call overhead, object slicing, ABI basics, and more — with answers and code examples.

---

## 1. What is cache locality? Why does it matter?

**Answer:**  
Locality (spatial/temporal) determines how well code utilizes CPU caches. Good locality = fewer cache misses.

```cpp
// Bad (column-major on row-major array)
for (int j = 0; j < N; j++)
    for (int i = 0; i < N; i++) sum += a[i][j];

// Good (row-major)
for (int i = 0; i < N; i++)
    for (int j = 0; j < N; j++) sum += a[i][j];
```

---

## 2. What is false sharing? How to avoid?

**Answer:**  
Threads on different cores modify different variables in same cache line → expensive cache coherence traffic.

**Fix:** Pad or align variables.

```cpp
struct alignas(64) Counter {
    int value;
    char padding[60]; // 64 - 4 bytes
};
```

---

## 3. What is `alignas` and `alignof`?

**Answer:**  
`alignas` sets alignment; `alignof` queries alignment requirement.

```cpp
struct alignas(16) Vec4 { float x,y,z,w; };
cout << alignof(Vec4); // 16
```

---

## 4. What is the difference between `inline` and macros?

**Answer:**  
`inline` is a compiler hint (function); macro is preprocessor text substitution.

```cpp
#define SQUARE(x) ((x)*(x)) // dangerous: SQUARE(++i)
inline int square(int x) { return x*x; } // safe
```

---

## 5. Does `inline` guarantee inlining?

**Answer:**  
No — compiler can ignore. Modern compilers inline without `inline` keyword.

---

## 6. What is a vtable? Cost of virtual function call?

**Answer:**  
vtable = virtual function table. Cost: extra indirection, prevents inlining.

```cpp
// Without virtual: call at compile-time (cheap)
// With virtual: call through vptr (2-3x slower, no inlining)
```

---

## 7. How to avoid virtual call overhead?

**Answer:**  
- CRTP (static polymorphism)
- `final` keyword helps devirtualization
- `-fdevirtualize` compiler flag

```cpp
template<typename Derived> class Base {
    void call() { static_cast<Derived*>(this)->impl(); }
};
```

---

## 8. What is object slicing? Performance impact?

**Answer:**  
Slicing discards derived data — silent logical error, not performance gain.

```cpp
Derived d; Base b = d; // slices: Derived::extra data lost
```

**Prevention:** Use pointers/references.

---

## 9. What is ABI? Why does it matter for C++?

**Answer:**  
Application Binary Interface — low-level details (vtable layout, name mangling, calling conventions). Incompatible ABI = libraries can't mix.

---

## 10. What C++ features break ABI?

**Answer:**  
- Adding virtual functions
- Changing member order
- Changing template definitions
- `std::string` implementation (small string optimization vs copy-on-write)

---

## 11. What is the difference between `malloc`/`free` and `new`/`delete`?

**Answer:**  
`new` calls constructor, `delete` calls destructor. `malloc`/`free` don't.

---

## 12. What is placement `new`?

**Answer:**  
Constructs object in pre-allocated memory.

```cpp
char buffer[sizeof(A)];
A* p = new(buffer) A(); // no allocation
p->~A(); // manual destructor call
```

---

## 13. What is memory alignment? Why needed?

**Answer:**  
CPU accesses aligned addresses faster; unaligned may trap or be slow.

```cpp
alignas(8) int x; // aligns x to 8 bytes
```

---

## 14. What is a cache line? Typical size?

**Answer:**  
Smallest unit of cache transfer. Typically 64 bytes on x86-64.

---

## 15. How to measure cache misses?

**Answer:**  
`perf stat -e cache-misses ./program` or Intel VTune, `std::chrono` with large data.

---

## 16. What is the performance difference between `vector<int>` and `list<int>`?

**Answer:**  
`vector`: contiguous cache-friendly; `list`: node-based (cache-unfriendly).

```cpp
// vector: iterate ~10x faster due to prefetching
```

---

## 17. What is branch prediction? Why `likely`/`unlikely`?

**Answer:**  
CPU predicts branch direction; hints can help.

```cpp
if (__builtin_expect(cond, 0)) { /* unlikely */ }
// C++20: [[likely]] [[unlikely]]
```

---

## 18. What is a tight loop? Why optimize?

**Answer:**  
Inner loop executed most; keeping it small and cache-local is critical.

---

## 19. What is loop unrolling? When useful?

**Answer:**  
Expands loop body to reduce branch overhead.

```cpp
// Compiler does: -funroll-loops
for (int i = 0; i < 4; i++) a[i] += b[i]; // manually unrolled
```

---

## 20. What is the performance cost of `std::function`?

**Answer:**  
Type erasure → heap allocation (for large callables) + virtual call overhead.

**Use template parameters instead when possible.**

```cpp
template<typename F> void call(F f) { f(); } // zero overhead
```

---

## 21. What is RVO and NRVO?

**Answer:**  
Return Value Optimization / Named RVO — compiler elides copy, constructs return value directly.

```cpp
A f() { A a; return a; } // NRVO: no copy
```

---

## 22. When does RVO not happen?

**Answer:**  
- Multiple return paths returning different named objects
- Conditional returns
- Compiler disabled `-fno-elide-constructors`

---

## 23. What is the cost of `std::shared_ptr`?

**Answer:**  
Two pointers (object + control block) + atomic reference count increment/decrement.

---

## 24. How to reduce `shared_ptr` overhead?

**Answer:**  
- Use `std::make_shared` (single allocation)
- Use `std::move` to transfer
- Use `unique_ptr` when possible

---

## 25. What is a `restrict` pointer?

**Answer:**  
Promise no alias — helps optimization (C99, some compilers `__restrict` in C++).

```cpp
void f(int* __restrict a, int* __restrict b) { *a += *b; }
```

---

## 26. What is the performance cost of `dynamic_cast`?

**Answer:**  
String comparison or traversal of inheritance tree — slow. O(1) if single inheritance with vtable.

**Avoid in performance-critical code.**

---

## 27. What is a `std::launder`?

**Answer:**  
Tells compiler object life started at a different address (C++17).

```cpp
alignas(T) char buf[sizeof(T)];
T* p = new(buf) T();
p->~T();
T* q = new(buf) T(); // UB until std::launder
T* r = std::launder(q);
```

---

## 28. What is strict aliasing rule?

**Answer:**  
Cannot dereference a pointer of type T to object of a different type (except `char*`). Violations = UB.

```cpp
int x = 10;
float* f = reinterpret_cast<float*>(&x);
*f = 3.14; // UB - strict aliasing violation
```

---

## 29. How to bypass strict aliasing safely?

**Answer:**  
`std::memcpy` or `std::bit_cast` (C++20).

```cpp
std::bit_cast<float>(x);
```

---

## 30. What is the cost of exception handling?

**Answer:**  
Zero-cost in non-throwing path (table-driven); 10-20x overhead on throw.

---

## 31. When should exceptions NOT be used?

**Answer:**  
Real-time systems, game engines, performance-critical hot paths. Use error codes or `std::optional`.

---

## 32. What is a `noexcept` function? Performance benefit?

**Answer:**  
`noexcept` tells compiler function won't throw — allows optimization (move semantics benefit).

```cpp
A(A&&) noexcept; // vector uses move not copy
```

---

## 33. What is the cost of `std::atomic` compared to mutex?

**Answer:**  
`atomic` = lock-free (often), mutex = OS synchronization (slower). Measure.

---

## 34. What is memory order `relaxed`? When use?

**Answer:**  
No ordering guarantees — fastest atomic. Use for counters, not flags.

```cpp
counter.fetch_add(1, std::memory_order_relaxed);
```

---

## 35. What is the difference between `volatile` and `atomic`?

**Answer:**  
`volatile` prevents compiler optimization but no atomic operations; `atomic` provides thread-safety.

---

## 36. What is `thread_local` overhead?

**Answer:**  
Access via TLS array (extra indirection, slower than stack variable).

---

## 37. How to make code cache-efficient for multidimensional arrays?

**Answer:**  
Traverse in contiguous memory order (row-major for C++ vectors of vectors).

---

## 38. What is a `cache-oblivious algorithm`?

**Answer:**  
Algorithm efficient without knowing cache size (e.g., divide-and-conquer).

---

## 39. What is the difference between `reserve()` and `resize()` for `vector`?

**Answer:**  
`reserve` allocates capacity but no objects; `resize` creates default-initialized elements.

---

## 40. Why is `shrink_to_fit` not guaranteed?

**Answer:**  
May be ignored, `realloc` not always possible.

---

## 41. What is `std::align`?

**Answer:**  
Aligns pointer to specified alignment.

```cpp
size_t space = 100;
void* ptr = malloc(100);
void* aligned = std::align(16, 10, ptr, space);
```

---

## 42. What is the cost of `std::stable_sort` vs `std::sort`?

**Answer:**  
Stable sort O(n log n) but needs extra memory (or slower algorithm).

---

## 43. How does `std::vector<bool>` differ? Performance trap?

**Answer:**  
Packed bits (1 byte stores 8 bools) — slow for threading, proxy iterators.

```cpp
vector<bool> vb(10);
auto& ref = vb[0]; // error: proxy reference, not bool&
```

---

## 44. What is a `jump table`? When used?

**Answer:**  
Switch on dense integers → compiler may generate jump table (O(1) dispatch).

---

## 45. What is `__declspec(align)` (MSVC) or `__attribute__((aligned))` (GCC)?

**Answer:**  
Compiler-specific alignment directives.

```cpp
__declspec(align(32)) int x; // Windows
int x __attribute__((aligned(32))); // GCC
```

---

## 46. How does `memcpy` compare to element-wise copy?

**Answer:**  
`memcpy` highly optimized (SIMD, loop unrolling) — faster for trivially copyable types.

---

## 47. What is the cost of `std::tuple` compared to `struct`?

**Answer:**  
Zero overhead (layout same as struct) if fields trivially copyable, but uses recursion in implementation.

---

## 48. What is a `CFG` (Control Flow Guard)?

**Answer:**  
Security feature — runtime check of indirect call targets. Small performance hit.

---

## 49. What is `-O3` dangerous optimization?

**Answer:**  
May break floating-point semantics (associative math), strict aliasing assumptions.

---

## 50. How to measure and profile performance in C++?

**Answer:**  
- `std::chrono::high_resolution_clock`
- `perf` (Linux)
- Intel VTune / AMD uProf
- `google/benchmark`
- `valgrind --tool=cachegrind` for cache simulation

---

Let me know if you want **350+ questions** across all 7 topics compiled into a single document, or a self-test quiz generator.