Here are **30 tricky C++ interview questions** with answers and examples, categorized by your requested subjects.

---

## 1. Memory Management

### Q1: What’s the difference between `delete` and `delete[]`? What happens if you mix them?
**A:** `delete` calls the destructor for a single object and frees memory; `delete[]` does so for an array. Mixing them leads to undefined behavior (heap corruption, wrong destructor calls).

**Example:**
```cpp
int* p1 = new int(5);
int* p2 = new int[10];
delete p1;   // OK
delete[] p2; // OK
// delete p2;   // WRONG – undefined behavior
```

### Q2: How does `weak_ptr` solve the dangling pointer problem?
**A:** `weak_ptr` doesn’t own the object; it only observes a `shared_ptr`. It can detect if the object is alive via `expired()` or `lock()`.

**Example:**
```cpp
shared_ptr<int> sp = make_shared<int>(42);
weak_ptr<int> wp = sp;
sp.reset();
if (wp.expired()) // true – object gone
    cout << "Dangling avoided";
```

### Q3: What’s wrong with `return shared_ptr<T>(this)` in a class?
**A:** It creates multiple unrelated `shared_ptr` owners, causing double-free. Use `enable_shared_from_this`.

**Example:**
```cpp
class Bad : public enable_shared_from_this<Bad> {
public:
    shared_ptr<Bad> get() { return shared_from_this(); } // correct
};
```

---

## 2. OOP & C++ Class Fundamentals

### Q4: Why should destructors be virtual in polymorphic base classes?
**A:** Otherwise, deleting a derived object through a base pointer won’t call the derived destructor → resource leak.

**Example:**
```cpp
class Base { public: ~Base() { cout << "Base\n"; } };
class Derived : public Base { public: ~Derived() { cout << "Derived\n"; } };
Base* p = new Derived();
delete p; // Only "Base" printed – leak if Derived had resources
```

### Q5: What’s the vtable and vptr? When are they created?
**A:** vtable (virtual table) holds function pointers for virtual functions. vptr points to vtable. vptr is set in the constructor (base → derived order).

### Q6: Can a constructor be virtual? Why?
**A:** No. Virtual calls depend on vptr which is not set until constructor runs.

---

## 3. Move Semantics & Perfect Forwarding

### Q7: Why is `std::move` just a cast? Does it actually move anything?
**A:** No, it’s just `static_cast<T&&>`. It marks an object as movable; the actual move happens in the constructor/assignment that takes an rvalue reference.

**Example:**
```cpp
vector<int> v1{1,2,3};
vector<int> v2 = std::move(v1); // now v1 is valid but unspecified
```

### Q8: What’s the difference between `std::move` and `std::forward`?
**A:** `std::move` unconditionally casts to rvalue. `std::forward` conditionally casts (preserves value category) — used in perfect forwarding.

**Example:**
```cpp
template<typename T>
void wrapper(T&& arg) {
    foo(std::forward<T>(arg)); // forwards as original type
}
```

### Q9: What is copy elision? When is it guaranteed in C++17?
**A:** Copy elision eliminates copy/move constructor calls. C++17 guarantees it for prvalues (temporary materialization).

**Example:**
```cpp
vector<int> make_vec() { return {1,2,3}; }
vector<int> v = make_vec(); // no copy/move in C++17
```

---

## 4. STL Containers & Iterators

### Q10: When does `vector::insert` invalidate iterators?
**A:** If reallocation occurs → all iterators invalidated. Without reallocation, only those after the insertion point are invalidated.

### Q11: Why is `emplace_back` faster than `push_back`?
**A:** `push_back` requires a constructed object (copy/move); `emplace_back` constructs in-place with forwarded arguments.

**Example:**
```cpp
struct S { S(int a, int b){} };
vector<S> v;
v.emplace_back(1,2); // constructs in-place
v.push_back(S(1,2)); // constructs then moves
```

### Q12: Choose `map` vs `unordered_map`?
**A:** `map` (ordered, O(log n), stable iterators) vs `unordered_map` (hash, O(1) avg, unstable). Use `map` for ordering/range queries, `unordered_map` for fast lookup.

---

## 5. Concurrency & Multithreading

