# C++ Pitfalls Cheat Sheet (Complete English Version)
Perfect for interviews, debugging, and writing safe, efficient C++ code — all pitfalls organized by category, with bug/fix examples and key takeaways.

---

## 1. STL Pitfalls
### 1.1 `std::map operator[]` Inserts Unintentionally
**Bug**: Accessing `map[key]` inserts a default value (0 for `int`) even when only checking existence.
```cpp
std::map<std::string, int> m;
if (m["key"] == 0) { /* key inserted unexpectedly! */ }
```
**Fix**: Use `find()` (C++11+) or `contains()` (C++20+) for safe lookup.
```cpp
if (m.find("key") != m.end()) { /* safe, no insertion */ }
if (m.contains("key")) { /* cleaner C++20 syntax */ }
```
**Note**: Only use `map[]` when you intend to insert/modify the key.

### 1.2 Invalidated Vector Iterators
**Bug**: Modifying a vector (`push_back`, `insert`, `erase`) can invalidate all iterators (due to reallocation), leading to crashes or undefined behavior.
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 3) v.push_back(99); // iterator `it` becomes invalid!
}
```
**Fix**: Use an index-based loop or collect changes first.
```cpp
// Safe index-based loop
for (size_t i = 0; i < v.size(); ++i) {
    if (v[i] == 3) v.push_back(99);
}

