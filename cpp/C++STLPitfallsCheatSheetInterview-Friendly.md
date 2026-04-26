# C\+\+ STL Pitfalls Cheat Sheet \(Interview\-Friendly\)

**Goal:** Avoid common STL bugs, undefined behavior \(UB\), and inefficiencies\. Short “Bug → Fix” pairs \+ key notes for quick recall\.

## Core Pitfalls \(Original 6 \+ 4 Extra\)

### 1\. map\[\] Inserts Unintentionally

**Bug:** Accessing map\[key\] inserts the key with default value \(e\.g\., 0 for int\), even if you only want to check existence\.

```cpp
// Bug
std::map<std::string, int> m;
if (m["key"] == 0) { /* key inserted! */ }
```

**Fix:** Use find\(\) \(C\+\+11\+\) or contains\(\) \(C\+\+20\+\)

```cpp
// Safe
if (m.find("key") != m.end()) { /* no insertion */ }
// Or (C++20+)
if (m.contains("key")) { ... }
```

**Note:** Only use map\[\] when you *intend* to insert/modify the key\.

### 2\. Vector Iterators Invalidated During Modification

**Bug:** push\_back\(\)/insert\(\) can invalidate all iterators \(reallocates memory\), leading to crash/UB\.

```cpp
// Bug
std::vector<int> v = {1,2,3};
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 3) v.push_back(99); // it is invalid!
}
```

**Fix:** Use index\-based loop or collect changes first

```cpp
// Fix 1: Index-based
for (size_t i = 0; i < v.size(); ++i) {
    if (v[i] == 3) v.push_back(99);
}
// Fix 2: Collect first
std::vector<int> to_add;
for (int num : v) if (num == 3) to_add.push_back(99);
for (int num : to_add) v.push_back(num);
```

### 3\. erase\-remove Idiom Forgotten

**Bug:** std::remove\(\) only shifts elements \(doesn’t erase\) → vector size stays the same \(garbage at the end\)\.

```cpp
// Bug
std::vector<int> v = {1,2,3,2,4};
std::remove(v.begin(), v.end(), 2); // size still 5!
```

**Fix:** Pair remove\(\) with erase\(\) \(erase\-remove idiom\)

```cpp
// Safe
v.erase(std::remove(v.begin(), v.end(), 2), v.end()); // size = 3
```

**Note:** Works for all sequence containers \(vector, deque, list\)\.

### 4\. size\(\) is Unsigned → Underflow

**Bug:** size\(\) returns unsigned \(size\_t\); size\(\) \- 1 underflows to a huge number if container is empty \(UB/infinite loop\)\.

```cpp
// Bug (if v is empty)
std::vector<int> v;
for (int i = 0; i <= v.size() - 1; ++i) { /* infinite loop */ }
```

**Fix:** Avoid size\(\)\-1, use range\-based for, or cast to int \(carefully\)

```cpp
// Safe Options
for (size_t i = 0; i < v.size(); ++i) { ... } // No math
for (int num : v) { ... } // Best (C++11+)
for (int i = 0; i <= (int)v.size() - 1; ++i) { ... } // Cast (safe if size ≤ INT_MAX)
```

### 5\. Accidental Deep Copy of Large Containers

**Bug:** Auto copies large containers \(e\.g\., map of vectors\) → slow, memory\-heavy \(O\(n\) time\)\.

```cpp
// Bug (full deep copy)
std::map<int, std::vector<int>> bigMap;
auto copy = bigMap; // O(n) time/memory
```

**Fix:** Use const reference \(O\(1\) time, no copy\)

```cpp
// Safe
const auto& ref = bigMap; // Read-only, no copy
// Use ref like normal (e.g., ref[0], ref.find(...))
```

**Note:** Use \&amp; for non\-const if you need to modify \(still no copy\)\.

### 6\. unordered\_map Hash Collisions \+ Rehashing

**Bug:** No preallocation → frequent rehashing \+ collisions → O\(n\) worst\-case lookups/inserts\.

```cpp
// Bug (many rehashes for 10k elements)
std::unordered_map<int, int> um;
for (int i = 0; i < 10000; ++i) um[i] = i;
```

**Fix:** Use reserve\(\) to preallocate space

```cpp
// Safe
std::unordered_map<int, int> um;
um.reserve(10000); // Pre-size to avoid rehashing
for (int i = 0; i < 10000; ++i) um[i] = i;
```

**Note:** Reserve \~1\.2x the number of elements for best performance\.

### 7\. Dangling string::c\_str\(\)

**Bug:** c\_str\(\) returns a pointer to the string’s internal buffer — if the string is destroyed, the pointer dangles \(UB\)\.

```cpp
// Bug
const char* ptr = std::string("hello").c_str();
// String is destroyed → ptr dangles!
```

**Fix:** Ensure the string outlives the pointer

```cpp
// Safe
std::string str = "hello";
const char* ptr = str.c_str(); // str exists → ptr is valid
```

### 8\. auto Drops const/References

**Bug:** auto deduces values, not const/references — accidental copies or unintended modifications\.

```cpp
// Bug (auto = int, not const int&)
const std::vector<int> v = {1,2,3};
for (auto num : v) { num = 4; } // No error, but does nothing (copy)
```

**Fix:** Use const auto\&amp; for read\-only access

```cpp
// Safe
for (const auto& num : v) { ... } // Read-only, no copy
// Use auto& if you need to modify (non-const containers)
```

