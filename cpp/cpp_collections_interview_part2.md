# C++ Interview Questions: Collections & STL — Part 2

> **80 additional questions with answers and code examples** — deep dives into algorithms, custom containers, concurrency, design patterns, and competitive-programming-level STL mastery.

---

## Table of Contents

1. [Advanced vector Techniques (Q1–Q8)](#advanced-vector-techniques)
2. [Advanced map / set Patterns (Q9–Q17)](#advanced-map--set-patterns)
3. [Advanced unordered Containers (Q18–Q24)](#advanced-unordered-containers)
4. [Algorithm Deep Dives (Q25–Q35)](#algorithm-deep-dives)
5. [Numeric Algorithms (Q36–Q41)](#numeric-algorithms)
6. [Custom Comparators & Ordering (Q42–Q47)](#custom-comparators--ordering)
7. [Memory, Allocators & Performance (Q48–Q54)](#memory-allocators--performance)
8. [Concurrent Collections & Thread Safety (Q55–Q60)](#concurrent-collections--thread-safety)
9. [Common Interview Coding Patterns (Q61–Q72)](#common-interview-coding-patterns)
10. [C++17 / C++20 / C++23 STL Features (Q73–Q80)](#c17--c20--c23-stl-features)

---

## Advanced vector Techniques

---

### Q1. How do you efficiently remove duplicates from a `vector` while preserving order?

**Answer:**  
Use an `unordered_set` as a "seen" tracker while building the result in one pass — O(n) time and space.

```cpp
template<typename T>
std::vector<T> removeDuplicatesOrdered(std::vector<T> v) {
    std::unordered_set<T> seen;
    std::vector<T> result;
    result.reserve(v.size());

    for (auto& x : v)
        if (seen.insert(x).second) // insert returns {iterator, bool}
            result.push_back(x);

    return result;
}

auto v = removeDuplicatesOrdered(std::vector<int>{5, 3, 1, 3, 5, 2, 1});
for (int x : v) std::cout << x << " "; // 5 3 1 2

// If order doesn't matter: sort + unique + erase — O(n log n), O(1) space
std::vector<int> w = {5, 3, 1, 3, 5, 2, 1};
std::sort(w.begin(), w.end());
w.erase(std::unique(w.begin(), w.end()), w.end());
for (int x : w) std::cout << x << " "; // 1 2 3 5
```

---

### Q2. How do you flatten a `vector<vector<T>>` into a single `vector<T>`?

**Answer:**

```cpp
std::vector<std::vector<int>> nested = {{1,2,3},{4,5},{6,7,8,9}};

// Method 1: reserve + insert
std::vector<int> flat;
flat.reserve(std::accumulate(nested.begin(), nested.end(), 0u,
    [](size_t sum, const auto& v) { return sum + v.size(); }));

for (auto& inner : nested)
    flat.insert(flat.end(), inner.begin(), inner.end());
// flat = {1,2,3,4,5,6,7,8,9}

// Method 2: ranges (C++20)
#include <ranges>
auto r = nested | std::views::join;
std::vector<int> flat2(r.begin(), r.end());

// Method 3: transform_reduce for total size
size_t total = std::transform_reduce(nested.begin(), nested.end(), 0u,
    std::plus<>{}, [](const auto& v){ return v.size(); });
```

---

### Q3. How do you rotate a `vector` efficiently?

**Answer:**  
`std::rotate` rotates elements in-place in O(n) with no extra allocation.

```cpp
std::vector<int> v = {1, 2, 3, 4, 5, 6, 7};

// Rotate left by 3 (bring v[3] to front)
std::rotate(v.begin(), v.begin() + 3, v.end());
for (int x : v) std::cout << x << " "; // 4 5 6 7 1 2 3

// Rotate right by 2 (bring last 2 to front)
std::rotate(v.begin(), v.end() - 2, v.end());
for (int x : v) std::cout << x << " "; // 2 3 4 5 6 7 1

// Rotate a specific element to front
std::vector<int> w = {3, 1, 4, 1, 5, 9};
auto target = std::find(w.begin(), w.end(), 5);
if (target != w.end())
    std::rotate(w.begin(), target, w.end());
for (int x : w) std::cout << x << " "; // 5 9 3 1 4 1
```

---

### Q4. How do you merge two sorted `vector`s efficiently?

**Answer:**

```cpp
std::vector<int> a = {1, 3, 5, 7, 9};
std::vector<int> b = {2, 4, 6, 8, 10};

// std::merge: O(n+m)
std::vector<int> merged;
merged.resize(a.size() + b.size());
std::merge(a.begin(), a.end(), b.begin(), b.end(), merged.begin());
for (int x : merged) std::cout << x << " "; // 1 2 3 4 5 6 7 8 9 10

// In-place merge (modifies the range): append then inplace_merge
std::vector<int> combined = a;
combined.insert(combined.end(), b.begin(), b.end());
std::inplace_merge(combined.begin(),
                   combined.begin() + a.size(),
                   combined.end());
for (int x : combined) std::cout << x << " "; // 1 2 3...10

// Check if sorted range includes another (set operations)
std::vector<int> intersection;
std::set_intersection(a.begin(), a.end(), b.begin(), b.end(),
                      std::back_inserter(intersection));
std::cout << intersection.empty(); // true (no common elements)
```

---

### Q5. How do you use `vector<bool>` and what are its pitfalls?

**Answer:**  
`vector<bool>` is a special case — it packs bits (1 bit per bool) to save memory. But it does **not** behave like a normal `vector`: elements are not individually addressable, and `auto& x = v[i]` gives a proxy object, not a real `bool&`.

```cpp
std::vector<bool> vb = {true, false, true, true, false};

std::cout << vb.size();  // 5
std::cout << vb[2];      // true (via proxy)

// PITFALL: no real reference to elements
auto ref = vb[0];        // proxy object, NOT bool&
// bool& realRef = vb[0]; // Error!

// PITFALL: &vb[0] doesn't give a pointer to bool
// bool* ptr = &vb[0];   // Error!

// Prefer deque<bool> or vector<uint8_t> for mutable element access
std::vector<uint8_t> flags = {1, 0, 1, 1, 0};
uint8_t& realRef = flags[0]; // works fine

// Or use bitset for fixed-size bit arrays
std::bitset<5> bits("10110");
std::cout << bits[2]; // 1
```

---

### Q6. How do you implement a circular buffer using `vector` or `deque`?

**Answer:**

```cpp
template<typename T>
class CircularBuffer {
    std::vector<T> buf;
    size_t head = 0, tail = 0, count = 0;
    size_t cap;
public:
    explicit CircularBuffer(size_t capacity)
        : buf(capacity), cap(capacity) {}

    void push(const T& val) {
        buf[tail] = val;
        tail = (tail + 1) % cap;
        if (count < cap) ++count;
        else head = (head + 1) % cap; // overwrite oldest
    }

    T pop() {
        if (empty()) throw std::underflow_error("Empty buffer");
        T val = buf[head];
        head = (head + 1) % cap;
        --count;
        return val;
    }

    bool empty() const { return count == 0; }
    bool full()  const { return count == cap; }
    size_t size()const { return count; }
};

CircularBuffer<int> cb(4);
cb.push(1); cb.push(2); cb.push(3); cb.push(4);
cb.push(5); // overwrites 1 (full)
std::cout << cb.pop(); // 2 (oldest remaining)
std::cout << cb.pop(); // 3
```

---

### Q7. How do you efficiently find the top-K elements from a `vector`?

**Answer:**

```cpp
std::vector<int> v = {9, 3, 7, 1, 5, 8, 2, 4, 6};
int k = 3;

// Method 1: partial_sort — O(n log k), sorted top-k
std::vector<int> copy1 = v;
std::partial_sort(copy1.begin(), copy1.begin() + k, copy1.end(),
                  std::greater<int>());
for (int i = 0; i < k; ++i) std::cout << copy1[i] << " "; // 9 8 7

// Method 2: nth_element — O(n), top-k unsorted
std::vector<int> copy2 = v;
std::nth_element(copy2.begin(), copy2.begin() + k, copy2.end(),
                 std::greater<int>());
std::vector<int> topK(copy2.begin(), copy2.begin() + k);
// topK = {9, 8, 7} in some order

// Method 3: min-heap of size k — O(n log k), streaming-friendly
std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
for (int x : v) {
    minHeap.push(x);
    if ((int)minHeap.size() > k) minHeap.pop();
}
// heap contains top-k elements
while (!minHeap.empty()) {
    std::cout << minHeap.top() << " "; // 7 8 9
    minHeap.pop();
}
```

---

### Q8. How do you check if one `vector` is a permutation of another?

**Answer:**

```cpp
std::vector<int> a = {1, 2, 3, 4, 5};
std::vector<int> b = {5, 3, 1, 4, 2};
std::vector<int> c = {1, 2, 3, 4, 6};

// Method 1: std::is_permutation — O(n²) worst case
std::cout << std::is_permutation(a.begin(), a.end(), b.begin()); // true
std::cout << std::is_permutation(a.begin(), a.end(), c.begin()); // false

// Method 2: sort both copies — O(n log n)
auto sortedA = a, sortedB = b;
std::sort(sortedA.begin(), sortedA.end());
std::sort(sortedB.begin(), sortedB.end());
std::cout << (sortedA == sortedB); // true

// Method 3: frequency map — O(n) for integer types
auto isPermutation = [](const std::vector<int>& x,
                        const std::vector<int>& y) {
    if (x.size() != y.size()) return false;
    std::unordered_map<int, int> freq;
    for (int v : x) ++freq[v];
    for (int v : y) if (--freq[v] < 0) return false;
    return true;
};
std::cout << isPermutation(a, b); // true
```

---

## Advanced map / set Patterns

---

### Q9. How do you invert a `map` (swap keys and values)?

**Answer:**

```cpp
std::map<std::string, int> original = {
    {"Alice", 1}, {"Bob", 2}, {"Carol", 3}
};

// Invert: value becomes key
std::map<int, std::string> inverted;
for (auto& [k, v] : original)
    inverted[v] = k;

for (auto& [k, v] : inverted)
    std::cout << k << " -> " << v << "\n";
// 1 -> Alice, 2 -> Bob, 3 -> Carol

// With duplicate values: use multimap
std::map<std::string, int> dupes = {{"A", 1}, {"B", 1}, {"C", 2}};
std::multimap<int, std::string> invertedMulti;
for (auto& [k, v] : dupes)
    invertedMulti.insert({v, k});

auto [lo, hi] = invertedMulti.equal_range(1);
for (auto it = lo; it != hi; ++it)
    std::cout << it->second << " "; // A B
```

---

### Q10. How do you implement a frequency counter with `map`?

**Answer:**

```cpp
// Word frequency counter
std::string text = "the cat sat on the mat the cat";
std::istringstream iss(text);
std::map<std::string, int> freq;

std::string word;
while (iss >> word) freq[word]++;

// Sort by frequency (descending) — map is sorted by key, need vector
std::vector<std::pair<std::string, int>> sorted(freq.begin(), freq.end());
std::sort(sorted.begin(), sorted.end(),
    [](const auto& a, const auto& b) {
        return b.second < a.second; // descending frequency
    });

for (auto& [w, c] : sorted)
    std::cout << w << ": " << c << "\n";
// the: 3, cat: 2, mat: 1, on: 1, sat: 1

// Find most frequent
auto maxIt = std::max_element(freq.begin(), freq.end(),
    [](const auto& a, const auto& b) {
        return a.second < b.second;
    });
std::cout << "Most frequent: " << maxIt->first; // "the"
```

---

### Q11. How do you implement a bidirectional map (bimap)?

**Answer:**

```cpp
template<typename A, typename B>
class BiMap {
    std::map<A, B> forward;
    std::map<B, A> backward;
public:
    void insert(const A& a, const B& b) {
        forward[a] = b;
        backward[b] = a;
    }

    B getByA(const A& a) const { return forward.at(a); }
    A getByB(const B& b) const { return backward.at(b); }

    bool containsA(const A& a) const { return forward.count(a) > 0; }
    bool containsB(const B& b) const { return backward.count(b) > 0; }

    void eraseByA(const A& a) {
        auto it = forward.find(a);
        if (it != forward.end()) {
            backward.erase(it->second);
            forward.erase(it);
        }
    }
};

BiMap<std::string, int> bm;
bm.insert("one", 1);
bm.insert("two", 2);
bm.insert("three", 3);

std::cout << bm.getByA("two");    // 2
std::cout << bm.getByB(3);        // "three"
```

---

### Q12. How do you use `map` to implement a simple in-memory database index?

**Answer:**

```cpp
struct Record {
    int    id;
    std::string name;
    int    age;
    std::string dept;
};

class Database {
    std::vector<Record> records;
    std::map<int, size_t>         idIndex;      // id → row
    std::multimap<std::string, size_t> nameIndex; // name → row(s)
    std::multimap<std::string, size_t> deptIndex; // dept → row(s)

public:
    void insert(Record r) {
        size_t row = records.size();
        records.push_back(r);
        idIndex[r.id] = row;
        nameIndex.insert({r.name, row});
        deptIndex.insert({r.dept, row});
    }

    const Record* findById(int id) const {
        auto it = idIndex.find(id);
        return it == idIndex.end() ? nullptr : &records[it->second];
    }

    std::vector<const Record*> findByDept(const std::string& dept) const {
        std::vector<const Record*> result;
        auto [lo, hi] = deptIndex.equal_range(dept);
        for (auto it = lo; it != hi; ++it)
            result.push_back(&records[it->second]);
        return result;
    }
};

Database db;
db.insert({1, "Alice", 30, "Engineering"});
db.insert({2, "Bob",   25, "Marketing"});
db.insert({3, "Carol", 28, "Engineering"});

auto* rec = db.findById(2);
std::cout << rec->name; // Bob

auto eng = db.findByDept("Engineering");
for (auto* r : eng) std::cout << r->name << " "; // Alice Carol
```

---

### Q13. How do you merge two `map`s?

**Answer:**

```cpp
std::map<std::string, int> m1 = {{"a", 1}, {"b", 2}, {"c", 3}};
std::map<std::string, int> m2 = {{"b", 20}, {"c", 30}, {"d", 40}};

// C++17: merge() — moves nodes from m2 into m1 without copying
// Only inserts if key is not already in m1
m1.merge(m2); // m2 loses entries that were successfully moved
std::cout << m1["b"]; // 2 (m1's value wins)
std::cout << m1["d"]; // 40 (moved from m2)
std::cout << m2.size(); // 2 (b and c remain — already in m1)

// Merge with override (m2 wins on conflict):
std::map<std::string, int> m3 = {{"a",1},{"b",2}};
std::map<std::string, int> m4 = {{"b",20},{"c",30}};
for (auto& [k, v] : m4)
    m3.insert_or_assign(k, v); // m4 values override m3
std::cout << m3["b"]; // 20
```

---

### Q14. How do you implement a sliding window maximum using `deque` + `map`?

**Answer:**

```cpp
// Sliding window maximum using monotonic deque
std::vector<int> slidingWindowMax(const std::vector<int>& nums, int k) {
    std::deque<int> dq; // stores indices
    std::vector<int> result;

    for (int i = 0; i < (int)nums.size(); ++i) {
        // Remove indices outside window
        while (!dq.empty() && dq.front() < i - k + 1)
            dq.pop_front();

        // Maintain decreasing order
        while (!dq.empty() && nums[dq.back()] < nums[i])
            dq.pop_back();

        dq.push_back(i);

        if (i >= k - 1)
            result.push_back(nums[dq.front()]);
    }
    return result;
}

auto res = slidingWindowMax({1, 3, -1, -3, 5, 3, 6, 7}, 3);
for (int x : res) std::cout << x << " "; // 3 3 5 5 6 7

// Alternatively with ordered_map (for frequency tracking):
auto slidingWindowMaxMap = [](const std::vector<int>& v, int k) {
    std::map<int, int, std::greater<int>> window; // descending
    std::vector<int> result;
    for (int i = 0; i < (int)v.size(); ++i) {
        ++window[v[i]];
        if (i >= k) {
            if (--window[v[i-k]] == 0)
                window.erase(v[i-k]);
        }
        if (i >= k-1)
            result.push_back(window.begin()->first);
    }
    return result;
};
```

---

### Q15. How do you use `std::set` to implement an interval set?

**Answer:**

```cpp
// Merge overlapping intervals stored in a set
using Interval = std::pair<int, int>;

class IntervalSet {
    std::set<Interval> intervals;
public:
    void insert(int lo, int hi) {
        Interval newI{lo, hi};
        // Find first interval that could overlap: start <= hi
        auto it = intervals.lower_bound({lo, INT_MIN});
        if (it != intervals.begin()) {
            --it;
            if (it->second < lo) ++it; // no overlap with previous
        }
        // Merge all overlapping intervals
        while (it != intervals.end() && it->first <= hi) {
            newI.first  = std::min(newI.first,  it->first);
            newI.second = std::max(newI.second, it->second);
            it = intervals.erase(it);
        }
        intervals.insert(newI);
    }

    void print() const {
        for (auto& [lo, hi] : intervals)
            std::cout << "[" << lo << "," << hi << "] ";
        std::cout << "\n";
    }
};

IntervalSet is;
is.insert(1, 3);  // [1,3]
is.insert(6, 9);  // [1,3] [6,9]
is.insert(2, 5);  // [1,5] [6,9]   ← merged with [1,3]
is.insert(4, 7);  // [1,9]          ← merged all
is.print();        // [1,9]
```

---

### Q16. How do you count elements in a range using `map` (order statistics)?

**Answer:**

```cpp
std::map<int, int> freq; // value → count

std::vector<int> data = {5, 3, 7, 3, 1, 5, 3, 9, 5};
for (int x : data) ++freq[x];

// Count elements less than 5
int lessThan5 = 0;
for (auto it = freq.begin(); it != freq.lower_bound(5); ++it)
    lessThan5 += it->second;
std::cout << lessThan5; // 4 (three 3s + one 1)

// Count elements in range [3, 7]
int inRange = 0;
for (auto it = freq.lower_bound(3); it != freq.upper_bound(7); ++it)
    inRange += it->second;
std::cout << inRange; // 7 (three 3s + three 5s + one 7)

// Rank of element: how many elements are strictly less
auto rank = [&](int x) {
    int r = 0;
    for (auto it = freq.begin(); it != freq.lower_bound(x); ++it)
        r += it->second;
    return r;
};
std::cout << rank(5); // 4 (rank of 5)
```

---

### Q17. How do you implement an LRU Cache using `list` + `unordered_map`?

**Answer:**

```cpp
class LRUCache {
    int capacity;
    std::list<std::pair<int,int>> cache;        // {key, value}, MRU at front
    std::unordered_map<int,
        std::list<std::pair<int,int>>::iterator> map;

public:
    LRUCache(int cap) : capacity(cap) {}

    int get(int key) {
        auto it = map.find(key);
        if (it == map.end()) return -1;
        cache.splice(cache.begin(), cache, it->second); // O(1) move to front
        return it->second->second;
    }

    void put(int key, int value) {
        auto it = map.find(key);
        if (it != map.end()) {
            it->second->second = value;
            cache.splice(cache.begin(), cache, it->second);
            return;
        }
        if ((int)cache.size() == capacity) {
            map.erase(cache.back().first);
            cache.pop_back(); // evict LRU
        }
        cache.push_front({key, value});
        map[key] = cache.begin();
    }
};

LRUCache lru(3);
lru.put(1, 10); lru.put(2, 20); lru.put(3, 30);
std::cout << lru.get(1);  // 10 — 1 becomes MRU
lru.put(4, 40);           // evicts 2 (LRU)
std::cout << lru.get(2);  // -1 (evicted)
std::cout << lru.get(3);  // 30
```

---

## Advanced unordered Containers

---

### Q18. How do you implement a hash for a `std::pair` or `std::tuple`?

**Answer:**  
The standard library doesn't provide hashes for `pair` or `tuple`. You must provide them.

```cpp
// Hash for std::pair
struct PairHash {
    template<typename A, typename B>
    std::size_t operator()(const std::pair<A, B>& p) const {
        auto h1 = std::hash<A>{}(p.first);
        auto h2 = std::hash<B>{}(p.second);
        // Boost-style hash combine
        return h1 ^ (h2 * 2654435761ULL + 0x9e3779b9 + (h1 << 6) + (h1 >> 2));
    }
};

std::unordered_map<std::pair<int,int>, std::string, PairHash> grid;
grid[{0, 0}] = "origin";
grid[{1, 0}] = "right";
std::cout << grid[{0, 0}]; // "origin"

// Hash for tuple
struct TupleHash {
    template<typename... Ts>
    std::size_t operator()(const std::tuple<Ts...>& t) const {
        std::size_t seed = 0;
        std::apply([&seed](const auto&... args) {
            (( seed ^= std::hash<std::decay_t<decltype(args)>>{}(args)
                        + 0x9e3779b9 + (seed << 6) + (seed >> 2) ), ...);
        }, t);
        return seed;
    }
};

std::unordered_set<std::tuple<int,int,int>, TupleHash> points3d;
points3d.insert({1, 2, 3});
points3d.insert({4, 5, 6});
```

---

### Q19. What is the difference between `count()` and `find()` for `unordered_map`?

**Answer:**  
For unique-key maps, `count()` returns 0 or 1 and `find()` returns an iterator. Prefer `find()` when you also need access to the value — avoids a second lookup.

```cpp
std::unordered_map<std::string, int> m = {{"a",1},{"b",2},{"c",3}};

// count: just check existence
if (m.count("a")) std::cout << "exists\n"; // one lookup

// find: check AND access value
if (auto it = m.find("b"); it != m.end())
    std::cout << it->second; // 2 — one lookup, also gets value

// BAD: two lookups
if (m.count("c"))          // lookup 1
    std::cout << m["c"];   // lookup 2 (and inserts if missing!)

// GOOD: one lookup
if (auto it = m.find("c"); it != m.end())
    std::cout << it->second; // one lookup
```

---

### Q20. How do you prevent worst-case O(n) behavior in `unordered_map`?

**Answer:**  
Worst case occurs when all keys hash to the same bucket (deliberate or accidental). Mitigations:

```cpp
// 1. Use a better hash (e.g., randomized)
struct SafeHash {
    static uint64_t splitmix64(uint64_t x) {
        x += 0x9e3779b97f4a7c15ULL;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
    std::size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM =
            std::chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};

std::unordered_map<int, int, SafeHash> safe;

// 2. Reserve and control load factor
std::unordered_map<int,int> m;
m.reserve(10000);            // pre-allocate
m.max_load_factor(0.25);     // keep sparse (fewer collisions, more memory)

// 3. Use sorted map for guaranteed O(log n) if adversarial input possible
std::map<int,int> guaranteed; // always O(log n)
```

---

### Q21. How do you cache function results using `unordered_map` (memoization)?

**Answer:**

```cpp
// Fibonacci with memoization
std::unordered_map<int, long long> memo;

long long fib(int n) {
    if (n <= 1) return n;
    auto it = memo.find(n);
    if (it != memo.end()) return it->second;
    return memo[n] = fib(n-1) + fib(n-2);
}

std::cout << fib(50); // 12586269025 (instant)

// Generic memoize wrapper
template<typename Func>
auto memoize(Func f) {
    using Arg    = int; // simplified for int→int
    using Result = int;
    std::unordered_map<Arg, Result> cache;
    return [f, cache](Arg x) mutable -> Result {
        auto it = cache.find(x);
        if (it != cache.end()) return it->second;
        return cache[x] = f(x);
    };
}

auto squareMemo = memoize([](int x) { return x * x; });
std::cout << squareMemo(5);  // 25 (computed)
std::cout << squareMemo(5);  // 25 (cached)
```

---

### Q22. How do you use `unordered_map` for anagram grouping?

**Answer:**

```cpp
std::vector<std::string> words = {
    "eat","tea","tan","ate","nat","bat"
};

// Group anagrams: sorted word → group
std::unordered_map<std::string, std::vector<std::string>> groups;
for (const auto& w : words) {
    std::string key = w;
    std::sort(key.begin(), key.end()); // canonical form
    groups[key].push_back(w);
}

for (auto& [key, group] : groups) {
    for (const auto& w : group) std::cout << w << " ";
    std::cout << "\n";
}
// eat tea ate
// tan nat
// bat

// Alternative: use character frequency as key
auto charFreqKey = [](const std::string& s) {
    std::array<int, 26> freq{};
    for (char c : s) ++freq[c - 'a'];
    std::string key;
    for (int i = 0; i < 26; ++i)
        key += std::to_string(freq[i]) + ",";
    return key;
};
```

---

### Q23. How do you implement a thread-safe lookup table with `unordered_map`?

**Answer:**

```cpp
#include <shared_mutex>

template<typename K, typename V>
class ThreadSafeMap {
    mutable std::shared_mutex mutex;
    std::unordered_map<K, V> data;

public:
    void insert(const K& key, const V& value) {
        std::unique_lock lock(mutex);  // exclusive write
        data[key] = value;
    }

    std::optional<V> get(const K& key) const {
        std::shared_lock lock(mutex);  // shared read (multiple readers OK)
        auto it = data.find(key);
        if (it == data.end()) return std::nullopt;
        return it->second;
    }

    bool erase(const K& key) {
        std::unique_lock lock(mutex);
        return data.erase(key) > 0;
    }

    size_t size() const {
        std::shared_lock lock(mutex);
        return data.size();
    }
};

ThreadSafeMap<std::string, int> tsm;
tsm.insert("x", 42);
auto val = tsm.get("x");
if (val) std::cout << *val; // 42
```

---

### Q24. What is a hash map with open addressing vs separate chaining?

**Answer:**  
`std::unordered_map` uses **separate chaining** (each bucket is a linked list). Open addressing stores all elements in the array itself, using probing on collision.

```cpp
// Simplified open addressing (linear probing) hash map
template<typename K, typename V>
class OpenAddressMap {
    struct Slot { K key; V val; bool occupied = false; };
    std::vector<Slot> table;
    size_t count = 0;
    size_t cap;

    size_t probe(const K& key) const {
        size_t h = std::hash<K>{}(key) % cap;
        while (table[h].occupied && table[h].key != key)
            h = (h + 1) % cap; // linear probe
        return h;
    }
public:
    OpenAddressMap(size_t capacity = 16) : table(capacity), cap(capacity) {}

    void insert(const K& k, const V& v) {
        size_t h = probe(k);
        if (!table[h].occupied) ++count;
        table[h] = {k, v, true};
    }

    V* find(const K& k) {
        size_t h = probe(k);
        if (!table[h].occupied) return nullptr;
        return &table[h].val;
    }
};

OpenAddressMap<int, std::string> oam;
oam.insert(1, "one");
oam.insert(2, "two");
auto* v = oam.find(1);
if (v) std::cout << *v; // "one"
```

---

## Algorithm Deep Dives

---

### Q25. How does `std::sort` work internally?

**Answer:**  
Modern STL implementations use **Introsort** — a hybrid of Quicksort, Heapsort, and Insertion sort:
- Quicksort for average performance (O(n log n))
- Heapsort as fallback when recursion depth exceeds O(log n) — guarantees O(n log n) worst case
- Insertion sort for small sub-ranges (< ~16 elements) — cache-friendly

```cpp
std::vector<int> v = {9,3,7,1,5,8,2,4,6};

// Regular sort: O(n log n), not stable
std::sort(v.begin(), v.end()); // {1,2,3,4,5,6,7,8,9}

// Stable sort: O(n log n), preserves equal element order
std::stable_sort(v.begin(), v.end());

// Sort with custom comparator
struct Student { std::string name; int gpa; };
std::vector<Student> s = {{"Alice",38},{"Bob",35},{"Carol",38}};
std::stable_sort(s.begin(), s.end(),
    [](const Student& a, const Student& b){ return a.gpa > b.gpa; });
// Alice, Carol (both gpa=38, original order), then Bob

// Verify sort correctness
std::cout << std::is_sorted(v.begin(), v.end()); // true
```

---

### Q26. What is `std::lower_bound` vs `std::upper_bound` vs `std::equal_range`?

**Answer:**  
All require a sorted range and use binary search — O(log n).

```cpp
std::vector<int> v = {1, 2, 2, 2, 3, 4, 5};
//                    0  1  2  3  4  5  6

// lower_bound: first position where value >= target
auto lb = std::lower_bound(v.begin(), v.end(), 2);
std::cout << std::distance(v.begin(), lb); // 1 (index of first 2)

// upper_bound: first position where value > target
auto ub = std::upper_bound(v.begin(), v.end(), 2);
std::cout << std::distance(v.begin(), ub); // 4 (one past last 2)

// count of value 2:
std::cout << std::distance(lb, ub); // 3

// equal_range: both at once
auto [lo, hi] = std::equal_range(v.begin(), v.end(), 2);
std::cout << std::distance(lo, hi); // 3

// Insert while maintaining sorted order
auto pos = std::lower_bound(v.begin(), v.end(), 3);
v.insert(pos, 3); // {1,2,2,2,3,3,4,5}
```

---

### Q27. What is `std::partition` and `std::stable_partition`?

**Answer:**  
`partition` rearranges elements so those satisfying a predicate come first — O(n), not stable. `stable_partition` preserves relative order — O(n log n).

```cpp
std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Partition: evens first, odds after (order not preserved)
auto pivot = std::partition(v.begin(), v.end(),
                            [](int x){ return x % 2 == 0; });
// v = {10,2,8,4,6, 5,7,3,9,1} (some order — evens | odds)
std::cout << std::distance(v.begin(), pivot); // 5 (5 even numbers)

// stable_partition: preserves relative order
std::vector<int> v2 = {1,2,3,4,5,6,7,8,9,10};
std::stable_partition(v2.begin(), v2.end(),
                      [](int x){ return x % 2 == 0; });
// v2 = {2,4,6,8,10,1,3,5,7,9} (order preserved within each group)

// is_partitioned check
std::cout << std::is_partitioned(v2.begin(), v2.end(),
    [](int x){ return x % 2 == 0; }); // true

// partition_point: binary search for pivot (requires already partitioned)
auto pt = std::partition_point(v2.begin(), v2.end(),
    [](int x){ return x % 2 == 0; });
std::cout << *pt; // 1 (first odd element)
```

---

### Q28. What are set operations on sorted ranges?

**Answer:**

```cpp
std::vector<int> a = {1, 2, 3, 4, 5};
std::vector<int> b = {3, 4, 5, 6, 7};
std::vector<int> result;

// Intersection: elements in both
std::set_intersection(a.begin(), a.end(), b.begin(), b.end(),
                      std::back_inserter(result));
// result = {3, 4, 5}
result.clear();

// Union: elements in either
std::set_union(a.begin(), a.end(), b.begin(), b.end(),
               std::back_inserter(result));
// result = {1, 2, 3, 4, 5, 6, 7}
result.clear();

// Difference: in a but not b
std::set_difference(a.begin(), a.end(), b.begin(), b.end(),
                    std::back_inserter(result));
// result = {1, 2}
result.clear();

// Symmetric difference: in one but not both
std::set_symmetric_difference(a.begin(), a.end(), b.begin(), b.end(),
                              std::back_inserter(result));
// result = {1, 2, 6, 7}

// Subset check
std::vector<int> sub = {3, 4};
std::cout << std::includes(a.begin(), a.end(), sub.begin(), sub.end()); // true
```

---

### Q29. How does `std::next_permutation` work?

**Answer:**  
Rearranges elements into the next lexicographically greater permutation. Returns `false` when wrapping around to the smallest permutation.

```cpp
std::vector<int> v = {1, 2, 3};

// Generate all permutations
do {
    for (int x : v) std::cout << x;
    std::cout << " ";
} while (std::next_permutation(v.begin(), v.end()));
// 123 132 213 231 312 321

// Count permutations (= n!)
int count = 0;
std::vector<int> perm = {1,2,3,4};
std::sort(perm.begin(), perm.end()); // must start from smallest
do { ++count; } while (std::next_permutation(perm.begin(), perm.end()));
std::cout << count; // 24 (4!)

// prev_permutation: goes the other way
std::vector<int> desc = {3, 2, 1};
std::prev_permutation(desc.begin(), desc.end());
for (int x : desc) std::cout << x; // 3 1 2
```

---

### Q30. What is `std::sample` (C++17)?

**Answer:**  
Selects n elements from a range uniformly at random without replacement.

```cpp
#include <random>

std::vector<int> population(100);
std::iota(population.begin(), population.end(), 1); // {1..100}

std::vector<int> sample;
sample.reserve(10);

std::mt19937 rng(std::random_device{}());
std::sample(population.begin(), population.end(),
            std::back_inserter(sample), 10, rng);

// sample contains 10 distinct elements from 1..100
std::cout << sample.size(); // 10
std::cout << std::is_sorted(sample.begin(), sample.end()); // true (preserves order)
```

---

### Q31. How do you use `std::for_each` with stateful lambdas?

**Answer:**

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// Accumulate with for_each
int sum = 0;
std::for_each(v.begin(), v.end(), [&sum](int x) { sum += x; });
std::cout << sum; // 15

// Stateful: track min/max simultaneously
int mn = INT_MAX, mx = INT_MIN;
std::for_each(v.begin(), v.end(), [&](int x) {
    mn = std::min(mn, x);
    mx = std::max(mx, x);
});
std::cout << mn << " " << mx; // 1 5

// for_each returns the functor (useful for stateful objects)
struct Printer {
    int count = 0;
    void operator()(int x) { std::cout << x << " "; ++count; }
};
auto p = std::for_each(v.begin(), v.end(), Printer{});
std::cout << "\nProcessed: " << p.count; // 5
```

---

### Q32. What is `std::adjacent_find` and `std::adjacent_difference`?

**Answer:**

```cpp
// adjacent_find: find first pair of adjacent equal/matching elements
std::vector<int> v = {1, 2, 2, 3, 4, 4, 5};
auto it = std::adjacent_find(v.begin(), v.end());
std::cout << *it;       // 2 (first element of first equal pair)
std::cout << std::distance(v.begin(), it); // 1

// Find first adjacent pair where second > first + 2 (gap check)
auto it2 = std::adjacent_find(v.begin(), v.end(),
    [](int a, int b){ return b - a > 1; });
std::cout << *it2; // 4 (pair 4→5 has gap but 4→4 comes first; check: 1→2, 2→2, 2→3 gap=1, 3→4, 4→4, 4→5 gap=1... no gap>1 here)

// adjacent_difference: compute differences between consecutive elements
#include <numeric>
std::vector<int> seq = {2, 5, 9, 14, 20};
std::vector<int> diffs(seq.size());
std::adjacent_difference(seq.begin(), seq.end(), diffs.begin());
for (int d : diffs) std::cout << d << " "; // 2 3 4 5 6
// First element is copied; rest are seq[i] - seq[i-1]
```

---

### Q33. What is `std::search` and `std::find_end`?

**Answer:**

```cpp
std::vector<int> haystack = {1, 2, 3, 4, 5, 2, 3, 6};
std::vector<int> needle   = {2, 3};

// search: find first occurrence of needle in haystack
auto it = std::search(haystack.begin(), haystack.end(),
                      needle.begin(), needle.end());
std::cout << std::distance(haystack.begin(), it); // 1 (index of first 2,3)

// find_end: find LAST occurrence
auto it2 = std::find_end(haystack.begin(), haystack.end(),
                         needle.begin(), needle.end());
std::cout << std::distance(haystack.begin(), it2); // 5 (index of last 2,3)

// search with predicate
auto it3 = std::search(haystack.begin(), haystack.end(),
                       needle.begin(), needle.end(),
                       [](int a, int b){ return a % 3 == b % 3; });
// Finds elements that match modulo 3

// search with Boyer-Moore (C++17) for faster string-like searches
// std::search(hay.begin(), hay.end(),
//             std::boyer_moore_searcher(needle.begin(), needle.end()));
```

---

### Q34. How do you use `std::min_element` and `std::max_element`?

**Answer:**

```cpp
std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5};

auto minIt = std::min_element(v.begin(), v.end());
auto maxIt = std::max_element(v.begin(), v.end());

std::cout << *minIt << " at index " << std::distance(v.begin(), minIt); // 1 at 1
std::cout << *maxIt << " at index " << std::distance(v.begin(), maxIt); // 9 at 5

// minmax_element: both in one pass
auto [lo, hi] = std::minmax_element(v.begin(), v.end());
std::cout << *lo << " " << *hi; // 1 9

// With custom comparator
std::vector<std::string> words = {"banana","apple","cherry","date"};
auto shortest = std::min_element(words.begin(), words.end(),
    [](const auto& a, const auto& b){ return a.size() < b.size(); });
std::cout << *shortest; // "date"

// clamp value to range
int x = 15;
std::cout << std::clamp(x, 0, 10); // 10 (clamped to max)
std::cout << std::clamp(x, 0, 20); // 15 (within range)
```

---

### Q35. What are execution policies in C++17 algorithms?

**Answer:**  
C++17 added execution policy tags to many algorithms to enable parallelism:

```cpp
#include <execution>

std::vector<int> v(10'000'000);
std::iota(v.begin(), v.end(), 0);

// Sequential (default, same as before)
std::sort(std::execution::seq, v.begin(), v.end());

// Parallel (may use multiple threads)
std::sort(std::execution::par, v.begin(), v.end());

// Parallel + vectorized (SIMD + threads)
std::sort(std::execution::par_unseq, v.begin(), v.end());

// Works with most algorithms
long long sum = std::reduce(std::execution::par,
                            v.begin(), v.end(), 0LL);

std::transform(std::execution::par_unseq,
               v.begin(), v.end(), v.begin(),
               [](int x){ return x * 2; });

// Note: par_unseq requires lambdas to be safe for SIMD vectorization
// (no locks, no external state mutation)
```

---

## Numeric Algorithms

---

### Q36. What is `std::accumulate` and its variations?

**Answer:**

```cpp
#include <numeric>
std::vector<int> v = {1, 2, 3, 4, 5};

// Sum
int sum = std::accumulate(v.begin(), v.end(), 0);           // 15

// Product
int prod = std::accumulate(v.begin(), v.end(), 1,
    std::multiplies<int>());                                 // 120

// String join
std::vector<std::string> words = {"hello","world","cpp"};
std::string joined = std::accumulate(
    std::next(words.begin()), words.end(), words[0],
    [](const std::string& a, const std::string& b) {
        return a + " " + b;
    });
std::cout << joined; // "hello world cpp"

// Max (prefer std::max_element but possible with accumulate)
int maxVal = std::accumulate(v.begin(), v.end(), INT_MIN,
    [](int a, int b){ return std::max(a, b); });
std::cout << maxVal; // 5

// Running product into new vector
std::vector<int> running(v.size());
std::partial_sum(v.begin(), v.end(), running.begin(),
                 std::multiplies<int>());
for (int x : running) std::cout << x << " "; // 1 2 6 24 120
```

---

### Q37. What is `std::partial_sum` and `std::exclusive_scan` / `std::inclusive_scan` (C++17)?

**Answer:**

```cpp
#include <numeric>
std::vector<int> v = {1, 2, 3, 4, 5};

// partial_sum: cumulative sum (inclusive)
std::vector<int> prefix(v.size());
std::partial_sum(v.begin(), v.end(), prefix.begin());
for (int x : prefix) std::cout << x << " "; // 1 3 6 10 15

// inclusive_scan (C++17): same as partial_sum but parallelizable
std::vector<int> inc(v.size());
std::inclusive_scan(v.begin(), v.end(), inc.begin());
for (int x : inc) std::cout << x << " "; // 1 3 6 10 15

// exclusive_scan (C++17): result[i] = sum of v[0..i-1] (excludes v[i])
std::vector<int> exc(v.size());
std::exclusive_scan(v.begin(), v.end(), exc.begin(), 0);
for (int x : exc) std::cout << x << " "; // 0 1 3 6 10

// Use case: prefix sum for range queries
// rangeSum(l, r) = prefix[r+1] - prefix[l]
std::vector<int> data = {3,1,4,1,5,9,2,6};
std::vector<int> ps(data.size() + 1, 0);
std::partial_sum(data.begin(), data.end(), ps.begin() + 1);
// Sum of data[2..5] = ps[6] - ps[2] = 20 - 4 = 16 (4+1+5+9 - wait: [2..5] = 4,1,5,9 = 19)
std::cout << ps[6] - ps[2]; // sum of indices 2..5 = 4+1+5+9 = 19
```

---

### Q38. What is `std::inner_product`?

**Answer:**  
Computes the inner (dot) product of two ranges, with customizable operations.

```cpp
#include <numeric>
std::vector<int> a = {1, 2, 3, 4, 5};
std::vector<int> b = {5, 4, 3, 2, 1};

// Default: sum of element-wise products (dot product)
int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0);
std::cout << dot; // 1*5+2*4+3*3+4*2+5*1 = 35

// Custom ops: sum of element-wise max
int sumMax = std::inner_product(a.begin(), a.end(), b.begin(), 0,
    std::plus<int>(),
    [](int x, int y){ return std::max(x, y); });
std::cout << sumMax; // 5+4+3+4+5 = 21

// Euclidean distance squared
int distSq = std::inner_product(a.begin(), a.end(), b.begin(), 0,
    std::plus<int>(),
    [](int x, int y){ return (x-y)*(x-y); });
std::cout << distSq; // (1-5)²+(2-4)²+... = 16+4+0+4+16 = 40
```

---

### Q39. What is `std::gcd`, `std::lcm`, and `std::midpoint` (C++17/C++20)?

**Answer:**

```cpp
#include <numeric>

// GCD and LCM
std::cout << std::gcd(12, 8);   // 4
std::cout << std::lcm(4, 6);    // 12

// Reduce a vector to GCD of all elements
std::vector<int> v = {12, 8, 4, 16};
int gcdAll = std::accumulate(v.begin(), v.end(), 0,
    [](int a, int b){ return std::gcd(a, b); });
std::cout << gcdAll; // 4

// midpoint (C++20): safe midpoint without overflow
int a = INT_MAX - 1, b = INT_MAX;
// int mid = (a + b) / 2; // OVERFLOW!
int mid = std::midpoint(a, b); // safe: INT_MAX - 1

// Works with pointers and iterators too
std::vector<int> vec = {1,2,3,4,5};
auto mptr = std::midpoint(vec.data(), vec.data() + vec.size());
std::cout << *mptr; // 3 (middle element)
```

---

### Q40. What is `std::iota` and how do you use it creatively?

**Answer:**

```cpp
#include <numeric>

// Basic: fill with sequential values
std::vector<int> v(10);
std::iota(v.begin(), v.end(), 0); // {0,1,2,...,9}
std::iota(v.begin(), v.end(), 1); // {1,2,3,...,10}

// Generate indices for an indirect sort
std::vector<std::string> words = {"banana","apple","cherry","date"};
std::vector<int> idx(words.size());
std::iota(idx.begin(), idx.end(), 0); // {0,1,2,3}

std::sort(idx.begin(), idx.end(),
    [&words](int a, int b){ return words[a] < words[b]; });
// idx = {1,0,3,2} (apple < banana < cherry < date... wait)
// idx now gives indices that would sort words alphabetically
for (int i : idx) std::cout << words[i] << " "; // apple banana cherry date

// Arithmetic progression (non-integer step workaround)
std::vector<double> range(11);
std::iota(range.begin(), range.end(), 0);
std::transform(range.begin(), range.end(), range.begin(),
               [](double x){ return x * 0.1; }); // {0.0, 0.1, ..., 1.0}
```

---

### Q41. What is `std::exchange` and how is it used with containers?

**Answer:**  
`std::exchange(obj, new_val)` assigns `new_val` to `obj` and returns the old value — atomic read+write semantics.

```cpp
#include <utility>

// Swap idiom in move constructors
class MyVec {
    int* data;
    size_t size;
public:
    MyVec(MyVec&& other) noexcept
        : data(std::exchange(other.data, nullptr)),
          size(std::exchange(other.size, 0)) {}
    // other.data = nullptr, other.size = 0, this gets old values
};

// Drain a vector (pop while processing)
std::vector<int> tasks = {1, 2, 3, 4, 5};
auto pending = std::exchange(tasks, {}); // atomically swap for empty
std::cout << tasks.size();   // 0
std::cout << pending.size(); // 5

// Counter reset
int processed = 0;
auto count = [&]{ return std::exchange(processed, 0); };
processed = 42;
std::cout << count();     // 42 (old value)
std::cout << processed;   // 0  (reset)
```

---

## Custom Comparators & Ordering

---

### Q42. How do you sort a `vector` of structs by multiple fields?

**Answer:**

```cpp
struct Employee {
    std::string dept;
    int salary;
    std::string name;
};

std::vector<Employee> employees = {
    {"Engineering", 90000, "Alice"},
    {"Marketing",   75000, "Bob"},
    {"Engineering", 85000, "Carol"},
    {"Marketing",   75000, "Alice"},
    {"Engineering", 90000, "Dave"},
};

// Sort: dept ascending, salary descending, name ascending
std::sort(employees.begin(), employees.end(),
    [](const Employee& a, const Employee& b) {
        if (a.dept != b.dept) return a.dept < b.dept;
        if (a.salary != b.salary) return a.salary > b.salary; // desc
        return a.name < b.name;
    });

for (auto& e : employees)
    std::cout << e.dept << " " << e.salary << " " << e.name << "\n";
// Engineering 90000 Alice
// Engineering 90000 Dave
// Engineering 85000 Carol
// Marketing   75000 Alice
// Marketing   75000 Bob
```

---

### Q43. What is `std::tie` and how does it simplify comparators?

**Answer:**  
`std::tie` creates a tuple of references, enabling lexicographic comparison of multiple fields.

```cpp
struct Point {
    int x, y, z;

    // Using std::tie for natural ordering
    bool operator<(const Point& other) const {
        return std::tie(x, y, z) < std::tie(other.x, other.y, other.z);
    }
    bool operator==(const Point& other) const {
        return std::tie(x, y, z) == std::tie(other.x, other.y, other.z);
    }
};

std::set<Point> pts;
pts.insert({1, 2, 3});
pts.insert({1, 2, 0});
pts.insert({0, 5, 5});
// Sorted: {0,5,5}, {1,2,0}, {1,2,3}

// Sort vector with tie
std::vector<std::tuple<int,int,std::string>> records = {
    {3, 2, "c"}, {1, 5, "a"}, {3, 1, "b"}, {1, 5, "z"}
};
std::sort(records.begin(), records.end()); // tuple comparison is lexicographic
for (auto& [a,b,c] : records)
    std::cout << a << b << c << " "; // 15a 15z 31b 32c
```

---

### Q44. How do you implement a `priority_queue` for a custom struct?

**Answer:**

```cpp
struct Job {
    int priority;
    int id;
    std::string name;

    // For use in priority_queue: greater priority = comes first
    // operator< defines the heap order (max-heap by default)
    bool operator<(const Job& other) const {
        if (priority != other.priority) return priority < other.priority;
        return id > other.id; // lower id = higher priority on tie
    }
};

std::priority_queue<Job> pq;
pq.push({3, 2, "Low-priority"});
pq.push({9, 1, "Critical"});
pq.push({9, 3, "Also critical"});
pq.push({5, 4, "Medium"});

while (!pq.empty()) {
    auto& j = pq.top();
    std::cout << j.name << " (P=" << j.priority << " id=" << j.id << ")\n";
    pq.pop();
}
// Critical (P=9 id=1)       ← same priority, lower id wins
// Also critical (P=9 id=3)
// Medium (P=5 id=4)
// Low-priority (P=3 id=2)
```

---

### Q45. What is `std::less`, `std::greater`, and transparent comparators?

**Answer:**

```cpp
// std::less<T>: default comparator (a < b)
std::set<int, std::less<int>> ascending  = {5,3,1,4,2}; // {1,2,3,4,5}
std::set<int, std::greater<int>> descending = {5,3,1,4,2}; // {5,4,3,2,1}

for (int x : descending) std::cout << x << " "; // 5 4 3 2 1

// Transparent comparators (C++14): std::less<void> / std::less<>
// Allow heterogeneous lookup without constructing the key type
std::set<std::string, std::less<>> strSet = {"apple","banana","cherry"};
// Lookup with string_view — no std::string construction
std::string_view sv = "banana";
auto it = strSet.find(sv); // works! (transparent)
std::cout << (it != strSet.end()); // true

// Also for map
std::map<std::string, int, std::less<>> m;
m["hello"] = 1;
auto it2 = m.find(std::string_view("hello")); // no string allocation
std::cout << it2->second; // 1
```

---

### Q46. How do you sort strings in a locale-aware way?

**Answer:**

```cpp
#include <locale>

std::vector<std::string> names = {"Önder","Alice","Ångström","Bob","Çelik"};

// Locale-aware comparison using std::locale
std::locale loc("en_US.UTF-8"); // or "de_DE.UTF-8" etc.
std::sort(names.begin(), names.end(),
    [&loc](const std::string& a, const std::string& b) {
        return loc(a, b); // locale's operator() compares strings
    });

for (const auto& n : names) std::cout << n << "\n";

// Case-insensitive sort
std::vector<std::string> words = {"Banana","apple","Cherry","date"};
std::sort(words.begin(), words.end(),
    [](const std::string& a, const std::string& b) {
        return std::lexicographical_compare(
            a.begin(), a.end(), b.begin(), b.end(),
            [](char ca, char cb) {
                return std::tolower(ca) < std::tolower(cb);
            });
    });
for (const auto& w : words) std::cout << w << " "; // apple Banana Cherry date
```

---

### Q47. How do you use `std::map` with a lambda comparator?

**Answer:**

```cpp
// Lambda comparator must be passed at construction (not inferred from type alone)
auto cmp = [](const std::string& a, const std::string& b) {
    return a.size() < b.size() || (a.size() == b.size() && a < b);
};

// Must use decltype for the type
std::map<std::string, int, decltype(cmp)> m(cmp);
m["hi"]     = 1;
m["hello"]  = 2;
m["hey"]    = 3;
m["howdy"]  = 4;

for (auto& [k, v] : m)
    std::cout << k << "(" << k.size() << ") "; // hi hey hello howdy

// C++20 approach: use a named struct instead (cleaner)
struct LenThenAlpha {
    bool operator()(const std::string& a, const std::string& b) const {
        return std::tie(a.size(), a) < std::tie(b.size(), b);
    }
};
std::map<std::string, int, LenThenAlpha> m2;
```

---

## Memory, Allocators & Performance

---

### Q48. What is an allocator and how does it affect STL containers?

**Answer:**  
An allocator is the memory management policy for a container. Every STL container accepts a custom allocator as its last template parameter. The default is `std::allocator<T>` (uses `new`/`delete`).

```cpp
// All containers: Container<T, Allocator = std::allocator<T>>
std::vector<int> v;                              // uses std::allocator
std::vector<int, std::allocator<int>> v2;        // same, explicit

// Custom allocator for stack-based storage (arena allocator concept)
template<typename T, size_t N>
struct StackAllocator {
    using value_type = T;
    alignas(alignof(T)) char buf[N * sizeof(T)];
    size_t pos = 0;

    T* allocate(size_t n) {
        if (pos + n > N) throw std::bad_alloc{};
        T* ptr = reinterpret_cast<T*>(buf + pos * sizeof(T));
        pos += n;
        return ptr;
    }
    void deallocate(T*, size_t) {} // no-op for arena

    template<typename U>
    struct rebind { using other = StackAllocator<U, N>; };
};
```

---

### Q49. What is `std::pmr` (Polymorphic Memory Resources, C++17)?

**Answer:**  
`std::pmr` containers use a `memory_resource*` instead of a template allocator, enabling runtime-selected allocation strategies without changing the container type.

```cpp
#include <memory_resource>

// Monotonic buffer: no deallocation until buffer is released — very fast
char buffer[1024];
std::pmr::monotonic_buffer_resource mbr(buffer, sizeof(buffer));

std::pmr::vector<int> v(&mbr);  // allocates from buffer, not heap
for (int i = 0; i < 100; ++i) v.push_back(i);
// All allocations from stack buffer — zero heap allocation!

// Pool resource: fixed-size block recycling
std::pmr::unsynchronized_pool_resource pool;
std::pmr::map<int, std::pmr::string> m(&pool);
m[1] = "one";
m[2] = "two";

// Chain resources: fall back to heap when buffer full
std::pmr::monotonic_buffer_resource bufRsrc(buffer, sizeof(buffer),
    std::pmr::get_default_resource()); // falls back to new/delete
```

---

### Q50. How does container memory layout affect cache performance?

**Answer:**

```cpp
// AoS vs SoA: critical for cache-sensitive code

struct ParticleAoS { float x, y, z, vx, vy, vz; }; // Array of Structs
struct ParticleSoA {
    std::vector<float> x, y, z, vx, vy, vz;         // Struct of Arrays
};

// If we only update positions: SoA is faster (only x,y,z in cache)
void updatePositionsAoS(std::vector<ParticleAoS>& p, float dt) {
    for (auto& particle : p) {
        particle.x += particle.vx * dt; // loads all 6 floats (24B) per particle
        particle.y += particle.vy * dt; // but only uses 4 of them
        particle.z += particle.vz * dt;
    }
}

void updatePositionsSoA(ParticleSoA& p, float dt) {
    for (size_t i = 0; i < p.x.size(); ++i) {
        p.x[i] += p.vx[i] * dt; // x and vx arrays: 8 bytes per iteration
        p.y[i] += p.vy[i] * dt; // perfectly cache-line utilized
        p.z[i] += p.vz[i] * dt;
    }
}
// SoA can be 2-4× faster for position-only updates due to cache efficiency
```

---

### Q51. How does `reserve()` prevent iterator invalidation in `vector`?

**Answer:**

```cpp
std::vector<int> v;

// Without reserve: each push_back may reallocate, invalidating ALL iterators
v.push_back(1);
auto it = v.begin(); // safe for now
v.push_back(2);      // may reallocate!
// *it is now potentially UB if reallocation occurred

// With reserve: no reallocation until capacity is exceeded
std::vector<int> v2;
v2.reserve(100);
auto it2 = v2.begin(); // safe
for (int i = 0; i < 100; ++i) v2.push_back(i); // no reallocation
// it2 still valid (points to v2[0])

// Pointer stability example
std::vector<int> v3;
v3.reserve(5);
v3.push_back(10);
int* ptr = &v3[0];
v3.push_back(20); v3.push_back(30); // no reallocation (capacity is 5)
std::cout << *ptr; // 10 — ptr still valid
v3.push_back(40); v3.push_back(50); v3.push_back(60); // reallocation!
// ptr is now dangling!
```

---

### Q52. What is small buffer optimization (SBO) in STL containers?

**Answer:**  
Many implementations apply SBO for small data — storing elements inline instead of heap-allocating, avoiding the pointer indirection and allocation overhead.

```cpp
// std::string typically uses SSO (Small String Optimization)
std::string small = "hi";      // stored inline (no heap) — typically < 15-22 chars
std::string large(100, 'x');   // heap allocated

// sizeof(std::string) is constant regardless of content
std::cout << sizeof(small); // 24 or 32 (platform-dependent)
std::cout << sizeof(large); // same!

// Check if string is using heap (implementation-defined)
// Some implementations expose this via capacity() < threshold

// std::any, std::function also use SBO:
std::function<int()> small_fn = []{ return 42; }; // stored inline
// Large captured state may force heap allocation

// For custom classes: inline buffer optimization
template<typename T, size_t N = 8>
class SmallVec {
    T inlineBuf[N];
    T* data;
    size_t size_, cap_;
    // use inlineBuf when size <= N, heap otherwise
};
```

---

### Q53. How do you benchmark STL container operations?

**Answer:**

```cpp
#include <chrono>
#include <iostream>

template<typename Func>
double benchmark(Func f, int iterations = 1000) {
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < iterations; ++i) f();
    auto end = std::chrono::high_resolution_clock::now();
    return std::chrono::duration<double, std::milli>(end - start).count()
           / iterations;
}

// Compare map vs unordered_map lookup
std::map<int,int>           omap;
std::unordered_map<int,int> umap;
const int N = 100000;
for (int i = 0; i < N; ++i) { omap[i] = i; umap[i] = i; }

double omapTime = benchmark([&](){
    volatile int v = 0;
    for (int i = 0; i < N; ++i) v += omap[i];
});
double umapTime = benchmark([&](){
    volatile int v = 0;
    for (int i = 0; i < N; ++i) v += umap[i];
});

std::cout << "map:          " << omapTime << " ms\n";
std::cout << "unordered_map:" << umapTime << " ms\n";
// unordered_map typically 3-10× faster for random access
```

---

### Q54. What is `std::deque` vs `std::vector` for push_front performance?

**Answer:**

```cpp
#include <chrono>
const int N = 100000;

// vector push_front: O(n) per operation — O(n²) total
auto vecFrontTime = benchmark([&](){
    std::vector<int> v;
    v.reserve(N);
    for (int i = 0; i < N; ++i)
        v.insert(v.begin(), i); // shifts all elements each time!
});

// deque push_front: O(1) per operation — O(n) total
auto dequeFrontTime = benchmark([&](){
    std::deque<int> d;
    for (int i = 0; i < N; ++i)
        d.push_front(i); // O(1) amortized
});

std::cout << "vector push_front: " << vecFrontTime << " ms\n"; // very slow
std::cout << "deque  push_front: " << dequeFrontTime << " ms\n"; // fast

// But deque is slower than vector for random access / iteration
// due to segmented memory (cache miss at segment boundaries)
```

---

## Concurrent Collections & Thread Safety

---

### Q55. Are STL containers thread-safe?

**Answer:**  
STL containers are **not thread-safe** for concurrent modifications. The rules are:
- Multiple concurrent reads: safe.
- Any concurrent write (even to different elements): unsafe without synchronization.
- Concurrent read + write: unsafe.

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// SAFE: multiple readers
std::thread t1([&v]{ std::cout << v[0]; });
std::thread t2([&v]{ std::cout << v[1]; });
t1.join(); t2.join(); // OK

// UNSAFE: concurrent write
std::thread t3([&v]{ v.push_back(6); }); // UB if any other thread reads/writes
std::thread t4([&v]{ v.push_back(7); }); // data race!
// t3.join(); t4.join(); // DO NOT DO THIS without mutex

// SAFE: protect with mutex
std::mutex mtx;
auto safePush = [&](int x) {
    std::lock_guard<std::mutex> lock(mtx);
    v.push_back(x);
};
std::thread t5([&]{ safePush(6); });
std::thread t6([&]{ safePush(7); });
t5.join(); t6.join(); // safe
```

---

### Q56. How do you implement a thread-safe queue?

**Answer:**

```cpp
#include <mutex>
#include <condition_variable>

template<typename T>
class BlockingQueue {
    std::queue<T> queue;
    mutable std::mutex mutex;
    std::condition_variable cv;
    bool closed = false;

public:
    void push(T value) {
        {
            std::lock_guard<std::mutex> lock(mutex);
            if (closed) throw std::runtime_error("Queue closed");
            queue.push(std::move(value));
        }
        cv.notify_one();
    }

    bool pop(T& value) {
        std::unique_lock<std::mutex> lock(mutex);
        cv.wait(lock, [this]{ return !queue.empty() || closed; });
        if (queue.empty()) return false; // closed and empty
        value = std::move(queue.front());
        queue.pop();
        return true;
    }

    void close() {
        std::lock_guard<std::mutex> lock(mutex);
        closed = true;
        cv.notify_all();
    }
};

BlockingQueue<int> bq;
std::thread producer([&]() {
    for (int i = 1; i <= 5; ++i) bq.push(i);
    bq.close();
});
std::thread consumer([&]() {
    int val;
    while (bq.pop(val)) std::cout << val << " "; // 1 2 3 4 5
});
producer.join(); consumer.join();
```

---

### Q57. What is `std::atomic` and how can it replace a mutex for simple cases?

**Answer:**

```cpp
#include <atomic>

// Atomic counter: no mutex needed for increment
std::atomic<int> counter{0};

std::vector<std::thread> threads;
for (int i = 0; i < 10; ++i)
    threads.emplace_back([&counter]() {
        for (int j = 0; j < 1000; ++j)
            ++counter; // atomic increment
    });
for (auto& t : threads) t.join();
std::cout << counter; // always 10000

// Atomic flag for one-shot signaling
std::atomic<bool> done{false};
std::thread worker([&done]() {
    // do work...
    done.store(true, std::memory_order_release);
});
while (!done.load(std::memory_order_acquire)) { /* wait */ }
worker.join();

// compare_exchange: lock-free stack push
struct Node { int val; Node* next; };
std::atomic<Node*> head{nullptr};
auto push = [&](int val) {
    Node* n = new Node{val, head.load()};
    while (!head.compare_exchange_weak(n->next, n));
};
```

---

### Q58. How do you implement a concurrent hash map with striped locking?

**Answer:**

```cpp
template<typename K, typename V, int STRIPES = 16>
class StripedHashMap {
    struct Stripe {
        std::unordered_map<K, V> map;
        mutable std::shared_mutex mutex;
    };
    std::array<Stripe, STRIPES> stripes;

    Stripe& getStripe(const K& key) {
        return stripes[std::hash<K>{}(key) % STRIPES];
    }

public:
    void insert(const K& key, const V& value) {
        auto& stripe = getStripe(key);
        std::unique_lock lock(stripe.mutex);
        stripe.map[key] = value;
    }

    std::optional<V> get(const K& key) const {
        auto& stripe = const_cast<Stripe&>(
            const_cast<StripedHashMap*>(this)->getStripe(key));
        std::shared_lock lock(stripe.mutex);
        auto it = stripe.map.find(key);
        if (it == stripe.map.end()) return std::nullopt;
        return it->second;
    }
};

StripedHashMap<int, std::string> shm;
shm.insert(1, "one");
shm.insert(2, "two");
auto v = shm.get(1);
if (v) std::cout << *v; // "one"
// 16 independent locks: 16× less contention than a single lock
```

---

### Q59. What is a `std::shared_mutex` and when do you use it with containers?

**Answer:**  
`shared_mutex` allows multiple concurrent readers (shared lock) but exclusive writers (unique lock) — ideal for read-heavy containers.

```cpp
#include <shared_mutex>

class ReadHeavyMap {
    std::unordered_map<std::string, int> data;
    mutable std::shared_mutex rwLock;

public:
    // Read: multiple threads can read simultaneously
    int get(const std::string& key) const {
        std::shared_lock lock(rwLock);   // shared — multiple readers OK
        auto it = data.find(key);
        return it == data.end() ? -1 : it->second;
    }

    // Write: exclusive access
    void set(const std::string& key, int value) {
        std::unique_lock lock(rwLock);   // exclusive — blocks all readers
        data[key] = value;
    }

    size_t size() const {
        std::shared_lock lock(rwLock);
        return data.size();
    }
};

ReadHeavyMap rhm;
rhm.set("x", 42);

// Many concurrent readers:
std::vector<std::thread> readers;
for (int i = 0; i < 10; ++i)
    readers.emplace_back([&rhm]{ std::cout << rhm.get("x"); });
for (auto& t : readers) t.join(); // all print 42 concurrently
```

---

### Q60. What is the difference between `std::queue` and a lock-free queue?

**Answer:**

```cpp
// std::queue: NOT thread-safe, needs mutex
std::queue<int> q;
std::mutex qMtx;
q.push(1); // thread-unsafe without lock

// Lock-based thread-safe queue (simple but may contend)
auto push = [&](int x) {
    std::lock_guard<std::mutex> lock(qMtx);
    q.push(x);
};

// Lock-free queue concept using atomics (simplified single-producer single-consumer)
template<typename T, size_t Cap>
class SPSCQueue {
    std::array<T, Cap> buf;
    std::atomic<size_t> head{0}, tail{0};
public:
    bool push(const T& val) {
        size_t t = tail.load(std::memory_order_relaxed);
        size_t next = (t + 1) % Cap;
        if (next == head.load(std::memory_order_acquire)) return false; // full
        buf[t] = val;
        tail.store(next, std::memory_order_release);
        return true;
    }
    bool pop(T& val) {
        size_t h = head.load(std::memory_order_relaxed);
        if (h == tail.load(std::memory_order_acquire)) return false; // empty
        val = buf[h];
        head.store((h + 1) % Cap, std::memory_order_release);
        return true;
    }
};
```

---

## Common Interview Coding Patterns

---

### Q61. Two Sum — optimal solution with `unordered_map`

**Answer:**

```cpp
// Given an array and target, find indices of two numbers that add to target
std::pair<int,int> twoSum(const std::vector<int>& nums, int target) {
    std::unordered_map<int, int> seen; // value → index
    for (int i = 0; i < (int)nums.size(); ++i) {
        int complement = target - nums[i];
        auto it = seen.find(complement);
        if (it != seen.end())
            return {it->second, i};
        seen[nums[i]] = i;
    }
    return {-1, -1}; // no solution
}

auto [i, j] = twoSum({2, 7, 11, 15}, 9);
std::cout << i << " " << j; // 0 1 (nums[0]+nums[1]=9)
```

---

### Q62. Longest substring without repeating characters — `unordered_map` sliding window

**Answer:**

```cpp
int lengthOfLongestSubstring(const std::string& s) {
    std::unordered_map<char, int> lastSeen; // char → last seen index
    int maxLen = 0, left = 0;

    for (int right = 0; right < (int)s.size(); ++right) {
        char c = s[right];
        auto it = lastSeen.find(c);
        if (it != lastSeen.end() && it->second >= left)
            left = it->second + 1; // shrink window past duplicate

        lastSeen[c] = right;
        maxLen = std::max(maxLen, right - left + 1);
    }
    return maxLen;
}

std::cout << lengthOfLongestSubstring("abcabcbb"); // 3 ("abc")
std::cout << lengthOfLongestSubstring("pwwkew");   // 3 ("wke")
std::cout << lengthOfLongestSubstring("bbbbb");    // 1 ("b")
```

---

### Q63. Group elements by frequency using `map` + `vector`

**Answer:**

```cpp
// Top K frequent elements
std::vector<int> topKFrequent(std::vector<int>& nums, int k) {
    // Count frequencies
    std::unordered_map<int, int> freq;
    for (int n : nums) ++freq[n];

    // Bucket sort by frequency
    int n = nums.size();
    std::vector<std::vector<int>> buckets(n + 1);
    for (auto& [val, cnt] : freq)
        buckets[cnt].push_back(val);

    // Collect top-k from highest frequency buckets
    std::vector<int> result;
    for (int i = n; i >= 0 && (int)result.size() < k; --i)
        for (int v : buckets[i])
            if ((int)result.size() < k)
                result.push_back(v);

    return result;
}

std::vector<int> nums = {1,1,1,2,2,3};
auto top = topKFrequent(nums, 2);
for (int x : top) std::cout << x << " "; // 1 2
```

---

### Q64. Implement a trie using `unordered_map`

**Answer:**

```cpp
struct TrieNode {
    std::unordered_map<char, std::unique_ptr<TrieNode>> children;
    bool isEnd = false;
};

class Trie {
    std::unique_ptr<TrieNode> root;
public:
    Trie() : root(std::make_unique<TrieNode>()) {}

    void insert(const std::string& word) {
        TrieNode* node = root.get();
        for (char c : word) {
            if (!node->children.count(c))
                node->children[c] = std::make_unique<TrieNode>();
            node = node->children[c].get();
        }
        node->isEnd = true;
    }

    bool search(const std::string& word) const {
        const TrieNode* node = root.get();
        for (char c : word) {
            auto it = node->children.find(c);
            if (it == node->children.end()) return false;
            node = it->second.get();
        }
        return node->isEnd;
    }

    bool startsWith(const std::string& prefix) const {
        const TrieNode* node = root.get();
        for (char c : prefix) {
            auto it = node->children.find(c);
            if (it == node->children.end()) return false;
            node = it->second.get();
        }
        return true;
    }
};

Trie trie;
trie.insert("apple"); trie.insert("app");
std::cout << trie.search("apple");    // true
std::cout << trie.search("app");      // true
std::cout << trie.search("ap");       // false
std::cout << trie.startsWith("ap");   // true
```

---

### Q65. Find all pairs with a given difference using `set`

**Answer:**

```cpp
std::vector<std::pair<int,int>> pairsWithDiff(
        std::vector<int> nums, int k) {
    std::set<int> seen(nums.begin(), nums.end());
    std::vector<std::pair<int,int>> result;

    for (int x : seen) {
        // Check if x + k exists in set
        if (seen.count(x + k))
            result.emplace_back(x, x + k);
    }
    return result;
}

auto pairs = pairsWithDiff({1,5,3,4,2}, 2);
for (auto [a,b] : pairs)
    std::cout << "(" << a << "," << b << ") ";
// (1,3) (2,4) (3,5)
```

---

### Q66. Subarray sum equals K using prefix sum + `unordered_map`

**Answer:**

```cpp
int subarraySum(const std::vector<int>& nums, int k) {
    std::unordered_map<int, int> prefixCount;
    prefixCount[0] = 1; // empty prefix
    int sum = 0, count = 0;

    for (int x : nums) {
        sum += x;
        // If (sum - k) was seen before, there's a subarray summing to k
        auto it = prefixCount.find(sum - k);
        if (it != prefixCount.end())
            count += it->second;
        ++prefixCount[sum];
    }
    return count;
}

std::cout << subarraySum({1, 1, 1}, 2);       // 2 ([1,1] at 0..1 and 1..2)
std::cout << subarraySum({1, 2, 3, -3, 3}, 3); // 4
```

---

### Q67. Implement a stack using two queues

**Answer:**

```cpp
class StackFromQueues {
    std::queue<int> q1, q2;
public:
    void push(int x) {
        q2.push(x);
        while (!q1.empty()) { q2.push(q1.front()); q1.pop(); }
        std::swap(q1, q2);
    }
    int pop() {
        int val = q1.front(); q1.pop(); return val;
    }
    int top() const { return q1.front(); }
    bool empty() const { return q1.empty(); }
};

StackFromQueues s;
s.push(1); s.push(2); s.push(3);
std::cout << s.top(); // 3 (LIFO)
s.pop();
std::cout << s.top(); // 2
```

---

### Q68. Find the median of a data stream using two heaps

**Answer:**

```cpp
class MedianFinder {
    std::priority_queue<int> maxHeap;                           // lower half
    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap; // upper half

public:
    void addNum(int num) {
        maxHeap.push(num);
        // Balance: move max of lower half to upper half
        minHeap.push(maxHeap.top()); maxHeap.pop();
        // Keep maxHeap size >= minHeap size
        if (maxHeap.size() < minHeap.size()) {
            maxHeap.push(minHeap.top()); minHeap.pop();
        }
    }

    double findMedian() const {
        if (maxHeap.size() > minHeap.size())
            return maxHeap.top();
        return (maxHeap.top() + minHeap.top()) / 2.0;
    }
};

MedianFinder mf;
mf.addNum(1); mf.addNum(2);
std::cout << mf.findMedian(); // 1.5
mf.addNum(3);
std::cout << mf.findMedian(); // 2.0
```

---

### Q69. Reconstruct a sequence from a `map`-based graph (topological sort)

**Answer:**

```cpp
std::vector<int> topologicalSort(
        int n,
        const std::vector<std::pair<int,int>>& edges) {
    std::vector<int> inDegree(n, 0);
    std::map<int, std::vector<int>> adj;

    for (auto [u, v] : edges) {
        adj[u].push_back(v);
        ++inDegree[v];
    }

    // Min-heap for lexicographically smallest result
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;
    for (int i = 0; i < n; ++i)
        if (inDegree[i] == 0) pq.push(i);

    std::vector<int> result;
    while (!pq.empty()) {
        int node = pq.top(); pq.pop();
        result.push_back(node);
        for (int neighbor : adj[node])
            if (--inDegree[neighbor] == 0) pq.push(neighbor);
    }
    return result.size() == (size_t)n ? result : {}; // empty if cycle
}

auto order = topologicalSort(6,
    {{5,2},{5,0},{4,0},{4,1},{2,3},{3,1}});
for (int x : order) std::cout << x << " "; // 4 5 0 2 3 1
```

---

### Q70. Count distinct elements in every window of size K

**Answer:**

```cpp
std::vector<int> countDistinct(const std::vector<int>& arr, int k) {
    std::unordered_map<int, int> window; // element → count in window
    std::vector<int> result;

    for (int i = 0; i < (int)arr.size(); ++i) {
        ++window[arr[i]];

        // Remove element that left the window
        if (i >= k) {
            int old = arr[i - k];
            if (--window[old] == 0)
                window.erase(old);
        }

        // Record distinct count for full windows
        if (i >= k - 1)
            result.push_back(window.size());
    }
    return result;
}

auto res = countDistinct({1, 2, 1, 3, 4, 2, 3}, 4);
for (int x : res) std::cout << x << " "; // 3 4 4 3
```

---

### Q71. Serialize and deserialize a `vector<vector<int>>`

**Answer:**

```cpp
// Simple length-prefixed serialization
std::string serialize(const std::vector<std::vector<int>>& grid) {
    std::ostringstream oss;
    oss << grid.size() << '\n';
    for (const auto& row : grid) {
        oss << row.size() << '\n';
        for (int x : row) oss << x << ' ';
        oss << '\n';
    }
    return oss.str();
}

std::vector<std::vector<int>> deserialize(const std::string& data) {
    std::istringstream iss(data);
    int rows; iss >> rows;
    std::vector<std::vector<int>> grid(rows);
    for (auto& row : grid) {
        int cols; iss >> cols;
        row.resize(cols);
        for (int& x : row) iss >> x;
    }
    return grid;
}

std::vector<std::vector<int>> original = {{1,2,3},{4,5},{6,7,8,9}};
auto s = serialize(original);
auto restored = deserialize(s);
std::cout << (original == restored); // true (if sizes match)
```

---

### Q72. Find the first non-repeating character using `map` + `queue`

**Answer:**

```cpp
// Streaming: find first non-repeating char at each step
std::vector<char> firstNonRepeating(const std::string& stream) {
    std::unordered_map<char, int> freq;
    std::queue<char> q; // maintains order of first occurrence
    std::vector<char> result;

    for (char c : stream) {
        ++freq[c];
        q.push(c);

        // Remove from front all chars that are now repeated
        while (!q.empty() && freq[q.front()] > 1)
            q.pop();

        result.push_back(q.empty() ? '#' : q.front());
    }
    return result;
}

auto res = firstNonRepeating("aabc");
for (char c : res) std::cout << c << " "; // # # b b
// After 'a':  freq{a:1} → first non-rep = 'a'... wait:
// 'a'→a:1 → 'a'
// 'a'→a:2 → '#' (a repeats)
// 'b'→b:1 → 'b'
// 'c'→c:1 → 'b' (still b first)
```

---

## C++17 / C++20 / C++23 STL Features

---

### Q73. What new container/algorithm features came in C++17?

**Answer:**

```cpp
// 1. std::optional
std::optional<int> maybeFind(const std::vector<int>& v, int target) {
    auto it = std::find(v.begin(), v.end(), target);
    if (it == v.end()) return std::nullopt;
    return *it;
}

// 2. std::string_view — non-owning string reference
void processName(std::string_view name) { /* no copy */ }

// 3. Structured bindings with maps
std::map<std::string, int> m = {{"a",1},{"b",2}};
for (auto& [key, val] : m) std::cout << key << val;

// 4. std::variant — type-safe union
std::variant<int, double, std::string> v = 42;
v = "hello";
std::cout << std::get<std::string>(v);
// or visit
std::visit([](auto&& x){ std::cout << x; }, v);

// 5. std::any — type-erased container
std::any any = 42;
any = std::string("hello");
std::cout << std::any_cast<std::string>(any);

// 6. map::merge(), map::extract()
std::map<int,int> m1 = {{1,10},{2,20}};
std::map<int,int> m2 = {{3,30},{4,40}};
m1.merge(m2); // m2 elements moved into m1
```

---

### Q74. What is `std::map::extract` (C++17) and why is it useful?

**Answer:**  
`extract` removes a node from a map and returns a node handle. You can modify the key (previously impossible) and reinsert it — without any allocation/deallocation.

```cpp
std::map<int, std::string> m = {{1,"a"},{2,"b"},{3,"c"}};

// Extract a node by key
auto node = m.extract(2);           // removes key 2
std::cout << node.key();            // 2
std::cout << node.mapped();         // "b"
std::cout << m.size();              // 2 (key 2 gone)

// Modify the key (impossible without extract before C++17)
node.key() = 99;
m.insert(std::move(node));          // reinsert with new key
std::cout << m[99];                 // "b"

// Move nodes between maps without allocation
std::map<int, std::string> m2;
m2.insert(m.extract(1));            // zero copy!
std::cout << m2[1];                 // "a"
```

---

### Q75. What are C++20 ranges and views?

**Answer:**

```cpp
#include <ranges>
#include <vector>

std::vector<int> v = {1,2,3,4,5,6,7,8,9,10};

// Composable, lazy pipeline
auto result = v
    | std::views::filter([](int x){ return x % 2 == 0; })  // evens
    | std::views::transform([](int x){ return x * x; })    // squared
    | std::views::reverse                                   // reverse
    | std::views::take(3);                                  // first 3

for (int x : result) std::cout << x << " "; // 100 64 36

// iota_view: lazy range
for (int x : std::views::iota(1, 11))       // {1..10}
    std::cout << x << " ";

// keys and values views on maps
std::map<std::string, int> m = {{"a",1},{"b",2},{"c",3}};
for (auto& k : m | std::views::keys)
    std::cout << k << " ";   // a b c
for (auto& v : m | std::views::values)
    std::cout << v << " ";   // 1 2 3

// zip_view (C++23)
// auto zipped = std::views::zip(v1, v2);
```

---

### Q76. What are C++20 `std::erase` and `std::erase_if`?

**Answer:**  
C++20 added free functions to replace the verbose erase-remove idiom.

```cpp
#include <vector>
#include <list>
#include <map>

// vector: replaces erase-remove idiom
std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};
std::erase(v, 2);                         // remove all 2s
for (int x : v) std::cout << x << " ";   // 1 3 4 5

// erase_if: remove by predicate
std::vector<int> v2 = {1,2,3,4,5,6,7,8};
std::erase_if(v2, [](int x){ return x % 2 == 0; }); // remove evens
for (int x : v2) std::cout << x << " ";  // 1 3 5 7

// Works for list too (replaces list::remove)
std::list<int> lst = {1,2,3,2,4};
std::erase(lst, 2);

// Works for map (erase by predicate)
std::map<int,int> m = {{1,10},{2,20},{3,30},{4,40}};
std::erase_if(m, [](const auto& p){ return p.second > 20; });
// m = {{1,10},{2,20}}
for (auto& [k,v] : m) std::cout << k << ":" << v << " "; // 1:10 2:20
```

---

### Q77. What is `std::span` and how does it replace pointer+size patterns?

**Answer:**

```cpp
#include <span>

// Old C style
void processOld(int* arr, size_t n) {
    for (size_t i = 0; i < n; ++i) arr[i] *= 2;
}

// Modern: std::span
void processNew(std::span<int> data) {
    for (int& x : data) x *= 2;
    std::cout << "size=" << data.size() << "\n";
}

int arr[] = {1, 2, 3, 4, 5};
std::vector<int> v = {1, 2, 3, 4, 5};
std::array<int, 5> a = {1, 2, 3, 4, 5};

processNew(arr);   // works
processNew(v);     // works
processNew(a);     // works

// Sub-span
processNew(std::span<int>(v).subspan(1, 3)); // elements [1..3]

// Const span (read-only view)
void readOnly(std::span<const int> data) {
    for (int x : data) std::cout << x;
    // data[0] = 99; // Error: const element
}
readOnly(v);
```

---

### Q78. What is `std::ranges::to` (C++23)?

**Answer:**  
Converts any range/view into a container. Previously you had to manually construct.

```cpp
#include <ranges>

std::vector<int> v = {1,2,3,4,5,6,7,8,9,10};

// C++23: collect range result directly into a container
auto evens = v
    | std::views::filter([](int x){ return x % 2 == 0; })
    | std::ranges::to<std::vector>();                        // C++23

for (int x : evens) std::cout << x << " "; // 2 4 6 8 10

// Into a set
auto sorted_unique = v
    | std::views::transform([](int x){ return x % 3; })
    | std::ranges::to<std::set>();

// Into a map (using zip)
// auto m = std::views::zip(keys, vals) | std::ranges::to<std::map>();

// Pre-C++23 workaround:
std::vector<int> evens2;
auto view = v | std::views::filter([](int x){ return x % 2 == 0; });
std::ranges::copy(view, std::back_inserter(evens2));
```

---

### Q79. What is `std::flat_map` vs `std::map` (C++23)?

**Answer:**

```cpp
#include <flat_map>  // C++23

// flat_map: sorted contiguous storage (vector-backed), not tree
std::flat_map<std::string, int> fm;
fm.insert({"banana", 2});
fm.insert({"apple",  1});
fm.insert({"cherry", 3});

// Same interface as map: sorted iteration
for (auto& [k, v] : fm)
    std::cout << k << ": " << v << "\n"; // alpha order

// Better cache performance than map for iteration and range queries
// Worse for insertion/deletion (O(n) shifts vs O(log n))

// Choose flat_map when:
// - Read-heavy (build once, query many times)
// - Iteration-heavy (cache friendliness matters)
// - Small-medium size (< ~1000 elements)

// Choose map when:
// - Write-heavy (frequent insert/delete)
// - Large size (avoid O(n) shifts)

std::flat_set<int> fs = {5, 3, 1, 4, 2};
for (int x : fs) std::cout << x << " "; // 1 2 3 4 5
std::cout << fs.contains(3); // true
```

---

### Q80. What is `std::generator` (C++23) and how does it relate to lazy sequences?

**Answer:**  
`std::generator` is a coroutine-based range that produces values lazily — like Python generators. It works with range algorithms.

```cpp
#include <generator> // C++23
#include <ranges>

// Infinite Fibonacci sequence
std::generator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto tmp = a + b;
        a = b;
        b = tmp;
    }
}

// Take first 10 Fibonacci numbers
for (int x : fibonacci() | std::views::take(10))
    std::cout << x << " "; // 0 1 1 2 3 5 8 13 21 34

// Range of primes
std::generator<int> primes() {
    std::vector<int> found;
    for (int n = 2; ; ++n) {
        bool isPrime = std::none_of(found.begin(), found.end(),
            [n](int p){ return n % p == 0; });
        if (isPrime) {
            found.push_back(n);
            co_yield n;
        }
    }
}

for (int p : primes() | std::views::take(8))
    std::cout << p << " "; // 2 3 5 7 11 13 17 19
```

---

## Quick Reference: Part 2 Summary

| Topic | Key Tools |
|---|---|
| Deduplication | `set`, `unordered_set`, sort+unique, frequency map |
| Sliding window | `deque` (monotonic), `unordered_map` (frequency) |
| Top-K | `priority_queue` (min-heap), `nth_element`, `partial_sort` |
| Intervals | `set<pair>`, `lower_bound`/`upper_bound` |
| LRU Cache | `list` + `unordered_map` |
| Frequency count | `map`/`unordered_map` + `vector<pair>` for sort |
| Graph algorithms | `priority_queue` (Dijkstra), `queue` (BFS), `stack` (DFS) |
| Streams | `unordered_map` (running count), two heaps (median) |
| Thread safety | `mutex`, `shared_mutex`, `atomic`, striped locks |
| Modern C++ | `ranges`, `views`, `span`, `optional`, `flat_map` |

---

*With both parts combined — 160 questions — you have comprehensive STL mastery for any C++ interview!*