// Alternative: Collect changes first
std::vector<int> to_add;
for (int num : v) { if (num == 3) to_add.push_back(99); }
for (int num : to_add) v.push_back(num);
```

### 1.3 Forgotten `erase` in erase-remove Idiom
**Bug**: `std::remove` only shifts elements to the end (does not erase them), leaving the container size unchanged and garbage values at the end.
```cpp
std::vector<int> v2 = {1, 2, 3, 2, 4};
std::remove(v2.begin(), v2.end(), 2); // size remains 5 (bug!)
```
**Fix**: Always pair `std::remove` with `vector::erase` (erase-remove idiom).
```cpp
v2.erase(std::remove(v2.begin(), v2.end(), 2), v2.end()); // size = 3 (correct)
```

### 1.4 Unsigned `size()` Underflow
**Bug**: `size()` returns an unsigned type (`size_t`); if the container is empty, `size() - 1` underflows to a huge number, causing infinite loops or out-of-bounds access.
```cpp
std::vector<int> v3; // empty vector
for (int i = 0; i <= v3.size() - 1; ++i) { /* infinite loop! */ }
```
**Fix**: Avoid unsigned arithmetic; use range-based for or `size()` without subtraction.
```cpp
// Safe options
for (size_t i = 0; i < v3.size(); ++i) { /* no underflow risk */ }
for (int num : v3) { /* cleanest, C++11+ */ }
```

### 1.5 Accidental Deep Copy of Large Containers
**Bug**: Using `auto` to assign a container creates an expensive deep copy (O(n) time/memory), which is wasteful for large datasets.
```cpp
std::map<int, std::vector<int>> bigMap; // large container
auto copy = bigMap; // O(n) deep copy (bug for performance)
```
**Fix**: Use a `const` reference for zero-cost access (no copy).
```cpp
const auto& ref = bigMap; // O(1), no copy (read-only)
auto& ref = bigMap; // Use non-const reference if modification is needed
```

### 1.6 `unordered_map` Hash Collisions & Rehashing
**Bug**: Without preallocation, `unordered_map` frequently rehashes (resizes its internal table) during large insertions, leading to O(n) worst-case performance.
```cpp
std::unordered_map<int, int> um;
for (int i = 0; i < 10000; ++i) um[i] = i; // many rehashes (slow)
```
**Fix**: Use `reserve()` to preallocate space and avoid rehashing.
```cpp
std::unordered_map<int, int> um;
um.reserve(10000); // pre-size to fit all elements
for (int i = 0; i < 10000; ++i) um[i] = i; // fast, no rehashes
```
**Note**: Reserve ~1.2x the number of elements for optimal performance.

---

## 2. Basic Syntax Pitfalls
### 2.1 Uninitialized Variables
**Bug**: Local variables are not initialized, so they hold random garbage values (undefined behavior).
```cpp
int x; // uninitialized (garbage value)
std::cout << x; // undefined behavior (random output)
```
**Fix**: Always initialize variables (explicitly or via zero-initialization).
```cpp
int x = 0; // explicit initialization
int y{}; // C++11+ list initialization (zero-initializes)
```

### 2.2 Confusing `=` (Assignment) and `==` (Comparison)
**Bug**: Accidentally using `=` (assignment) instead of `==` (equality check) in conditionals, leading to logical errors.
```cpp
int a = 5;
if (a = 3) { // assigns 3 to a; condition is always true (non-zero = true)
    // unintended logic
}
```
**Fix**: Use `==` for comparisons; place constants on the left to catch typos.
```cpp
if (a == 3) { /* correct */ }
if (3 == a) { /* safer: typo `=` will cause compile error (cannot assign to constant) */ }
```

### 2.3 Integer Division Truncation
**Bug**: Dividing two integers truncates the decimal part (no rounding), leading to incorrect calculations.
```cpp
int a = 5, b = 2;
int res = a / b; // result = 2 (not 2.5)
int res2 = -5 / 2; // result = -2 (truncates toward zero, not -3)
```
**Fix**: Cast one operand to a floating-point type before division.
```cpp
double res = static_cast<double>(a) / b; // 5.0 / 2 = 2.5
double res2 = static_cast<double>(-5) / 2; // -2.5
```

### 2.4 Extra/Missing Semicolons
**Bug 1 (Missing Semicolon)**: Forgetting a semicolon at the end of a statement causes a compile error.
```cpp
int x = 5 // missing semicolon (compile error)
if (x > 3) { /* ... */ }
```
**Bug 2 (Extra Semicolon)**: An extra semicolon after `if`/`while`/`for` turns the condition into a no-op, making the following block run unconditionally.
```cpp
if (x > 3); { // extra semicolon (empty statement)
    std::cout << "x > 3"; // runs even if x <= 3
}
```
**Fix**: Always add a semicolon at the end of statements; never add a semicolon after conditional keywords.

### 2.5 Variable Shadowing
**Bug**: A local variable has the same name as an outer/global variable, "hiding" the outer variable and leading to accidental use of the wrong value.
```cpp
int num = 10; // global variable
void func() {
    int num = 20; // local variable (hides global num)
    std::cout << num; // prints 20 (not 10 — easy to misinterpret)
}
```
**Fix**: Avoid naming conflicts; use the scope resolution operator (`::`) to access global variables if needed.
```cpp
void func() {
    int local_num = 20; // rename to avoid conflict
    std::cout << ::num; // access global num (prints 10)
}
```

### 2.6 Invalid C-String Literal Concatenation
**Bug**: Using `+` to concatenate string literals (`const char*`) performs pointer arithmetic (undefined behavior), not string concatenation.
```cpp
// Error: "Hello" and " World" are const char*; + adds their addresses
std::cout << "Hello" + " World";
```
**Fix**: Convert one literal to `std::string` or use stream insertion (`<<`) for concatenation.
```cpp
std::cout << std::string("Hello") + " World"; // correct concatenation
std::cout << "Hello" << " World"; // alternative (simple and safe)
```

### 2.7 Missing `break` in `switch-case`
**Bug**: Without `break`, execution "falls through" to the next `case`, leading to unexpected behavior.
```cpp
int choice = 2;
switch (choice) {
    case 1: std::cout << "1";
    case 2: std::cout << "2"; // runs
    case 3: std::cout << "3"; // also runs (fallthrough bug)
}
// Output: 23 (unintended)
```
**Fix**: Add `break` to every `case` (except for intentional fallthrough).
```cpp
switch (choice) {
    case 1: std::cout << "1"; break;
    case 2: std::cout << "2"; break; // no fallthrough
    case 3: std::cout << "3"; break;
    default: std::cout << "Invalid choice"; break;
}
// Output: 2 (correct)
```

---

## 3. Memory Management Pitfalls
### 3.1 Memory Leak (Forgotten `delete`/`delete[]`)
**Bug**: Allocating memory with `new`/`new[]` but never freeing it with `delete`/`delete[]` leads to memory leaks (memory is lost forever).
```cpp
void leak() {
    int* ptr = new int(10); // allocate on heap
    // No delete → memory leak (ptr goes out of scope, memory is unreachable)
}
```
**Fix**: Use smart pointers (`std::unique_ptr`, `std::shared_ptr`) to auto-manage memory, or explicitly free with `delete`/`delete[]`.
```cpp
// Best fix: smart pointer (auto-frees when out of scope)
std::unique_ptr<int> ptr = std::make_unique<int>(10);

