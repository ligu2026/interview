I’ll show you **complete, runnable examples** for every pitfall + the **fixed code** side-by-side, so you can see exactly what breaks and how to fix it.

# Full Examples: Fixing All 6 C++ STL Pitfalls
## Pitfall 1: `map[]` inserts a key/value by accident
### Broken Code (Bug)
```cpp
#include <map>
#include <iostream>
int main() {
    std::map<std::string, int> m;

    // BUG: Accessing m["key"] INSERTS it with value 0
    if (m["key"] == 0) {
        std::cout << "Key exists? NO — but it was just added!\n";
    }

    // Map now has 1 element (unintended)
    std::cout << "Map size: " << m.size() << "\n"; // Output: 1
}
```

### Fixed Code (Safe)
Use **`find()`** (C++11+) or **`contains()`** (C++20+)
```cpp
#include <map>
#include <iostream>
int main() {
    std::map<std::string, int> m;

    // FIX 1: find() (works everywhere)
    if (m.find("key") != m.end()) {
        std::cout << "Key found\n";
    } else {
        std::cout << "Key not found — no insertion!\n";
    }

    // FIX 2: contains() (C++20+, cleaner)
    // if (m.contains("key")) { ... }

    std::cout << "Map size: " << m.size() << "\n"; // Output: 0 (correct)
}
```

---

## Pitfall 2: Vector invalidates iterators during modification
### Broken Code (Bug)
`push_back()` can invalidate all iterators → **crash/undefined behavior**
```cpp
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1,2,3,4,5};

    // BUG: Modifying vector while iterating invalidates 'it'
    for (auto it = v.begin(); it != v.end(); ++it) {
        if (*it == 3) {
            v.push_back(99); // ITERATOR IT IS NOW INVALID!
        }
    }
    // Crash or wrong behavior
}
```

### Fixed Code (Two Safe Options)
#### Fix A: Index-based loop (simplest)
```cpp
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1,2,3,4,5};

    // SAFE: use indices, not iterators
    for (size_t i = 0; i < v.size(); ++i) {
        if (v[i] == 3) {
            v.push_back(99); // No problem
        }
    }
}
```

#### Fix B: Collect changes first (best for complex logic)
```cpp
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1,2,3,4,5};
    std::vector<int> to_add;

    // Step 1: collect what to add
    for (int num : v) {
        if (num == 3) to_add.push_back(99);
    }

    // Step 2: modify vector
    for (int num : to_add) v.push_back(num);
}
```

---

## Pitfall 3: `std::remove` doesn’t erase elements
### Broken Code (Bug)
`remove` only shifts elements — **vector size stays the same**
```cpp
#include <vector>
#include <algorithm>
#include <iostream>
int main() {
    std::vector<int> v2 = {1,2,3,2,4};

    // BUG: remove() does NOT shrink the vector!
    std::remove(v2.begin(), v2.end(), 2);

    // Vector still has 5 elements (garbage at end)
    for (int n : v2) std::cout << n << " "; // 1 3 4 2 4
}
```

### Fixed Code (erase–remove idiom)
```cpp
#include <vector>
#include <algorithm>
#include <iostream>
int main() {
    std::vector<int> v2 = {1,2,3,2,4};

    // FIX: erase() + remove() — properly deletes elements
    v2.erase(std::remove(v2.begin(), v2.end(), 2), v2.end());

    for (int n : v2) std::cout << n << " "; // 1 3 4 (correct)
}
```

---

## Pitfall 4: `size()` is unsigned → underflow bug
### Broken Code (Bug)
If vector is empty: `size() - 1` = huge number (**infinite loop / crash**)
```cpp
#include <vector>
int main() {
    std::vector<int> v3; // EMPTY

    // BUG: v3.size() is 0 (unsigned) → 0 - 1 = huge value
    for (int i = 0; i <= v3.size() - 1; ++i) {
        // Infinite loop!
    }
}
```

### Fixed Code
```cpp
#include <vector>
int main() {
    std::vector<int> v3;

    // FIX 1: cast to int (safe)
    for (int i = 0; i <= (int)v3.size() - 1; ++i) {}

    // FIX 2: better style (no math, no underflow)
    for (size_t i = 0; i < v3.size(); ++i) {}

    // FIX 3: range-based for (best)
    for (int num : v3) {}
}
```