### Q13: How do you avoid deadlock with two mutexes?
**A:** Lock in consistent order or use `std::lock` or `std::scoped_lock` (C++17).

**Example:**
```cpp
std::mutex m1, m2;
void safe() {
    std::scoped_lock lock(m1, m2); // deadlock-safe
}
```

### Q14: What’s a race condition? Give a simple example.
**A:** Two threads accessing shared data without synchronization, at least one writes.

```cpp
int counter = 0;
void increment() { ++counter; } // thread-unsafe
```

### Q15: What memory orders in `std::atomic` guarantee sequential consistency?
**A:** `std::memory_order_seq_cst` (default). Others: `relaxed`, `acquire`, `release`, `acq_rel`.

---

## 6. Templates & Generic Programming

### Q16: What is SFINAE? Give a minimal example.
**A:** Substitution Failure Is Not An Error — compiler discards templates that produce invalid types.

**Example:**
```cpp
template<typename T>
auto test(T t) -> decltype(t.foo(), void()) { } // only if T has foo()
```

### Q17: What’s the difference between `typename` and `class` in template parameters?
**A:** No difference, except `typename` is needed for dependent types.

```cpp
template<typename T>
void f() { typename T::type var; }
```

### Q18: How to check if a type is a pointer at compile time?
**A:** Use `std::is_pointer` trait.

```cpp
static_assert(std::is_pointer<int*>::value);
```

---

## 7. C++11/14/17/20 Modern Features

### Q19: What’s the benefit of `constexpr` over `const`?
**A:** `constexpr` guarantees evaluation at compile time where possible; can be used in array sizes, template args.

**Example:**
```cpp
constexpr int square(int x) { return x*x; }
int arr[square(5)]; // OK
```

### Q20: What are structured bindings? Give example.
**A:** Decompose tuple/pair/struct into named variables (C++17).

```cpp
map<int,string> m = {{1,"a"}};
for (const auto& [key,val] : m) { }
```

### Q21: `std::optional` vs returning a sentinel like `-1`?
**A:** `optional` makes absence explicit, type-safe, avoids magic numbers.

```cpp
optional<int> find(string s) {
    if (s == "a") return 42;
    return nullopt;
}
```

---

## 8. Const Correctness & Type Safety

### Q22: What does `mutable` do? When is it used?
**A:** Allows modifying a class member inside a `const` member function — typically for caching, mutexes.

**Example:**
```cpp
class Cache {
    mutable int cached_val = 0;
public:
    int get() const { cached_val = 42; return cached_val; } // OK
};
```

### Q23: Difference between `const int*`, `int const*`, `int* const`?
**A:** `const int*` = pointer to const int. `int const*` = same. `int* const` = const pointer to int.

### Q24: Why use `explicit` constructor?
**A:** Prevents implicit conversions that may cause unintended behavior.

```cpp
class String {
public:
    explicit String(int size) {} // prevents String s = 5;
};
```

---

## 9. Exception Safety & Error Handling

### Q25: What are the three exception safety levels?
**A:** Basic (no leak, invariants ok), Strong (commit/rollback), Nothrow (`noexcept`).

### Q26: Why RAII helps with exception safety?
**A:** Destructors run during stack unwinding, automatically freeing resources.

```cpp
void f() {
    vector<int> v(10000); // automatically destroyed
    throw runtime_error(""); // no leak
}
```

### Q27: When is `noexcept` useful?
**A:** Enables move optimizations (e.g., `vector` uses moves if `noexcept`), signals intent.

---

## 10. Low-Level / System & Performance

### Q28: What is false sharing? How to avoid?
**A:** Cache line contention when unrelated variables share a line. Avoid by padding/alignment (`alignas(64)`).

### Q29: What causes object slicing?
**A:** Assigning derived object to base object by value — derived part cut off.

```cpp
struct Base { int a; };
struct Derived : Base { int b; };
Derived d;
Base b = d; // b lost d.b
```

### Q30: Why are virtual calls slower than non-virtual?
**A:** Indirect through vptr + prevents inlining unless devirtualized by compiler.

---

Would you like me to expand any of these into a full code‑walkthrough or provide a downloadable cheat‑sheet version?