# C++ Interview Questions: Collections & STL Containers

> **80 curated questions with answers and code examples** — covering `vector`, `list`, `deque`, `array`, `map`, `set`, `unordered_map`, `unordered_set`, `stack`, `queue`, `priority_queue`, and STL algorithms.

---

## Table of Contents

1. [STL Fundamentals (Q1–Q7)](#stl-fundamentals)
2. [vector (Q8–Q19)](#vector)
3. [array (Q20–Q23)](#array)
4. [list & forward_list (Q24–Q29)](#list--forward_list)
5. [deque (Q30–Q33)](#deque)
6. [map & multimap (Q34–Q41)](#map--multimap)
7. [set & multiset (Q42–Q47)](#set--multiset)
8. [unordered_map & unordered_set (Q48–Q55)](#unordered_map--unordered_set)
9. [stack, queue & priority_queue (Q56–Q63)](#stack-queue--priority_queue)
10. [Iterators & Ranges (Q64–Q69)](#iterators--ranges)
11. [STL Algorithms (Q70–Q76)](#stl-algorithms)
12. [Container Selection & Advanced Topics (Q77–Q80)](#container-selection--advanced-topics)

---

## STL Fundamentals

---

### Q1. What is the STL and what are its main components?

**Answer:**  
The Standard Template Library (STL) is a collection of generic algorithms and data structures built into C++. Its three main pillars are:

- **Containers** — store collections of objects (`vector`, `map`, `set`, etc.)
- **Iterators** — uniform mechanism to traverse containers
- **Algorithms** — generic operations on ranges (`sort`, `find`, `transform`, etc.)

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 1, 4, 2};  // Container

    std::sort(v.begin(), v.end());           // Algorithm using Iterators

    for (int x : v)                          // Range-based for (iterator-backed)
        std::cout << x << " ";               // 1 2 3 4 5
}
```

---

### Q2. What are the container categories in the STL?

**Answer:**

| Category | Containers | Key Property |
|---|---|---|
| Sequence | `vector`, `deque`, `list`, `forward_list`, `array` | Ordered by insertion position |
| Associative | `map`, `multimap`, `set`, `multiset` | Sorted by key; O(log n) operations |
| Unordered Associative | `unordered_map`, `unordered_set`, `unordered_multimap`, `unordered_multiset` | Hash-based; O(1) average |
| Container Adaptors | `stack`, `queue`, `priority_queue` | Restricted interface over another container |

```cpp
#include <vector>       // sequence
#include <map>          // associative
#include <unordered_map>// unordered associative
#include <stack>        // adaptor

std::vector<int>              seq   = {1, 2, 3};
std::map<std::string, int>    assoc = {{"a", 1}};
std::unordered_map<int, char> hash  = {{1, 'A'}};
std::stack<int>               adapt;
```

---

### Q3. What is an iterator and what categories exist?

**Answer:**  
An iterator is an object that points to an element in a range and can be advanced/dereferenced. There are five categories, from most to least powerful:

| Category | Operations | Containers |
|---|---|---|
| Random Access | `+n`, `-n`, `[]`, comparison | `vector`, `deque`, `array` |
| Bidirectional | `++`, `--` | `list`, `map`, `set` |
| Forward | `++` only | `forward_list`, `unordered_map` |
| Input | Read-once `++` | `istream_iterator` |
| Output | Write-once `++` | `ostream_iterator` |

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

auto it = v.begin();    // Random access iterator
it += 2;                // jump forward 2 (only random access)
std::cout << *it;       // 3

++it;                   // increment
std::cout << *it;       // 4

--it;                   // decrement (bidirectional+)
std::cout << *it;       // 3

std::cout << v.end() - v.begin(); // distance: 5 (random access only)
```

---

### Q4. What is the difference between `size()` and `capacity()` in containers?

**Answer:**  
- `size()` — number of elements currently stored.
- `capacity()` — number of elements the container can hold before reallocating (only relevant for `vector` and `string`).

```cpp
std::vector<int> v;
std::cout << v.size();     // 0
std::cout << v.capacity(); // 0 (implementation-defined)

v.reserve(10);
std::cout << v.size();     // 0  — no elements yet
std::cout << v.capacity(); // 10 — memory allocated

v.push_back(1);
v.push_back(2);
std::cout << v.size();     // 2
std::cout << v.capacity(); // 10 — still 10, no reallocation
```

---

### Q5. What are `begin()`, `end()`, `cbegin()`, `rbegin()` and their variants?

**Answer:**

| Method | Returns | Direction | Mutates? |
|---|---|---|---|
| `begin()` / `end()` | mutable iterator | forward | yes |
| `cbegin()` / `cend()` | const iterator | forward | no |
| `rbegin()` / `rend()` | reverse mutable | backward | yes |
| `crbegin()` / `crend()` | reverse const | backward | no |

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// Forward
for (auto it = v.begin(); it != v.end(); ++it)
    std::cout << *it << " "; // 1 2 3 4 5

// Reverse
for (auto it = v.rbegin(); it != v.rend(); ++it)
    std::cout << *it << " "; // 5 4 3 2 1

// Const (read-only)
for (auto it = v.cbegin(); it != v.cend(); ++it)
    std::cout << *it << " "; // 1 2 3 4 5
// *it = 99; // Error: const iterator
```

---

### Q6. What does it mean for an iterator to be invalidated?

**Answer:**  
Invalidation means an iterator no longer safely refers to a valid element — using it is undefined behavior. Each container has different invalidation rules.

```cpp
std::vector<int> v = {1, 2, 3};
auto it = v.begin() + 1; // points to 2

v.push_back(4); // MAY reallocate — ALL iterators invalidated!
// std::cout << *it; // UB: it may be dangling

// Safe pattern: refresh iterator after modification
v.push_back(5);
it = v.begin() + 1; // re-obtain after mutation
std::cout << *it;    // 2 (safe)

// list iterators are NOT invalidated by insert/erase (except erased element)
std::list<int> lst = {1, 2, 3};
auto lit = std::next(lst.begin()); // points to 2
lst.push_back(4);    // lit still valid!
lst.push_front(0);   // lit still valid!
std::cout << *lit;   // 2 (safe)
```

---

### Q7. What is the difference between `emplace` and `insert`/`push_back`?

**Answer:**  
`insert`/`push_back` take an already-constructed object (may copy or move). `emplace`/`emplace_back` construct the object **in-place** inside the container using forwarded arguments — avoiding the extra copy/move.

```cpp
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {
        std::cout << "Constructed\n";
    }
    Point(const Point&) { std::cout << "Copied\n"; }
    Point(Point&&)      { std::cout << "Moved\n";  }
};

std::vector<Point> v;
v.reserve(4);

v.push_back(Point{1, 2});    // Constructed + Moved
v.emplace_back(3, 4);        // Constructed only (in-place)

std::map<int, Point> m;
m.emplace(1, Point{5, 6});          // Constructed + Moved
m.emplace(std::piecewise_construct, // Constructed in-place
          std::forward_as_tuple(2),
          std::forward_as_tuple(7, 8));
```

---

## vector

---

### Q8. What is `std::vector` and when should you use it?

**Answer:**  
`vector` is a dynamic array that stores elements contiguously in memory. It is the **default container choice** in C++ — fast random access, cache-friendly, and efficient amortized append.

Use `vector` when:
- You need indexed/random access (`O(1)`)
- You mostly append to the end
- Cache performance matters

```cpp
#include <vector>

std::vector<int> v = {10, 20, 30, 40, 50};

std::cout << v[2];      // 30 — O(1) random access
std::cout << v.at(2);   // 30 — bounds-checked
std::cout << v.front(); // 10
std::cout << v.back();  // 50
std::cout << v.size();  // 5

// Iteration
for (int x : v) std::cout << x << " ";
```

---

### Q9. How does `vector` grow when it runs out of capacity?

**Answer:**  
When `push_back` would exceed `capacity()`, the vector allocates a new larger block (typically 2× the current capacity), moves all elements to the new block, and releases the old one. This is `O(n)` for one reallocation, but amortized `O(1)` per push.

```cpp
std::vector<int> v;
for (int i = 0; i < 10; ++i) {
    v.push_back(i);
    std::cout << "size=" << v.size()
              << " cap=" << v.capacity() << "\n";
}
// Typical output shows capacity doubling: 1, 2, 4, 8, 16...

// Avoid reallocation with reserve:
std::vector<int> v2;
v2.reserve(100);        // allocate once
for (int i = 0; i < 100; ++i)
    v2.push_back(i);    // no reallocation
```

---

### Q10. What is the difference between `push_back` and `emplace_back`?

**Answer:**  
`push_back` takes a constructed value; `emplace_back` constructs in-place. For complex types, `emplace_back` is more efficient. For trivial types, they are identical.

```cpp
struct Employee {
    std::string name;
    int salary;
    Employee(std::string n, int s) : name(std::move(n)), salary(s) {}
};

std::vector<Employee> employees;
employees.reserve(3);

employees.push_back(Employee{"Alice", 90000}); // construct + move
employees.emplace_back("Bob", 85000);          // construct in-place
employees.emplace_back("Carol", 95000);        // construct in-place

for (auto& e : employees)
    std::cout << e.name << ": " << e.salary << "\n";
```

---

### Q11. What is `vector::reserve()` vs `vector::resize()`?

**Answer:**  
- `reserve(n)` — allocates memory for at least `n` elements without changing `size()`. No new elements created.
- `resize(n)` — changes `size()` to `n`. If growing, new elements are value-initialized. If shrinking, excess elements are destroyed.

```cpp
std::vector<int> v;

v.reserve(5);
std::cout << v.size();     // 0 — no elements
std::cout << v.capacity(); // >= 5

v.resize(5);
std::cout << v.size();     // 5 — 5 zero-initialized ints
std::cout << v[2];         // 0

v.resize(3);               // shrinks: elements 3,4 destroyed
std::cout << v.size();     // 3

v.resize(6, 99);           // grow, fill new elements with 99
std::cout << v[4];         // 99
```

---

### Q12. How do you insert and erase elements in a `vector`?

**Answer:**  
`insert` and `erase` operate at any position but are `O(n)` because elements must be shifted.

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// Insert at position
auto it = v.begin() + 2;
v.insert(it, 99);         // {1, 2, 99, 3, 4, 5}

// Insert multiple
v.insert(v.begin(), {10, 20}); // {10, 20, 1, 2, 99, 3, 4, 5}

// Erase single element
v.erase(v.begin() + 1);   // remove 20

// Erase range [first, last)
v.erase(v.begin(), v.begin() + 2); // remove first 2 elements

// Erase-remove idiom (remove all 99s)
v.insert(v.end(), {99, 7, 99});
v.erase(std::remove(v.begin(), v.end(), 99), v.end());
std::cout << v.size(); // no 99s remain
```

---

### Q13. What is the erase-remove idiom?

**Answer:**  
`std::remove` moves unwanted elements to the end (logically) and returns an iterator to the new logical end. `erase` then physically removes them. This is `O(n)` — the efficient way to remove by value from a `vector`.

```cpp
std::vector<int> v = {1, 4, 2, 4, 3, 4, 5};

// Remove all 4s
auto newEnd = std::remove(v.begin(), v.end(), 4);
// v is now {1, 2, 3, 5, ?, ?, ?} logically
v.erase(newEnd, v.end());
// v = {1, 2, 3, 5}

// One-liner
std::vector<int> v2 = {1, 2, 3, 4, 5, 6};
v2.erase(std::remove_if(v2.begin(), v2.end(),
    [](int x) { return x % 2 == 0; }), v2.end());
// v2 = {1, 3, 5} — all even numbers removed

// C++20 shorthand:
// std::erase(v2, 4);                     // remove by value
// std::erase_if(v2, [](int x){return x%2==0;}); // remove by predicate
```

---

### Q14. How does `vector` compare to a raw array in terms of performance?

**Answer:**  
For element access and iteration, `vector` performs identically to a raw array — both are contiguous, cache-friendly, and compile to equivalent machine code. The difference is in management overhead (only at modification points).

```cpp
// Both access patterns compile to virtually identical assembly:
int raw[1000];
std::vector<int> vec(1000);

// Identical performance:
for (int i = 0; i < 1000; ++i) raw[i] = i;
for (int i = 0; i < 1000; ++i) vec[i] = i;

// vector adds value: bounds checking with at(), dynamic sizing
try {
    vec.at(9999) = 1; // throws std::out_of_range
} catch (const std::out_of_range& e) {
    std::cout << e.what();
}
// raw[9999] = 1; // UB: no check
```

---

### Q15. What is `vector::shrink_to_fit()`?

**Answer:**  
`shrink_to_fit()` is a non-binding request to reduce `capacity()` to match `size()`. Useful to free excess memory after removing many elements.

```cpp
std::vector<int> v(1000, 0);
std::cout << v.capacity(); // 1000

v.resize(10);
std::cout << v.capacity(); // still 1000 (memory not freed)

v.shrink_to_fit();
std::cout << v.capacity(); // likely 10 (implementation may vary)

// Alternative swap trick (pre-C++11):
// std::vector<int>(v).swap(v);
```

---

### Q16. How do you safely access elements in a `vector`?

**Answer:**  
- `operator[]` — fast, no bounds check, UB if out of range.
- `at()` — bounds-checked, throws `std::out_of_range`.
- `front()` / `back()` — first/last element (UB if empty).
- `data()` — raw pointer to underlying array.

```cpp
std::vector<int> v = {10, 20, 30};

std::cout << v[1];      // 20 — no check
std::cout << v.at(1);   // 20 — checked
std::cout << v.front(); // 10
std::cout << v.back();  // 30
std::cout << *v.data(); // 10 — raw pointer

// Safe pattern for potentially empty vector:
if (!v.empty()) std::cout << v.front();

// C interop using data()
void cFunc(int* arr, int n);
cFunc(v.data(), static_cast<int>(v.size()));
```

---

### Q17. How do you sort and search a `vector`?

**Answer:**  
Use `std::sort` (O(n log n)) and `std::binary_search` / `std::lower_bound` (O(log n) on sorted range).

```cpp
std::vector<int> v = {5, 3, 8, 1, 9, 2, 7};

// Sort ascending
std::sort(v.begin(), v.end());
// v = {1, 2, 3, 5, 7, 8, 9}

// Sort descending
std::sort(v.begin(), v.end(), std::greater<int>());

// Sort with lambda
std::vector<std::string> words = {"banana", "apple", "cherry"};
std::sort(words.begin(), words.end(),
    [](const std::string& a, const std::string& b) {
        return a.length() < b.length(); // sort by length
    });

// Binary search (requires sorted range)
std::sort(v.begin(), v.end());
bool found = std::binary_search(v.begin(), v.end(), 5); // true

// Find insertion point
auto pos = std::lower_bound(v.begin(), v.end(), 5);
std::cout << std::distance(v.begin(), pos); // index of 5
```

---

### Q18. How do you copy and move a `vector`?

**Answer:**  
Copying is `O(n)` (deep copy of all elements). Moving is `O(1)` (ownership transfer — source becomes empty).

```cpp
std::vector<int> original = {1, 2, 3, 4, 5};

// Copy: independent duplicate
std::vector<int> copied = original;
copied[0] = 99;
std::cout << original[0]; // 1 — unchanged

// Move: transfer ownership
std::vector<int> moved = std::move(original);
std::cout << original.size(); // 0 — original is empty
std::cout << moved.size();    // 5

// Partial copy using iterators
std::vector<int> src = {10, 20, 30, 40, 50};
std::vector<int> sub(src.begin() + 1, src.begin() + 4);
// sub = {20, 30, 40}
```

---

### Q19. What is a 2D vector and how do you use it?

**Answer:**  
A 2D vector is a `vector<vector<T>>`. Each row is an independent vector (can have different sizes — jagged array). For fixed-size grids, prefer `vector<array<T,N>>` or a flattened `vector<T>`.

```cpp
// 3x4 grid initialized to 0
int rows = 3, cols = 4;
std::vector<std::vector<int>> grid(rows, std::vector<int>(cols, 0));

// Set values
grid[1][2] = 42;
std::cout << grid[1][2]; // 42

// Iterate
for (int r = 0; r < rows; ++r) {
    for (int c = 0; c < cols; ++c)
        std::cout << grid[r][c] << "\t";
    std::cout << "\n";
}

// Jagged (rows of different lengths)
std::vector<std::vector<int>> triangle;
for (int i = 1; i <= 4; ++i)
    triangle.push_back(std::vector<int>(i, i));
// Row 0: {1}
// Row 1: {2, 2}
// Row 2: {3, 3, 3}
```

---

## array

---

### Q20. What is `std::array` and how does it differ from a raw array?

**Answer:**  
`std::array<T, N>` is a fixed-size container wrapping a raw array. It has the same stack-allocated, zero-overhead performance but adds value semantics (copyable, assignable), STL compatibility, and bounds checking via `at()`.

```cpp
#include <array>

std::array<int, 5> arr = {1, 2, 3, 4, 5};

std::cout << arr[2];        // 3 — no bounds check
std::cout << arr.at(2);     // 3 — bounds-checked
std::cout << arr.size();    // 5 — always known at compile time
std::cout << arr.front();   // 1
std::cout << arr.back();    // 5

// Unlike raw arrays, std::array is copyable and assignable
std::array<int, 5> arr2 = arr; // deep copy
arr2[0] = 99;
std::cout << arr[0]; // 1 — unchanged
```

---

### Q21. What are the advantages of `std::array` over `std::vector`?

**Answer:**

| Feature | `std::array<T,N>` | `std::vector<T>` |
|---|---|---|
| Memory | Stack (no heap alloc) | Heap allocated |
| Size | Fixed at compile time | Dynamic |
| Overhead | Zero | Control block + capacity |
| `constexpr` | Yes | Partial (C++20) |
| Cache | Maximally friendly | Friendly (contiguous) |

```cpp
// array: stack allocation, zero overhead
constexpr std::array<int, 3> rgb = {255, 128, 0}; // constexpr!
static_assert(rgb[0] == 255);                       // compile-time check

// Perfect for fixed-size lookup tables
constexpr std::array<const char*, 7> days = {
    "Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"
};
std::cout << days[4]; // "Fri"

// Works with all STL algorithms
std::array<int, 5> a = {5, 3, 1, 4, 2};
std::sort(a.begin(), a.end());
// a = {1, 2, 3, 4, 5}
```

---

### Q22. How do you pass `std::array` to a function?

**Answer:**  
Pass by reference to avoid copying. The size is part of the type, so templates are needed for generic functions.

```cpp
// Fixed size: pass by const reference
void printArray(const std::array<int, 5>& arr) {
    for (int x : arr) std::cout << x << " ";
}

// Generic: template on size
template<std::size_t N>
void printAny(const std::array<int, N>& arr) {
    for (int x : arr) std::cout << x << " ";
}

// Even more generic: template on type and size
template<typename T, std::size_t N>
T sumArray(const std::array<T, N>& arr) {
    T total{};
    for (const auto& x : arr) total += x;
    return total;
}

std::array<int, 5>    a5  = {1, 2, 3, 4, 5};
std::array<double, 3> a3d = {1.1, 2.2, 3.3};

printAny(a5);
std::cout << sumArray(a5);  // 15
std::cout << sumArray(a3d); // 6.6
```

---

### Q23. What is structured binding with `std::array`?

**Answer:**  
C++17 structured bindings let you unpack array elements into named variables.

```cpp
std::array<int, 3> rgb = {255, 128, 0};
auto [r, g, b] = rgb;
std::cout << r << ", " << g << ", " << b; // 255, 128, 0

// Works with pairs and tuples too
std::array<std::string, 2> kv = {"name", "Alice"};
auto [key, value] = kv;
std::cout << key << ": " << value; // name: Alice

// Modification through reference
auto& [fr, fg, fb] = rgb;
fr = 200;
std::cout << rgb[0]; // 200
```

---

## list & forward_list

---

### Q24. What is `std::list` and when should you use it?

**Answer:**  
`std::list` is a doubly linked list. Elements are not contiguous — each node holds data plus prev/next pointers. Use it when you need `O(1)` insertion/deletion anywhere and don't need random access.

```cpp
#include <list>

std::list<int> lst = {1, 2, 3, 4, 5};

// O(1) insert at front or back
lst.push_front(0);
lst.push_back(6);
// {0, 1, 2, 3, 4, 5, 6}

// O(1) erase given an iterator
auto it = std::next(lst.begin(), 3); // points to 3
lst.erase(it);                       // O(1)
// {0, 1, 2, 4, 5, 6}

// Insert before an iterator
it = std::next(lst.begin(), 2);      // points to 2
lst.insert(it, 99);
// {0, 1, 99, 2, 4, 5, 6}

// NO random access: lst[3] is a compile error
```

---

### Q25. What are the key differences between `vector` and `list`?

**Answer:**

| Feature | `vector` | `list` |
|---|---|---|
| Memory layout | Contiguous | Node-based (scattered) |
| Random access | O(1) | O(n) |
| Insert/erase at front | O(n) | O(1) |
| Insert/erase at middle | O(n) | O(1) (with iterator) |
| Insert/erase at back | O(1) amortized | O(1) |
| Cache performance | Excellent | Poor |
| Iterator invalidation | On reallocation | Never (except erased) |
| Memory overhead | Low | High (2 pointers per node) |

```cpp
// list shines: frequent insert/erase in the middle
std::list<int> lst = {1, 2, 3, 4, 5};
auto mid = std::next(lst.begin(), 2); // O(n) to find, O(1) to insert
lst.insert(mid, 99); // O(1) — no shifting!

// vector faster in practice for small N (cache effects dominate)
// Benchmark before choosing list over vector!
```

---

### Q26. What special operations does `std::list` offer?

**Answer:**  
`list` has exclusive member functions that exploit its linked structure.

```cpp
std::list<int> a = {1, 3, 5};
std::list<int> b = {2, 4, 6};

// splice: move elements from b into a at position (O(1)!)
auto it = std::next(a.begin()); // points to 3
a.splice(it, b);                // moves all of b before 3
// a = {1, 2, 4, 6, 3, 5}, b = {}

// merge: merge two sorted lists (O(n))
std::list<int> x = {1, 3, 5};
std::list<int> y = {2, 4, 6};
x.merge(y);     // x = {1, 2, 3, 4, 5, 6}, y = {}

// sort: O(n log n) stable sort
std::list<int> unsorted = {5, 2, 8, 1, 9};
unsorted.sort();  // {1, 2, 5, 8, 9}

// remove: erase all matching values (O(n))
unsorted.remove(5);

// unique: remove consecutive duplicates
std::list<int> dups = {1, 1, 2, 3, 3, 3, 4};
dups.unique(); // {1, 2, 3, 4}
```

---

### Q27. What is `std::forward_list`?

**Answer:**  
`forward_list` is a singly linked list — nodes have only a `next` pointer (no `prev`). It uses less memory than `list` but only supports forward iteration and `insert_after`/`erase_after`.

```cpp
#include <forward_list>

std::forward_list<int> fl = {1, 2, 3, 4, 5};

// Only forward iteration
for (int x : fl) std::cout << x << " "; // 1 2 3 4 5

// insert_after (no insert_before — no prev pointer)
auto it = fl.before_begin(); // special iterator before first element
fl.insert_after(it, 0);     // {0, 1, 2, 3, 4, 5}

// erase_after
it = fl.begin();             // points to 0
fl.erase_after(it);          // removes 1: {0, 2, 3, 4, 5}

// No size() method (would be O(n) — omitted by design)
// No push_back (no tail pointer)
fl.push_front(99);           // O(1)
```

---

### Q28. When would you choose `list` over `vector` in practice?

**Answer:**  
In practice, `vector` wins most benchmarks due to cache effects even when `list` has better algorithmic complexity. Prefer `list` when:

- You store large objects and need stable iterators/pointers after insert/erase.
- You do `splice` operations (moving sub-ranges between lists in O(1)).
- Your profiler confirms `vector` is too slow for your insert/erase pattern.

```cpp
// Classic use case: LRU Cache — move recently used to front (O(1))
class LRUCache {
    std::list<std::pair<int,int>> items; // {key, value}
    std::unordered_map<int, std::list<std::pair<int,int>>::iterator> map;
    int capacity;
public:
    LRUCache(int cap) : capacity(cap) {}

    int get(int key) {
        if (!map.count(key)) return -1;
        items.splice(items.begin(), items, map[key]); // O(1) move to front
        return items.front().second;
    }

    void put(int key, int value) {
        if (map.count(key)) items.erase(map[key]);
        else if ((int)items.size() == capacity) {
            map.erase(items.back().first);
            items.pop_back();
        }
        items.push_front({key, value});
        map[key] = items.begin();
    }
};
```

---

### Q29. What is iterator invalidation in `list` vs `vector`?

**Answer:**  
`list` iterators are extremely stable — insertion/deletion anywhere never invalidates any iterator except the erased element's own iterator. `vector` iterators are fragile — any reallocation invalidates all.

```cpp
// vector: invalidation on reallocation
std::vector<int> v = {1, 2, 3};
auto vit = v.begin() + 1;       // points to 2
v.push_back(4);                  // may reallocate — vit INVALID!

// list: stable iterators
std::list<int> lst = {1, 2, 3};
auto lit = std::next(lst.begin()); // points to 2
lst.push_back(4);    // lit still valid: points to 2
lst.push_front(0);   // lit still valid: points to 2
lst.erase(lst.begin()); // lit still valid: points to 2
std::cout << *lit;   // 2 — safe!
```

---

## deque

---

### Q30. What is `std::deque` and how does it work internally?

**Answer:**  
`deque` (double-ended queue) supports O(1) push/pop at **both** front and back. Internally it's a segmented array — a collection of fixed-size chunks with a map of pointers to chunks. This gives O(1) front operations that `vector` can't provide cheaply.

```cpp
#include <deque>

std::deque<int> dq = {3, 4, 5};

dq.push_front(2);   // O(1)
dq.push_front(1);   // O(1)
dq.push_back(6);    // O(1)
dq.push_back(7);    // O(1)
// {1, 2, 3, 4, 5, 6, 7}

std::cout << dq[3];  // 4 — O(1) random access
dq.pop_front();      // O(1): removes 1
dq.pop_back();       // O(1): removes 7

for (int x : dq) std::cout << x << " "; // 2 3 4 5 6
```

---

### Q31. How does `deque` compare to `vector` and `list`?

**Answer:**

| Feature | `vector` | `deque` | `list` |
|---|---|---|---|
| Random access | O(1) | O(1) | O(n) |
| Push/pop back | O(1) amortized | O(1) | O(1) |
| Push/pop front | O(n) | O(1) | O(1) |
| Contiguous memory | Yes | No (segmented) | No |
| Cache performance | Best | Good | Poor |
| Iterator stability | Fragile | Fragile | Stable |

```cpp
// deque is the backing container for std::stack and std::queue by default
std::stack<int>  s;  // uses deque internally
std::queue<int>  q;  // uses deque internally

// Good use case: sliding window, BFS queue with many push_front/push_back
std::deque<int> window;
std::vector<int> data = {1,3,2,5,4,6,7};
int k = 3;
for (int i = 0; i < (int)data.size(); ++i) {
    while (!window.empty() && window.front() < i - k + 1)
        window.pop_front();
    while (!window.empty() && data[window.back()] < data[i])
        window.pop_back();
    window.push_back(i);
    if (i >= k-1) std::cout << data[window.front()] << " ";
}
// Sliding window maximum
```

---

### Q32. What are the iterator invalidation rules for `deque`?

**Answer:**  
`deque` has complex invalidation rules:
- Push/pop at **front or back**: all iterators are invalidated, but references/pointers to elements remain valid.
- Insert/erase **in the middle**: all iterators, references, and pointers are invalidated.

```cpp
std::deque<int> dq = {1, 2, 3, 4, 5};
auto it = dq.begin() + 2; // points to 3
int& ref = dq[2];          // reference to 3

dq.push_back(6);   // it invalidated, but ref is still valid
dq.push_front(0);  // it invalidated, but ref is still valid
std::cout << ref;  // 3 — still valid

// After insert in middle: both it and ref invalidated
dq.insert(dq.begin() + 1, 99); // all bets off
```

---

### Q33. When should you use `deque` instead of `vector`?

**Answer:**  
Use `deque` when you need frequent O(1) insertions at both ends. Common scenarios: sliding windows, work queues, undo/redo stacks with access at both ends.

```cpp
// Palindrome checker using deque
bool isPalindrome(const std::string& s) {
    std::deque<char> dq(s.begin(), s.end());
    while (dq.size() > 1) {
        if (dq.front() != dq.back()) return false;
        dq.pop_front();
        dq.pop_back();
    }
    return true;
}

std::cout << isPalindrome("racecar"); // true
std::cout << isPalindrome("hello");   // false
```

---

## map & multimap

---

### Q34. What is `std::map` and how does it work internally?

**Answer:**  
`std::map<Key, Value>` is an ordered associative container storing key-value pairs with unique keys. Internally it's a **Red-Black Tree** (self-balancing BST). All operations are O(log n). Keys are always sorted.

```cpp
#include <map>

std::map<std::string, int> scores;

// Insert
scores["Alice"] = 95;
scores["Bob"]   = 87;
scores.insert({"Carol", 91});
scores.emplace("Dave", 78);

// Access
std::cout << scores["Alice"];       // 95
std::cout << scores.at("Bob");      // 87 — bounds-checked
// scores["Eve"] creates Eve with value 0! (operator[] inserts)

// Iteration: always in sorted key order
for (auto& [name, score] : scores)  // structured binding
    std::cout << name << ": " << score << "\n";
// Alice: 95, Bob: 87, Carol: 91, Dave: 78

std::cout << scores.size(); // 4
```

---

### Q35. What is the difference between `map::operator[]` and `map::at()`?

**Answer:**  
`operator[]` inserts a default-constructed value if the key doesn't exist — modifies the map even on read! `at()` throws `std::out_of_range` if the key is missing — safe for read-only access.

```cpp
std::map<std::string, int> m = {{"a", 1}, {"b", 2}};

// operator[]: dangerous for reading
std::cout << m["c"];     // inserts "c"→0, returns 0
std::cout << m.size();   // 3 (!) — "c" was inserted

// at(): safe for reading
try {
    std::cout << m.at("z"); // throws std::out_of_range
} catch (const std::out_of_range& e) {
    std::cout << "Key not found\n";
}

// Check before access:
if (m.count("a"))       std::cout << m.at("a"); // 1
if (m.contains("b"))    std::cout << m.at("b"); // 2 (C++20)
auto it = m.find("b");
if (it != m.end())      std::cout << it->second; // 2
```

---

### Q36. What is `map::find()`, `count()`, and `contains()` (C++20)?

**Answer:**

```cpp
std::map<int, std::string> m = {{1,"one"},{2,"two"},{3,"three"}};

// find(): returns iterator to element, or end() if not found
auto it = m.find(2);
if (it != m.end())
    std::cout << it->first << " -> " << it->second; // 2 -> two

// count(): 0 or 1 for map (unique keys)
std::cout << m.count(2);  // 1
std::cout << m.count(99); // 0

// contains() (C++20): cleaner bool check
std::cout << m.contains(3);  // true
std::cout << m.contains(99); // false
```

---

### Q37. How do you erase elements from a `map`?

**Answer:**  
Three overloads: by key, by iterator, or by range. All are O(log n) by key, O(1) by iterator.

```cpp
std::map<int, std::string> m = {{1,"a"},{2,"b"},{3,"c"},{4,"d"},{5,"e"}};

// Erase by key — returns number of erased elements (0 or 1)
m.erase(3);           // removes key 3
std::cout << m.size(); // 4

// Erase by iterator — O(1) amortized
auto it = m.find(2);
if (it != m.end()) m.erase(it);

// Erase range [first, last)
m.erase(m.begin(), m.end()); // clear all
std::cout << m.empty();       // true

// Erase while iterating (safe pattern):
std::map<int,int> data = {{1,10},{2,20},{3,30},{4,40}};
for (auto it = data.begin(); it != data.end(); ) {
    if (it->second > 20)
        it = data.erase(it); // erase returns next valid iterator
    else
        ++it;
}
// data = {{1,10},{2,20}}
```

---

### Q38. What are `lower_bound`, `upper_bound`, and `equal_range` on a map?

**Answer:**  
These ordered-search functions exploit the sorted nature of `map`:
- `lower_bound(k)` — first element with key `>= k`
- `upper_bound(k)` — first element with key `> k`
- `equal_range(k)` — pair of [lower_bound, upper_bound]

```cpp
std::map<int, std::string> m = {{1,"a"},{3,"c"},{5,"e"},{7,"g"},{9,"i"}};

auto lb = m.lower_bound(4); // first key >= 4 → key 5
auto ub = m.upper_bound(6); // first key >  6 → key 7

std::cout << lb->first; // 5
std::cout << ub->first; // 7

// Iterate over range [4, 8)
for (auto it = m.lower_bound(4); it != m.upper_bound(8); ++it)
    std::cout << it->first << " "; // 5 7

// equal_range for point lookup
auto [lo, hi] = m.equal_range(5);
if (lo != hi) std::cout << lo->second; // "e"
```

---

### Q39. What is `std::multimap`?

**Answer:**  
`multimap` allows **multiple entries with the same key**. `operator[]` is NOT available. Use `equal_range` to find all values for a key.

```cpp
#include <map>

std::multimap<std::string, int> grades;
grades.insert({"Alice", 90});
grades.insert({"Alice", 85});
grades.insert({"Bob",   78});
grades.insert({"Alice", 92});

std::cout << grades.count("Alice"); // 3

// All of Alice's grades:
auto [lo, hi] = grades.equal_range("Alice");
for (auto it = lo; it != hi; ++it)
    std::cout << it->second << " "; // 90 85 92 (insertion order within key)
```

---

### Q40. How do you use a custom comparator with `map`?

**Answer:**  
Provide a comparison functor or lambda as the third template argument.

```cpp
// Case-insensitive map
struct CaseInsensitive {
    bool operator()(const std::string& a, const std::string& b) const {
        return std::lexicographical_compare(
            a.begin(), a.end(), b.begin(), b.end(),
            [](char ca, char cb) {
                return std::tolower(ca) < std::tolower(cb);
            });
    }
};

std::map<std::string, int, CaseInsensitive> m;
m["Hello"] = 1;
m["HELLO"] = 2; // same key as "Hello"!
std::cout << m.size();      // 1
std::cout << m["hello"];    // 2

// Lambda comparator (C++20 or use decltype trick)
auto cmp = [](int a, int b) { return a > b; }; // descending
std::map<int, std::string, decltype(cmp)> descMap(cmp);
descMap[3] = "c"; descMap[1] = "a"; descMap[2] = "b";
for (auto& [k, v] : descMap)
    std::cout << k << " "; // 3 2 1
```

---

### Q41. What is `std::map::insert_or_assign` and `try_emplace` (C++17)?

**Answer:**  
- `insert_or_assign(key, value)` — inserts if key absent, assigns if present (unlike `operator[]` which default-constructs).
- `try_emplace(key, args...)` — inserts only if key absent; does NOT construct value if key exists (avoids unnecessary construction).

```cpp
std::map<std::string, int> m;

// insert_or_assign: always sets the value
auto [it1, inserted1] = m.insert_or_assign("x", 10);
std::cout << inserted1; // true (inserted)
auto [it2, inserted2] = m.insert_or_assign("x", 99);
std::cout << inserted2; // false (assigned, not inserted)
std::cout << m["x"];    // 99

// try_emplace: only inserts if absent
auto [it3, ok3] = m.try_emplace("y", 42);
std::cout << ok3;       // true (inserted)
auto [it4, ok4] = m.try_emplace("y", 999); // y already exists
std::cout << ok4;       // false (NOT inserted, value unchanged)
std::cout << m["y"];    // 42
```

---

## set & multiset

---

### Q42. What is `std::set` and when do you use it?

**Answer:**  
`set<T>` stores unique, sorted elements. Internally a Red-Black Tree — all operations O(log n). Use it when you need sorted unique elements and fast membership testing.

```cpp
#include <set>

std::set<int> s = {5, 3, 1, 4, 2, 3, 1}; // duplicates ignored
// s = {1, 2, 3, 4, 5} — sorted, unique

std::cout << s.size();        // 5
std::cout << s.count(3);      // 1 (present)
std::cout << s.count(99);     // 0 (absent)
std::cout << s.contains(4);   // true (C++20)

s.insert(6);
s.erase(1);
// s = {2, 3, 4, 5, 6}

for (int x : s) std::cout << x << " "; // 2 3 4 5 6
```

---

### Q43. How does `set` differ from `map`?

**Answer:**  
`set` stores keys only; `map` stores key-value pairs. Both are ordered, unique-key Red-Black Trees.

```cpp
// set: unique keys, no associated value
std::set<std::string> visited;
visited.insert("page1");
visited.insert("page2");
if (!visited.count("page3")) {
    visited.insert("page3");
    // process page3
}

// map: key → value mapping
std::map<std::string, int> wordCount;
std::string word;
// while (stream >> word) wordCount[word]++;
// map stores {word → count}

// set<pair>: simulate set with auxiliary data
std::set<std::pair<int, std::string>> ranked;
ranked.insert({1, "Gold"});
ranked.insert({3, "Bronze"});
ranked.insert({2, "Silver"});
for (auto& [rank, medal] : ranked)
    std::cout << rank << ": " << medal << "\n"; // sorted by rank
```

---

### Q44. What is `std::multiset`?

**Answer:**  
`multiset` stores sorted elements and **allows duplicates**. `equal_range` retrieves all equal elements.

```cpp
std::multiset<int> ms = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};
// Sorted: {1, 1, 2, 3, 3, 4, 5, 5, 5, 6, 9}

std::cout << ms.count(5);  // 3

auto [lo, hi] = ms.equal_range(5);
for (auto it = lo; it != hi; ++it)
    std::cout << *it << " "; // 5 5 5

ms.erase(ms.find(1)); // erase ONE occurrence of 1
std::cout << ms.count(1); // 1 (not 0!)

ms.erase(5); // erases ALL occurrences of 5
std::cout << ms.count(5); // 0
```

---

### Q45. How do you use `set` for ordered deduplication?

**Answer:**

```cpp
std::vector<int> data = {5, 3, 1, 4, 2, 3, 5, 1, 2};

// Option 1: sort + unique + erase
std::vector<int> v = data;
std::sort(v.begin(), v.end());
v.erase(std::unique(v.begin(), v.end()), v.end());
// v = {1, 2, 3, 4, 5}

// Option 2: set (auto-sorts and deduplicates)
std::set<int> s(data.begin(), data.end());
// s = {1, 2, 3, 4, 5}

// Convert back to vector
std::vector<int> deduped(s.begin(), s.end());
// deduped = {1, 2, 3, 4, 5}

// Preserve insertion order (unordered approach):
std::vector<int> ordered;
std::unordered_set<int> seen;
for (int x : data)
    if (seen.insert(x).second) ordered.push_back(x);
// ordered = {5, 3, 1, 4, 2}
```

---

### Q46. How do you use a custom comparator with `set`?

**Answer:**

```cpp
struct Point {
    int x, y;
};

// Compare points by distance from origin
struct ByDistance {
    bool operator()(const Point& a, const Point& b) const {
        return (a.x*a.x + a.y*a.y) < (b.x*b.x + b.y*b.y);
    }
};

std::set<Point, ByDistance> points;
points.insert({3, 4}); // distance 5
points.insert({1, 1}); // distance ~1.41
points.insert({5, 0}); // distance 5 — same as {3,4}!

std::cout << points.size(); // 2 — {3,4} and {5,0} treated equal by comparator!

// Reverse sorted set
std::set<int, std::greater<int>> descSet = {1, 5, 3, 2, 4};
for (int x : descSet) std::cout << x << " "; // 5 4 3 2 1
```

---

### Q47. What are `lower_bound` and `upper_bound` on `set`?

**Answer:**  
Same semantics as `map`: exploit sorted order for O(log n) range queries.

```cpp
std::set<int> s = {1, 3, 5, 7, 9, 11};

auto lb = s.lower_bound(5);  // first >= 5 → 5
auto ub = s.upper_bound(8);  // first >  8 → 9

std::cout << *lb; // 5
std::cout << *ub; // 9

// All elements in range [4, 9]
for (auto it = s.lower_bound(4); it != s.upper_bound(9); ++it)
    std::cout << *it << " "; // 5 7 9

// Count elements in range [3, 8]
auto lo = s.lower_bound(3);
auto hi = s.upper_bound(8);
std::cout << std::distance(lo, hi); // 3 (elements: 3, 5, 7)
```

---

## unordered_map & unordered_set

---

### Q48. What is `std::unordered_map` and how does it work?

**Answer:**  
`unordered_map<Key, Value>` is a hash table providing **average O(1)** insert, erase, and lookup (O(n) worst case on hash collision). Elements are not stored in sorted order.

```cpp
#include <unordered_map>

std::unordered_map<std::string, int> freq;

// Count word frequencies
std::vector<std::string> words = {"the","cat","sat","on","the","mat","the"};
for (const auto& w : words) freq[w]++;

// Access
std::cout << freq["the"];  // 3

// Iteration (no guaranteed order)
for (auto& [word, count] : freq)
    std::cout << word << ": " << count << "\n";

std::cout << freq.count("cat");    // 1
std::cout << freq.count("dog");    // 0
std::cout << freq.contains("sat"); // true (C++20)
```

---

### Q49. What is the difference between `map` and `unordered_map`?

**Answer:**

| Feature | `map` | `unordered_map` |
|---|---|---|
| Internal structure | Red-Black Tree | Hash Table |
| Ordering | Keys sorted | No order |
| Lookup | O(log n) | O(1) average |
| Worst case | O(log n) | O(n) |
| Memory | Moderate | Higher (load factor) |
| Custom key | Comparator | Hash function |
| Iterator stability | Stable | Stable (no rehash) |

```cpp
// Use map when: you need sorted order or range queries
std::map<int, std::string> sorted;     // always sorted by key

// Use unordered_map when: you need maximum lookup speed, order doesn't matter
std::unordered_map<int, std::string> fast; // O(1) average lookup

// Benchmark: unordered_map is typically 3-5× faster for lookup
```

---

### Q50. How do you use a custom hash for `unordered_map`?

**Answer:**  
Provide a hash functor as the third template argument.

```cpp
struct Point { int x, y; };

// Custom hash
struct PointHash {
    std::size_t operator()(const Point& p) const {
        std::size_t hx = std::hash<int>{}(p.x);
        std::size_t hy = std::hash<int>{}(p.y);
        return hx ^ (hy << 32 | hy >> 32); // combine hashes
    }
};

struct PointEqual {
    bool operator()(const Point& a, const Point& b) const {
        return a.x == b.x && a.y == b.y;
    }
};

std::unordered_map<Point, std::string, PointHash, PointEqual> grid;
grid[{0, 0}] = "origin";
grid[{1, 0}] = "right";
grid[{0, 1}] = "up";

std::cout << grid[{0, 0}]; // "origin"
```

---

### Q51. What is load factor and rehashing in `unordered_map`?

**Answer:**  
The load factor is `size / bucket_count`. When it exceeds `max_load_factor()` (default 1.0), the table rehashes — allocates more buckets and reinserts all elements (O(n)). Use `reserve()` to avoid rehashing.

```cpp
std::unordered_map<int, int> m;

std::cout << m.max_load_factor(); // 1.0 (default)
std::cout << m.bucket_count();    // implementation-defined

// Pre-size to avoid rehashing for 1000 elements
m.reserve(1000); // ensures bucket_count >= 1000 / max_load_factor

for (int i = 0; i < 1000; ++i)
    m[i] = i * 2; // no rehashing!

std::cout << m.load_factor(); // < 1.0
std::cout << m.bucket_count(); // >= 1000
```

---

### Q52. What is `std::unordered_set`?

**Answer:**  
`unordered_set<T>` is a hash-based set: unique elements, O(1) average lookup, no sorted order.

```cpp
#include <unordered_set>

std::unordered_set<int> seen;

// Classic "two-sum" problem
std::vector<int> nums = {2, 7, 11, 15};
int target = 9;
for (int n : nums) {
    if (seen.count(target - n)) {
        std::cout << "Found pair!\n"; // 2 + 7 = 9
        break;
    }
    seen.insert(n);
}

// Fast deduplication preserving some order
std::vector<int> data = {3, 1, 4, 1, 5, 9, 2, 6, 5};
std::unordered_set<int> us(data.begin(), data.end());
std::cout << us.size(); // 7 unique elements
```

---

### Q53. What are the hash collision strategies in `unordered_map`?

**Answer:**  
The C++ standard uses **separate chaining** (each bucket is a linked list of colliding elements). Worst case (all keys hash to one bucket) degrades to O(n).

```cpp
std::unordered_map<int, int> m;
m.reserve(8);
std::cout << m.bucket_count(); // e.g., 8

// See which bucket each key falls into
m[1] = 10; m[9] = 90; m[17] = 170; // all hash to same bucket if table size is 8!

for (auto& [k, v] : m)
    std::cout << "key=" << k << " bucket=" << m.bucket(k) << "\n";

// Bucket contents
std::size_t b = m.bucket(1);
std::cout << "Bucket " << b << " has " << m.bucket_size(b) << " elements\n";
```

---

### Q54. How do you handle `unordered_map` with string keys efficiently?

**Answer:**  
`std::string` has a built-in hash. For heterogeneous lookup (C++20), use `std::string_view` to avoid constructing a `string` just for lookup.

```cpp
// Standard string key
std::unordered_map<std::string, int> m;
m["hello"] = 1;

// Lookup with string_view (avoids allocation in C++20)
struct StringHash {
    using is_transparent = void; // enables heterogeneous lookup
    std::size_t operator()(std::string_view sv) const {
        return std::hash<std::string_view>{}(sv);
    }
};
struct StringEqual {
    using is_transparent = void;
    bool operator()(std::string_view a, std::string_view b) const {
        return a == b;
    }
};

std::unordered_map<std::string, int, StringHash, StringEqual> fastMap;
fastMap["hello"] = 42;

// Lookup with string_view — no heap allocation!
std::string_view key = "hello";
auto it = fastMap.find(key); // O(1) without constructing std::string
std::cout << it->second;     // 42
```

---

### Q55. What are `unordered_multimap` and `unordered_multiset`?

**Answer:**  
Unordered variants that allow duplicate keys. Same hash-table mechanics but multiple entries per key.

```cpp
std::unordered_multimap<std::string, int> index;
index.insert({"apple", 1});
index.insert({"apple", 5});
index.insert({"banana", 3});
index.insert({"apple", 9});

std::cout << index.count("apple"); // 3

// All values for "apple"
auto [lo, hi] = index.equal_range("apple");
for (auto it = lo; it != hi; ++it)
    std::cout << it->second << " "; // 1 5 9 (order unspecified)
```

---

## stack, queue & priority_queue

---

### Q56. What is `std::stack`?

**Answer:**  
`stack` is a LIFO (Last In First Out) container adaptor. By default backed by `deque`; can be backed by `vector` or `list`.

```cpp
#include <stack>

std::stack<int> s;

s.push(1); s.push(2); s.push(3);

std::cout << s.top();  // 3 — peek without removing
s.pop();               // removes 3
std::cout << s.top();  // 2
std::cout << s.size(); // 2
std::cout << s.empty(); // false

// Classic use: balanced parentheses checker
bool isBalanced(const std::string& expr) {
    std::stack<char> stk;
    for (char c : expr) {
        if (c == '(' || c == '[' || c == '{') stk.push(c);
        else if (c == ')' || c == ']' || c == '}') {
            if (stk.empty()) return false;
            char top = stk.top(); stk.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{')) return false;
        }
    }
    return stk.empty();
}
std::cout << isBalanced("{[()]}"); // true
std::cout << isBalanced("{[(])}"); // false
```

---

### Q57. What is `std::queue`?

**Answer:**  
`queue` is a FIFO (First In First Out) container adaptor. Backed by `deque` by default.

```cpp
#include <queue>

std::queue<std::string> q;

q.push("first");
q.push("second");
q.push("third");

std::cout << q.front(); // "first"
std::cout << q.back();  // "third"
std::cout << q.size();  // 3

q.pop();                // removes "first"
std::cout << q.front(); // "second"

// BFS using queue
auto bfs = [](int start, const std::vector<std::vector<int>>& adj) {
    std::queue<int> q;
    std::vector<bool> visited(adj.size(), false);
    q.push(start);
    visited[start] = true;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        std::cout << node << " ";
        for (int neighbor : adj[node])
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
    }
};
```

---

### Q58. What is `std::priority_queue`?

**Answer:**  
`priority_queue` is a max-heap by default — `top()` always returns the largest element. Backed by `vector` using `std::make_heap` internally.

```cpp
#include <queue>

// Max-heap (default)
std::priority_queue<int> maxPQ;
maxPQ.push(3); maxPQ.push(1); maxPQ.push(4); maxPQ.push(1); maxPQ.push(5);

while (!maxPQ.empty()) {
    std::cout << maxPQ.top() << " "; // 5 4 3 1 1
    maxPQ.pop();
}

// Min-heap
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;
minPQ.push(3); minPQ.push(1); minPQ.push(4);
std::cout << minPQ.top(); // 1

// Custom comparator
struct Task { int priority; std::string name; };
auto cmp = [](const Task& a, const Task& b) {
    return a.priority < b.priority; // higher priority = top
};
std::priority_queue<Task, std::vector<Task>, decltype(cmp)> taskQueue(cmp);
taskQueue.push({5, "Critical"});
taskQueue.push({1, "Low"});
taskQueue.push({3, "Medium"});
std::cout << taskQueue.top().name; // "Critical"
```

---

### Q59. How do you implement Dijkstra's algorithm with `priority_queue`?

**Answer:**

```cpp
#include <queue>
#include <vector>
#include <limits>

using Edge = std::pair<int,int>; // {weight, node}
using Graph = std::vector<std::vector<Edge>>;

std::vector<int> dijkstra(const Graph& g, int src) {
    int n = g.size();
    std::vector<int> dist(n, std::numeric_limits<int>::max());
    std::priority_queue<Edge, std::vector<Edge>, std::greater<Edge>> pq;

    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue; // stale entry

        for (auto [w, v] : g[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

---

### Q60. What is the difference between `stack`, `queue`, and `priority_queue`?

**Answer:**

| Adaptor | Order | Top/Access | Use Case |
|---|---|---|---|
| `stack` | LIFO | `top()` — last inserted | DFS, undo, parsing |
| `queue` | FIFO | `front()` — first inserted | BFS, task queues |
| `priority_queue` | By priority | `top()` — max (or custom) | Dijkstra, scheduling |

```cpp
std::stack<int> s;
s.push(1); s.push(2); s.push(3);
std::cout << s.top(); // 3 — LIFO

std::queue<int> q;
q.push(1); q.push(2); q.push(3);
std::cout << q.front(); // 1 — FIFO

std::priority_queue<int> pq;
pq.push(1); pq.push(3); pq.push(2);
std::cout << pq.top(); // 3 — max first
```

---

### Q61. How do you use `stack` with a different underlying container?

**Answer:**

```cpp
// Default: deque backing
std::stack<int> s1;                              // uses deque

// Vector backing (better cache, no push_front)
std::stack<int, std::vector<int>> s2;
s2.push(1); s2.push(2);
std::cout << s2.top(); // 2

// List backing (stable pointers, more memory overhead)
std::stack<int, std::list<int>> s3;
s3.push(10);
std::cout << s3.top(); // 10
```

---

### Q62. What is a monotonic stack and when is it used?

**Answer:**  
A monotonic stack maintains elements in monotonically increasing or decreasing order. Used for "next greater element", histogram area, and similar problems.

```cpp
// Next Greater Element for each position
std::vector<int> nextGreater(const std::vector<int>& arr) {
    int n = arr.size();
    std::vector<int> result(n, -1);
    std::stack<int> stk; // stores indices

    for (int i = 0; i < n; ++i) {
        // Pop elements smaller than current
        while (!stk.empty() && arr[stk.top()] < arr[i]) {
            result[stk.top()] = arr[i];
            stk.pop();
        }
        stk.push(i);
    }
    return result;
}

auto res = nextGreater({4, 5, 2, 10, 8});
for (int x : res) std::cout << x << " "; // 5 10 10 -1 -1
```

---

### Q63. How do you implement a min-stack (stack with O(1) getMin)?

**Answer:**

```cpp
class MinStack {
    std::stack<int> data;
    std::stack<int> minTracker;
public:
    void push(int val) {
        data.push(val);
        if (minTracker.empty() || val <= minTracker.top())
            minTracker.push(val);
    }
    void pop() {
        if (data.top() == minTracker.top())
            minTracker.pop();
        data.pop();
    }
    int top()    const { return data.top(); }
    int getMin() const { return minTracker.top(); } // O(1)!
};

MinStack ms;
ms.push(3); ms.push(5); ms.push(1); ms.push(2);
std::cout << ms.getMin(); // 1
ms.pop();
ms.pop();
std::cout << ms.getMin(); // 3
```

---

## Iterators & Ranges

---

### Q64. What are iterator adapters?

**Answer:**  
Iterator adapters transform existing iterators into new iterator types with different behavior.

```cpp
#include <iterator>

// back_inserter: push_back to container
std::vector<int> src = {1, 2, 3, 4, 5};
std::vector<int> dst;
std::copy(src.begin(), src.end(), std::back_inserter(dst));
// dst = {1, 2, 3, 4, 5}

// front_inserter: push_front to container (needs list/deque)
std::list<int> lst;
std::copy(src.begin(), src.end(), std::front_inserter(lst));
// lst = {5, 4, 3, 2, 1}

// inserter: insert at position
std::set<int> s;
std::copy(src.begin(), src.end(), std::inserter(s, s.begin()));
// s = {1, 2, 3, 4, 5}

// reverse_iterator
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.rbegin(); it != v.rend(); ++it)
    std::cout << *it << " "; // 5 4 3 2 1
```

---

### Q65. What is `std::distance` and `std::advance`?

**Answer:**  
- `distance(first, last)` — counts elements between iterators. O(1) for random access, O(n) for others.
- `advance(it, n)` — advances iterator by n. O(1) for random access, O(n) for others.

```cpp
std::list<int>   lst = {10, 20, 30, 40, 50};
std::vector<int> vec = {10, 20, 30, 40, 50};

// distance
auto lit = lst.begin();
std::cout << std::distance(lst.begin(), lst.end()); // 5 (O(n))
std::cout << std::distance(vec.begin(), vec.end()); // 5 (O(1))

// advance
std::advance(lit, 2);    // move forward 2
std::cout << *lit;       // 30

std::advance(lit, -1);   // move backward 1 (bidirectional)
std::cout << *lit;       // 20

// std::next and std::prev (non-mutating)
auto it2 = std::next(lst.begin(), 3); // points to 40
auto it3 = std::prev(lst.end(), 2);   // points to 40
std::cout << *it2 << " " << *it3;     // 40 40
```

---

### Q66. What are stream iterators?

**Answer:**  
Stream iterators treat I/O streams as sequences, enabling algorithms to read from/write to streams.

```cpp
#include <iterator>
#include <sstream>

// istream_iterator: read from stream
std::istringstream iss("1 2 3 4 5");
std::vector<int> v(std::istream_iterator<int>(iss),
                   std::istream_iterator<int>());
// v = {1, 2, 3, 4, 5}

// ostream_iterator: write to stream
std::copy(v.begin(), v.end(),
          std::ostream_iterator<int>(std::cout, ", "));
// Output: 1, 2, 3, 4, 5,

// Combine: read, sort, print
std::istringstream data("5 3 1 4 2");
std::vector<int> nums(std::istream_iterator<int>(data),
                      std::istream_iterator<int>());
std::sort(nums.begin(), nums.end());
std::copy(nums.begin(), nums.end(),
          std::ostream_iterator<int>(std::cout, " ")); // 1 2 3 4 5
```

---

### Q67. What are C++20 ranges?

**Answer:**  
Ranges (C++20) provide a composable, lazy pipeline of views and algorithms that operate on any range without needing explicit `begin()`/`end()` pairs.

```cpp
#include <ranges>
#include <vector>

std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Compose views lazily
auto result = v
    | std::views::filter([](int x) { return x % 2 == 0; }) // even
    | std::views::transform([](int x) { return x * x; })   // square
    | std::views::take(3);                                  // first 3

for (int x : result) std::cout << x << " "; // 4 16 36

// ranges::sort, ranges::find, etc.
std::ranges::sort(v);
auto it = std::ranges::find(v, 5);
std::cout << *it; // 5
```

---

### Q68. What is `std::span` (C++20)?

**Answer:**  
`std::span<T>` is a non-owning view over a contiguous sequence (array, vector, etc.). It replaces pointer+size pairs and works with any contiguous container.

```cpp
#include <span>

void process(std::span<int> data) {
    for (int& x : data) x *= 2;
}

// Works with raw arrays
int arr[] = {1, 2, 3, 4, 5};
process(arr); // modifies arr

// Works with vector
std::vector<int> v = {1, 2, 3, 4, 5};
process(v);
for (int x : v) std::cout << x << " "; // 2 4 6 8 10

// Sub-span
std::span<int> all(v);
std::span<int> middle = all.subspan(1, 3); // elements [1..3]
process(middle);
for (int x : v) std::cout << x << " "; // 2 8 12 16 10
```

---

### Q69. How do you use `std::iota` and range-based loops with containers?

**Answer:**  
`std::iota` fills a range with sequentially increasing values. Range-based `for` works with any container offering `begin()`/`end()`.

```cpp
#include <numeric>

std::vector<int> v(10);
std::iota(v.begin(), v.end(), 1); // {1,2,3,4,5,6,7,8,9,10}

// Range-based for with index (C++20 alternative)
for (auto [i, val] : std::views::enumerate(v)) // C++23
    std::cout << i << ":" << val << " ";

// Traditional index + range
int idx = 0;
for (int x : v) std::cout << idx++ << ":" << x << " ";

// Modify in range-based for (use reference)
for (int& x : v) x *= x;         // square each element
for (const int& x : v) std::cout << x << " "; // read-only
```

---

## STL Algorithms

---

### Q70. What are the most important STL algorithms?

**Answer:**

```cpp
#include <algorithm>
#include <numeric>
std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};

// Searching
auto it  = std::find(v.begin(), v.end(), 9);     // first 9
auto cnt = std::count(v.begin(), v.end(), 5);    // count of 5 = 3
bool any = std::any_of(v.begin(), v.end(), [](int x){ return x > 8; }); // true
bool all = std::all_of(v.begin(), v.end(), [](int x){ return x > 0; }); // true

// Sorting & ordering
std::sort(v.begin(), v.end());                   // ascending
auto mm  = std::minmax_element(v.begin(), v.end()); // {min, max}
std::cout << *mm.first << " " << *mm.second;     // 1 9

// Transformation
std::vector<int> out(v.size());
std::transform(v.begin(), v.end(), out.begin(),
               [](int x){ return x * 2; });

// Accumulation
int sum = std::accumulate(v.begin(), v.end(), 0);         // sum
int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<int>());

// Partitioning
std::partition(v.begin(), v.end(), [](int x){ return x % 2 == 0; });
// even elements before odd
```

---

### Q71. What is `std::transform` and how do you use it with two ranges?

**Answer:**

```cpp
std::vector<int> a = {1, 2, 3, 4, 5};
std::vector<int> b = {10, 20, 30, 40, 50};
std::vector<int> result(5);

// Unary transform: f(a[i]) → result[i]
std::transform(a.begin(), a.end(), result.begin(),
               [](int x) { return x * x; });
// result = {1, 4, 9, 16, 25}

// Binary transform: f(a[i], b[i]) → result[i]
std::transform(a.begin(), a.end(), b.begin(), result.begin(),
               [](int x, int y) { return x + y; });
// result = {11, 22, 33, 44, 55}

// In-place transform
std::transform(a.begin(), a.end(), a.begin(),
               [](int x) { return x * 3; });
// a = {3, 6, 9, 12, 15}
```

---

### Q72. What is `std::copy`, `std::copy_if`, and `std::copy_n`?

**Answer:**

```cpp
std::vector<int> src = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
std::vector<int> dst;

// copy all
std::copy(src.begin(), src.end(), std::back_inserter(dst));

// copy_if: only elements satisfying predicate
std::vector<int> evens;
std::copy_if(src.begin(), src.end(), std::back_inserter(evens),
             [](int x) { return x % 2 == 0; });
// evens = {2, 4, 6, 8, 10}

// copy_n: copy first n elements
std::vector<int> first3;
std::copy_n(src.begin(), 3, std::back_inserter(first3));
// first3 = {1, 2, 3}

// copy to output stream
std::copy(evens.begin(), evens.end(),
          std::ostream_iterator<int>(std::cout, " ")); // 2 4 6 8 10
```

---

### Q73. What is `std::reduce` and `std::transform_reduce` (C++17)?

**Answer:**  
Parallel-friendly versions of `accumulate`. `transform_reduce` combines transform and reduce in one pass.

```cpp
#include <numeric>

std::vector<int> v = {1, 2, 3, 4, 5};

// reduce: sum (like accumulate but order not guaranteed)
int sum = std::reduce(v.begin(), v.end(), 0); // 15

// Dot product of two vectors
std::vector<int> w = {2, 2, 2, 2, 2};
int dot = std::transform_reduce(
    v.begin(), v.end(),
    w.begin(),
    0,                    // initial value
    std::plus<int>(),     // reduce op
    std::multiplies<int>()// transform op
);
std::cout << dot; // 1*2+2*2+3*2+4*2+5*2 = 30

// Sum of squares
int sumSq = std::transform_reduce(v.begin(), v.end(), 0,
    std::plus<int>(), [](int x) { return x * x; });
std::cout << sumSq; // 1+4+9+16+25 = 55
```

---

### Q74. What is `std::partial_sort` vs `std::nth_element`?

**Answer:**  
- `partial_sort(first, middle, last)` — sorts the range such that `[first, middle)` contains the smallest elements in sorted order.
- `nth_element(first, nth, last)` — rearranges such that `*nth` is the element that would be there if fully sorted; elements before are ≤ and after are ≥. O(n) average.

```cpp
std::vector<int> v = {9, 3, 7, 1, 5, 8, 2, 4, 6};

// partial_sort: get 3 smallest, sorted
std::partial_sort(v.begin(), v.begin() + 3, v.end());
std::cout << v[0] << v[1] << v[2]; // 1 2 3

// nth_element: find median in O(n)
std::vector<int> data = {9, 3, 7, 1, 5, 8, 2, 4, 6};
int n = data.size();
std::nth_element(data.begin(), data.begin() + n/2, data.end());
std::cout << data[n/2]; // 5 — median
```

---

### Q75. What is `std::stable_sort` and `std::stable_partition`?

**Answer:**  
Stable algorithms preserve the relative order of equal elements — useful when elements carry identity beyond their sort key.

```cpp
struct Student { std::string name; int grade; };
std::vector<Student> students = {
    {"Alice", 90}, {"Bob", 85}, {"Carol", 90},
    {"Dave", 85},  {"Eve", 90}
};

// stable_sort: equal grades preserve name order
std::stable_sort(students.begin(), students.end(),
    [](const Student& a, const Student& b) {
        return a.grade > b.grade; // descending grade
    });

for (auto& s : students)
    std::cout << s.name << "(" << s.grade << ") ";
// Alice(90) Carol(90) Eve(90) Bob(85) Dave(85)
// ↑ Alice before Carol before Eve — original order preserved

// stable_partition
std::stable_partition(students.begin(), students.end(),
    [](const Student& s) { return s.grade >= 90; });
```

---

### Q76. What is `std::generate` and `std::fill`?

**Answer:**

```cpp
#include <random>

std::vector<int> v(8);

// fill: set all elements to a value
std::fill(v.begin(), v.end(), 42);
// v = {42, 42, 42, 42, 42, 42, 42, 42}

std::fill_n(v.begin(), 3, 99);
// v = {99, 99, 99, 42, 42, 42, 42, 42}

// generate: call a function for each element
int counter = 0;
std::generate(v.begin(), v.end(), [&counter]() { return counter++ * 10; });
// v = {0, 10, 20, 30, 40, 50, 60, 70}

// Random number generation
std::mt19937 rng(42); // seeded mersenne twister
std::uniform_int_distribution<int> dist(1, 100);
std::generate(v.begin(), v.end(), [&]() { return dist(rng); });
for (int x : v) std::cout << x << " "; // random values in [1, 100]
```

---

## Container Selection & Advanced Topics

---

### Q77. How do you choose the right container for a given problem?

**Answer:**

```
Need key-value pairs?
├── Yes → Need sorted order or range queries?
│         ├── Yes → map / multimap
│         └── No  → unordered_map / unordered_multimap (faster)
└── No  → Need only keys (set-like)?
          ├── Yes → Need sorted order?
          │         ├── Yes → set / multiset
          │         └── No  → unordered_set / unordered_multiset
          └── No  → Sequence container needed:
                    Need random access?
                    ├── Yes → Fixed size? → array
                    │         Dynamic?     → vector (default)
                    └── No  → Need O(1) front insert/delete?
                              ├── Yes + back too → deque
                              ├── Middle insert (with iterator)? → list
                              └── Just back        → vector
```

---

### Q78. What is the time complexity of common operations across containers?

**Answer:**

| Operation | vector | list | deque | map/set | unordered_map/set |
|---|---|---|---|---|---|
| Access by index | O(1) | O(n) | O(1) | — | — |
| Insert front | O(n) | O(1) | O(1) | — | — |
| Insert back | O(1)* | O(1) | O(1) | — | — |
| Insert middle | O(n) | O(1)† | O(n) | O(log n) | O(1)* |
| Search | O(n) | O(n) | O(n) | O(log n) | O(1)* |
| Delete by value | O(n) | O(n) | O(n) | O(log n) | O(1)* |
| Sort | O(n log n) | O(n log n) | O(n log n) | Always sorted | N/A |

\* amortized  
† given an iterator to position

---

### Q79. What is `std::flat_map` and `std::flat_set` (C++23)?

**Answer:**  
Sorted associative containers backed by a contiguous array instead of a tree. Better cache performance for small-medium sizes; worse insert/erase (O(n) shifts).

```cpp
// C++23
#include <flat_map>
#include <flat_set>

std::flat_map<std::string, int> fm;
fm["banana"] = 2;
fm["apple"]  = 1;
fm["cherry"] = 3;

// Sorted like map, stored contiguously like vector
for (auto& [k, v] : fm)
    std::cout << k << ": " << v << "\n"; // alphabetical order

// Faster iteration than std::map due to cache friendliness
// Slower insert than std::map due to shifting

std::flat_set<int> fs = {5, 3, 1, 4, 2};
for (int x : fs) std::cout << x << " "; // 1 2 3 4 5

// Best for: read-heavy, small-medium size, cache-sensitive code
```

---

### Q80. What are common pitfalls when using STL containers?

**Answer:**

```cpp
// PITFALL 1: operator[] inserts into map
std::map<std::string, int> m;
if (m["key"] == 0) { /* key was INSERTED with value 0! */ }
// Fix: use find() or contains()
if (m.find("key") != m.end()) { /* safe */ }

// PITFALL 2: Invalidated iterators in vector
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 3) v.push_back(99); // may invalidate it!
}
// Fix: collect, then modify; or use index-based loop

// PITFALL 3: Forgetting erase in erase-remove
std::vector<int> v2 = {1, 2, 3, 2, 4};
std::remove(v2.begin(), v2.end(), 2); // doesn't shrink!
// Fix: always pair with erase:
v2.erase(std::remove(v2.begin(), v2.end(), 2), v2.end());

// PITFALL 4: size() returns unsigned — underflow bug
std::vector<int> v3 = {1, 2, 3};
for (int i = 0; i <= (int)v3.size() - 1; ++i) {} // cast to int
// If v3 is empty: v3.size()-1 wraps to huge number (UB territory)

// PITFALL 5: Copying large containers accidentally
std::map<int, std::vector<int>> bigMap;
auto copy = bigMap;             // O(n) deep copy!
const auto& ref = bigMap;       // O(1) reference — use this

// PITFALL 6: unordered_map worst-case O(n) from hash collision
// Use a good hash or reserve() to reduce collisions
std::unordered_map<int, int> um;
um.reserve(10000);              // pre-size to avoid rehashing
```

---

## Quick Reference Summary

| Container | Header | Ordered? | Unique? | Access | Insert/Delete |
|---|---|---|---|---|---|
| `vector<T>` | `<vector>` | Positional | No | O(1) random | O(1) back / O(n) middle |
| `array<T,N>` | `<array>` | Positional | No | O(1) random | Fixed size |
| `list<T>` | `<list>` | Positional | No | O(n) | O(1) with iterator |
| `forward_list<T>` | `<forward_list>` | Positional | No | O(n) forward | O(1) with iterator |
| `deque<T>` | `<deque>` | Positional | No | O(1) random | O(1) front+back |
| `map<K,V>` | `<map>` | By key | Yes | O(log n) | O(log n) |
| `multimap<K,V>` | `<map>` | By key | No | O(log n) | O(log n) |
| `set<T>` | `<set>` | By value | Yes | O(log n) | O(log n) |
| `multiset<T>` | `<set>` | By value | No | O(log n) | O(log n) |
| `unordered_map<K,V>` | `<unordered_map>` | No | Yes | O(1) avg | O(1) avg |
| `unordered_set<T>` | `<unordered_set>` | No | Yes | O(1) avg | O(1) avg |
| `stack<T>` | `<stack>` | LIFO | No | top only | O(1) |
| `queue<T>` | `<queue>` | FIFO | No | front/back | O(1) |
| `priority_queue<T>` | `<queue>` | By priority | No | max only | O(log n) |

---

*Master STL containers and you command the full power of modern C++!*