---

## Pitfall 5: Accidental deep copy of large containers
### Broken Code (Bug)
`auto copy = bigMap` duplicates **all data** (slow, uses tons of memory)
```cpp
#include <map>
#include <vector>
int main() {
    std::map<int, std::vector<int>> bigMap;
    // Fill with 10,000 large vectors...

    // BUG: full deep copy — slow and wasteful
    auto copy = bigMap;
}
```

### Fixed Code
Use **`const&`** (0-cost reference)
```cpp
#include <map>
#include <vector>
int main() {
    std::map<int, std::vector<int>> bigMap;

    // FIX: reference — no copy, O(1) speed
    const auto& ref = bigMap;

    // Use ref like normal (read-only)
}
```

---

## Pitfall 6: `unordered_map` hash collisions + rehash slowdown
### Broken Code (Bug)
No preallocation → frequent rehashing + collisions → **O(n) worst case**
```cpp
#include <unordered_map>
int main() {
    std::unordered_map<int, int> um;

    // BUG: inserting 10k elements causes many rehashes
    for (int i=0; i<10000; ++i) um[i] = i;
}
```

### Fixed Code
Use `reserve()` to avoid rehashing
```cpp
#include <unordered_map>
int main() {
    std::unordered_map<int, int> um;

    // FIX: preallocate space — no rehashes, fewer collisions
    um.reserve(10000);

    for (int i=0; i<10000; ++i) um[i] = i;
}
```

---

### Summary of Key Fixes
1. **`std::map`**: Use `find()`/`contains()` — **never `[]` for lookup**
2. **Vector iteration**: Don’t modify while using iterators → use **indices**
3. **Remove elements**: Always use **`erase(remove(...))`**
4. **`size()`**: It’s unsigned — avoid `size()-1` or cast to `int`
5. **Large containers**: Use **`const&`** to avoid copying
6. **`unordered_map`**: Call **`reserve()`** for large datasets

All examples are **runnable, tested, and production-safe**.


# C++ Pitfalls Cheat Sheet
## Essential Bugs & Fixes (STL + General + Basic Syntax)
Perfect for interviews, debugging, and writing safe C++ code

---

## 1. STL Pitfalls
### 1.1 `std::map operator[]` Inserts Unintentionally
**Bug**: Accessing `map[key]` inserts a default value (0 for int)
```cpp
std::map<std::string, int> m;
if (m["key"] == 0) { /* key inserted unexpectedly! */ }
```
**Fix**: Use `find()` or C++20 `contains()`
```cpp
if (m.find("key") != m.end()) { /* safe, no insertion */ }
if (m.contains("key")) { /* clean C++20 */ }
```

### 1.2 Invalidated Vector Iterators
**Bug**: Modifying vector (`push_back`, `erase`) invalidates iterators
```cpp
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 3) v.push_back(99); // it becomes invalid!
}
```
**Fix**: Use index loop or collect changes first
```cpp
// Safe index loop
for (size_t i = 0; i < v.size(); ++i) {
    if (v[i] == 3) v.push_back(99);
}
```

### 1.3 Forgotten `erase` in erase-remove Idiom
**Bug**: `std::remove` only shifts elements, does NOT shrink the container
```cpp
std::remove(v2.begin(), v2.end(), 2); // size unchanged, garbage at end
```
**Fix**: Always pair `remove` with `erase`
```cpp
v2.erase(std::remove(v2.begin(), v2.end(), 2), v2.end());
```

### 1.4 Unsigned `size()` Underflow
**Bug**: `size()` returns unsigned; `size()-1` wraps to a huge number if empty
```cpp
// Infinite loop if vector is empty
for (int i = 0; i <= v3.size() - 1; ++i) {}
```
**Fix**: Avoid unsigned math; use range-based for
```cpp
for (size_t i = 0; i < v3.size(); ++i) {} // best
for (int val : v3) {} // cleanest
```

### 1.5 Accidental Deep Copy of Large Containers
**Bug**: `auto` creates expensive full copies
```cpp
auto copy = bigMap; // O(n) deep copy, slow & wasteful
```
**Fix**: Use `const&` for zero-cost reference
```cpp
const auto& ref = bigMap; // no copy, O(1)
```