### 9\. emplace vs insert \(Inefficiency\)

**Bug:** Using insert\(\) with temporary objects → extra copy/move \(inefficient for large types\)\.

```cpp
// Bug (inserts temp string → extra move)
std::set<std::string> s;
s.insert("hello world"); // Creates temp string, then moves it
```

**Fix:** Use emplace\(\) to construct in\-place \(no temp\)

```cpp
// Safe (constructs string directly in the set)
s.emplace("hello world");
```

**Note:** emplace\(\) works for all associative containers \(set, map, unordered\_\*\)\.

### 10\. Using vector::front\(\)/back\(\) on Empty Containers

**Bug:** front\(\)/back\(\) return references to the first/last element — UB if the vector is empty\.

```cpp
// Bug (empty vector)
std::vector<int> v;
int x = v.front(); // UB (no element to reference)
```

**Fix:** Check empty\(\) first

```cpp
// Safe
if (!v.empty()) {
    int x = v.front();
}
```

## Key Interview Takeaways

- Always avoid map\[\] for*lookup* \(use find\(\)/contains\(\)\)\.

- Never modify a vector while iterating with iterators \(use indices\)\.

- erase\(\) \+ remove\(\) is the only way to truly delete elements from a vector\.

- size\(\) is unsigned — never use size\(\) \- 1 \(risk of underflow\)\.

- Use const\&amp; for large containers to avoid accidental copies\.

- reserve\(\) is critical for unordered\_map performance \(avoids rehashing\)\.

**Pro Tip:** These pitfalls are *extremely common in interviews* — memorize the “bug → fix” pairs and explain why the bug happens \(e\.g\., “map\[\] inserts because it’s designed for access/modification, not lookup”\)\.

## Extra Common Programming Pitfalls \(Beyond STL\)

### 11\. Null Pointer Dereferencing

**Bug:** Dereferencing a null pointer \(or uninitialized pointer\) leads to UB \(crash, unpredictable behavior\)\.

```cpp
// Bug
int* ptr = nullptr;
int x = *ptr; // UB (dereferencing null)
```

**Fix:** Check for null before dereferencing; use smart pointers \(C\+\+11\+\) to avoid manual pointer management\.

```cpp
// Safe Option 1: Null check
int* ptr = nullptr;
if (ptr != nullptr) {
    int x = *ptr;
}
// Safe Option 2: Smart pointer (unique_ptr/shared_ptr)
std::unique_ptr<int> ptr = std::make_unique<int>(5);
int x = *ptr; // No null risk
```

**Note:** Smart pointers automatically manage memory \(avoid leaks\) and reduce null pointer risks\.

### 12\. Integer Overflow/Underflow

**Bug:** Integer values exceed their maximum/minimum range \(e\.g\., int max \+ 1\) → UB \(wrap\-around, incorrect values\)\.

```cpp
// Bug
int max_int = INT_MAX;
int overflow = max_int + 1; // UB (overflow)
int min_int = INT_MIN;
int underflow = min_int - 1; // UB (underflow)
```

**Fix:** Use larger integer types \(e\.g\., long long\) or check for overflow before operations; use C\+\+20 std::cmp\_add/cmp\_sub for safe comparisons\.

```cpp
// Safe
long long max_int = INT_MAX;
long long overflow = max_int + 1; // No overflow (larger type)
// Or (C++20)
if (std::cmp_add_overflow(max_int, 1, &overflow)) {
    // Handle overflow
}
```

### 13\. Off\-by\-One Errors \(Loop Bounds\)

**Bug:** Loop runs one extra time or one fewer time \(e\.g\., using \&lt;= instead of \&lt; for 0\-based indices\), leading to out\-of\-bounds access or incomplete processing\.

```cpp
// Bug (off-by-one: accesses v[5] which is out of bounds)
std::vector<int> v = {1,2,3,4,5};
for (int i = 0; i <= 5; ++i) { // Should be i < 5
    std::cout << v[i];
}
```

**Fix:** Use 0\-based bounds \(i \&lt; container\.size\(\)\) or range\-based for; avoid hardcoding indices\.

```cpp
// Safe
// Option 1: Use size()
for (int i = 0; i < v.size(); ++i) {
    std::cout << v[i];
}
// Option 2: Range-based for (best)
for (int num : v) {
    std::cout << num;
}
```

### 14\. Race Conditions \(Concurrent Programming\)

**Bug:** Multiple threads access/modify the same data without synchronization → UB \(data corruption, inconsistent results\)\.

```cpp
// Bug (race condition: two threads modify count)
int count = 0;
std::thread t1([&]() { for (int i=0; i<1000; ++i) count++; });
std::thread t2([&]() { for (int i=0; i<1000; ++i) count++; });
t1.join(); t2.join();
// count may be less than 2000 (data corruption)
```

**Fix:** Use mutexes, atomic variables, or other synchronization primitives to protect shared data\.

```cpp
// Safe Option 1: Mutex
int count = 0;
std::mutex mtx;
std::thread t1([&]() {
    for (int i=0; i<1000; ++i) {
        std::lock_guard<std::mutex> lock(mtx); // Lock during modification
        count++;
    }
});
// Option 2: Atomic variable (simpler for basic types)
std::atomic<int> count(0);
std::thread t1([&]() { for (int i=0; i<1000; ++i) count++; });
```

**Note:** Atomic variables are more efficient than mutexes for single\-variable operations\.

> （注：文档部分内容可能由 AI 生成）