// Alternative: explicit delete (riskier, easy to forget)
int* ptr = new int(10);
delete ptr; // free memory
```

### 3.2 Mismatched `new`/`delete` (Single vs Array)
**Bug**: Allocating an array with `new[]` but freeing it with `delete` (not `delete[]`) causes undefined behavior (heap corruption, crashes).
```cpp
int* arr = new int[5]; // allocate array
delete arr; // wrong (mismatched with new[])
```
**Fix**: Always match `new[]` with `delete[]` and `new` with `delete`.
```cpp
delete[] arr; // correct (frees the entire array)
```

### 3.3 Dangling Pointer (Use After Free)
**Bug**: Using a pointer after the memory it points to has been freed (deleted) leads to undefined behavior.
```cpp
int* ptr = new int(5);
delete ptr; // free memory
*ptr = 10; // dangling pointer (memory is no longer valid)
```
**Fix**: Set pointers to `nullptr` after `delete`; check for `nullptr` before dereferencing.
```cpp
delete ptr;
ptr = nullptr; // mark as invalid
if (ptr != nullptr) {
    *ptr = 10; // safe (skip if null)
}
```

### 3.4 Wild/Uninitialized Pointer
**Bug**: A pointer that is not initialized points to random memory (wild pointer), leading to undefined behavior when dereferenced.
```cpp
int* ptr; // uninitialized (wild pointer)
*ptr = 10; // undefined behavior (writes to random memory)
```
**Fix**: Always initialize pointers to `nullptr` or a valid memory address.
```cpp
int* ptr = nullptr; // safe (no random memory access)
int x = 5;
int* valid_ptr = &x; // valid (points to a local variable)
```

### 3.5 Double Free
**Bug**: Freeing the same memory twice (double `delete`) causes heap corruption and crashes.
```cpp
int* ptr = new int(5);
delete ptr;
delete ptr; // double free (bug)
```
**Fix**: Set pointers to `nullptr` after `delete`; `delete nullptr` is safe (does nothing).
```cpp
delete ptr;
ptr = nullptr;
delete ptr; // safe (no operation)
```

### 3.6 Returning Pointer to Local Stack Variable
**Bug**: Local variables are destroyed when the function exits, so returning a pointer to a local variable results in a dangling pointer.
```cpp
int* bad_func() {
    int x = 10; // local variable (stack-allocated)
    return &x; // x is destroyed when function returns
}

int* ptr = bad_func(); // dangling pointer
```
**Fix**: Return by value, use a `static` local variable, or allocate memory on the heap (with a smart pointer).
```cpp
// Best fix: return smart pointer (heap-allocated, auto-frees)
std::unique_ptr<int> good_func() {
    return std::make_unique<int>(10);
}

// Alternative: return by value (no pointer needed)
int good_func() {
    return 10;
}
```

### 3.7 Using Reference to Destroyed Object
**Bug**: A reference to an object becomes dangling when the object is destroyed, leading to undefined behavior when used.
```cpp
int& ref = *new int(5); // reference to heap-allocated int
delete &ref; // destroy the object
ref = 10; // dangling reference (object no longer exists)
```
**Fix**: Ensure the object’s lifetime exceeds the reference’s lifetime, or use smart pointers instead of raw references.

### 3.8 Not Handling `nullptr` in Raw Pointers
**Bug**: Dereferencing a `nullptr` (or a pointer that could be `nullptr`) causes a crash.
```cpp
int* ptr = nullptr;
int val = *ptr; // crash (dereferencing null)
```
**Fix**: Always check for `nullptr` before dereferencing a raw pointer.
```cpp
if (ptr != nullptr) {
    int val = *ptr; // safe
}
```

### 3.9 Heap Fragmentation
**Bug**: Frequent small allocations and deallocations leave small, unused "holes" in the heap, reducing available memory and slowing down future allocations.
```cpp
// Example: frequent small allocations (causes fragmentation)
for (int i = 0; i < 1000; ++i) {
    int* ptr = new int(i);
    delete ptr;
}
```
**Fix**: Use `reserve()` for containers (to preallocate contiguous memory), memory pools, or avoid frequent small allocations.

### 3.10 Forgetting Virtual Destructor in Polymorphic Classes
**Bug**: When a base class pointer points to a derived class object, deleting the base pointer without a virtual destructor causes only the base part of the object to be freed (memory leak).
```cpp
class Base {
public:
    ~Base() {} // not virtual (bug)
};