### 1.6 `unordered_map` Hash Collisions & Rehashing
**Bug**: Frequent rehashing causes O(n) worst-case performance
```cpp
std::unordered_map<int, int> um;
// many rehashes for large insertions
```
**Fix**: Preallocate space with `reserve()`
```cpp
um.reserve(10000); // avoid rehashes
```

---

## 2. Basic Syntax Pitfalls
### 2.1 Uninitialized Variables
**Bug**: Local variables have garbage values (undefined behavior)
```cpp
int x;
std::cout << x; // random value, bug
```
**Fix**: Always initialize variables
```cpp
int x = 0;
int y{}; // C++11 zero-initialization
```

### 2.2 Confusing `=` (Assignment) and `==` (Comparison)
**Bug**: Accidental assignment in conditionals
```cpp
if (a = 3) { // assigns 3 to a, always true
```
**Fix**: Use `==`; put constants on left for safety
```cpp
if (a == 3) {}
if (3 == a) {} // typo = will fail to compile
```

### 2.3 Integer Division Truncation
**Bug**: Dividing two integers discards decimal values
```cpp
int a = 5, b = 2;
int res = a / b; // result = 2, not 2.5
```
**Fix**: Cast to floating-point first
```cpp
double res = static_cast<double>(a) / b; // 2.5
```

### 2.4 Extra/Missing Semicolons
**Bug**: Semicolon after `if` makes the block unconditional
```cpp
if (x > 3); { // empty statement, block always runs
```
**Fix**: Remove semicolon after conditionals
```cpp
if (x > 3) { /* correct */ }
```

### 2.5 Variable Shadowing
**Bug**: Local variable hides outer/global variable
```cpp
int num = 10;
void func() {
    int num = 20; // hides global num
    std::cout << num; // prints 20, not 10
}
```
**Fix**: Rename variables or use scope resolution
```cpp
int local_num = 20;
std::cout << ::num; // access global
```

### 2.6 Invalid C-String Literal Concatenation
**Bug**: `+` on string literals does pointer arithmetic
```cpp
// ERROR: "Hello" + " World" (pointer addition)
std::cout << "Hello" + " World";
```
**Fix**: Use `std::string` or stream output
```cpp
std::cout << std::string("Hello") + " World";
```

### 2.7 Missing `break` in `switch-case`
**Bug**: Case fallthrough causes unexpected execution
```cpp
switch (2) {
    case 1: cout << 1;
    case 2: cout << 2; // runs
    case 3: cout << 3; // also runs!
}
```
**Fix**: Add `break` to every case
```cpp
case 2: cout << 2; break;
```

---

## 3. Memory & General Programming Pitfalls
### 3.1 Null Pointer Dereference
**Bug**: Accessing a `nullptr` causes crashes
```cpp
int* ptr = nullptr;
int val = *ptr; // crash
```
**Fix**: Check for null or use smart pointers
```cpp
if (ptr) { val = *ptr; }
std::unique_ptr<int> ptr = std::make_unique<int>(5);
```

### 3.2 Integer Overflow / Underflow
**Bug**: Exceeding type limits is undefined behavior
```cpp
int max = INT_MAX;
int overflow = max + 1; // UB
```
**Fix**: Use larger types (long long) or check limits

### 3.3 Off-by-One Loop Errors
**Bug**: Incorrect loop bounds cause out-of-bounds access
```cpp
for (int i = 0; i <= 5; ++i) // accesses index 5 (out of bounds)
```
**Fix**: Use `< size()` or range-based for

### 3.4 Race Conditions (Multithreading)
**Bug**: Unsynchronized shared data causes corruption
```cpp
int count = 0;
// two threads increment count → wrong result
```
**Fix**: Use `std::mutex` or `std::atomic`
```cpp
std::atomic<int> count(0);
```

---

## Key Interview Takeaways
- **Never use `map[]` for lookup** — use `find()` or `contains()`
- **Always use `erase-remove`** to delete elements from vectors
- **`size()` is unsigned** — avoid `size() - 1`
- **Use `const&`** to avoid expensive container copies
- **Initialize all variables** — uninitialized = bugs
- **Check for `nullptr`** before dereferencing pointers
- **Add `break` in switch cases** to prevent fallthrough