class Derived : public Base {
public:
    int* data = new int(10); // heap-allocated member
    ~Derived() { delete data; } // never called!
};

Base* ptr = new Derived;
delete ptr; // leaks Derived::data (only Base::~Base() is called)
```
**Fix**: Make the base class destructor `virtual` to ensure the derived class destructor is called.
```cpp
class Base {
public:
    virtual ~Base() = default; // virtual destructor (correct)
};
```

---

## 4. General Programming Pitfalls
### 4.1 Null Pointer Dereference (Repeated for Clarity)
**Bug**: Dereferencing a `nullptr` causes crashes or undefined behavior (covered in memory pitfalls but worth emphasizing).
```cpp
int* ptr = nullptr;
*ptr = 5; // crash
```
**Fix**: Check for `nullptr` before dereferencing.

### 4.2 Integer Overflow/Underflow
**Bug**: Exceeding the maximum/minimum value of an integer type (e.g., `INT_MAX + 1`) leads to undefined behavior.
```cpp
int max_int = INT_MAX;
int overflow = max_int + 1; // undefined behavior
int min_int = INT_MIN;
int underflow = min_int - 1; // undefined behavior
```
**Fix**: Use larger integer types (e.g., `long long`), or check for overflow before operations (C++20 `std::cmp_add_overflow`).
```cpp
long long max_int = INT_MAX;
long long overflow = max_int + 1; // no overflow (larger type)

// C++20: check for overflow
int res;
if (std::cmp_add_overflow(max_int, 1, &res)) {
    // handle overflow (e.g., log error)
}
```

### 4.3 Off-by-One Errors (Loop Bounds)
**Bug**: Incorrect loop bounds (e.g., using `<=` instead of `<` for 0-based indices) cause out-of-bounds access or incomplete processing.
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
// Bug: accesses v[5] (out of bounds; indices 0-4)
for (int i = 0; i <= 5; ++i) {
    std::cout << v[i];
}
```
**Fix**: Use `i < v.size()` for 0-based loops or range-based for.
```cpp
// Safe
for (int i = 0; i < v.size(); ++i) {
    std::cout << v[i];
}

// Cleanest (C++11+)
for (int num : v) {
    std::cout << num;
}
```

### 4.4 Race Conditions (Multithreading)
**Bug**: Multiple threads access/modify shared data without synchronization, leading to data corruption or inconsistent results.
```cpp
int count = 0;
// Two threads increment count (race condition)
std::thread t1([&]() { for (int i = 0; i < 1000; ++i) count++; });
std::thread t2([&]() { for (int i = 0; i < 1000; ++i) count++; });
t1.join();
t2.join();
// count may be less than 2000 (data corruption)
```
**Fix**: Use synchronization primitives like `std::mutex` or `std::atomic` (for basic types).
```cpp
// Fix 1: Mutex (for complex operations)
std::mutex mtx;
int count = 0;
std::thread t1([&]() {
    for (int i = 0; i < 1000; ++i) {
        std::lock_guard<std::mutex> lock(mtx); // lock during modification
        count++;
    }
});

// Fix 2: Atomic (simpler for basic types)
std::atomic<int> count(0);
std::thread t1([&]() { for (int i = 0; i < 1000; ++i) count++; });
```

---

## Key Interview Takeaways (Critical for Exams/Interviews)
1. **STL**: Never use `map[]` for lookup (use `find()`/`contains()`); always use `erase-remove` for vector element deletion.
2. **Syntax**: Initialize all variables; avoid confusing `=` and `==`; add `break` in `switch-case`; avoid variable shadowing.
3. **Memory**: Use smart pointers instead of raw `new`/`delete`; match `new[]` with `delete[]`; never use dangling pointers.
4. **General**: Check for `nullptr`; avoid integer overflow; use synchronization for multithreaded shared data.
5. **Polymorphism**: Always make base class destructors `virtual` to avoid memory leaks.

---

This is a complete, standalone Markdown file — you can save it as `cpp-pitfalls-cheat-sheet.md` and use it for quick revision. Let me know if you want to **add more pitfalls** (e.g., template-related, exception handling) or **simplify any section** for better readability!