Here's a comprehensive list of 50 tricky STL vector interview questions with detailed answers and examples:

## 1-10: Memory and Capacity Management

### 1. What happens to iterators when vector resizes?
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
auto it = vec.begin() + 2;  // Points to 3

vec.push_back(6);  // Might cause reallocation
// it is now INVALIDATED - undefined behavior to use it

// Solution: Use indices or recalculate positions
size_t index = 2;
vec.push_back(6);
auto new_it = vec.begin() + index;  // Safe
```

### 2. Difference between `reserve()` and `resize()`
```cpp
std::vector<int> vec;
vec.reserve(100);  
// capacity() >= 100, size() == 0
// Memory allocated but no elements constructed

vec.resize(50);    
// capacity() >= 100, size() == 50
// 50 elements default-constructed (value 0)

vec.resize(150);   
// capacity() >= 150, size() == 150
// New elements default-constructed, might reallocate

// Key: reserve() changes capacity, resize() changes size
```

### 3. Shrink-to-fit technique
```cpp
std::vector<int> vec(1000);
vec.erase(vec.begin() + 500, vec.end());  // size() = 500, capacity() = 1000

// C++11: shrink_to_fit (non-binding request)
vec.shrink_to_fit();  
// capacity() might still be > 500

// Guaranteed shrink:
std::vector<int>(vec).swap(vec);  // Creates temporary copy and swaps
// Now capacity() == size()
```

### 4. Why `vector<bool>` is problematic
```cpp
std::vector<bool> vb = {true, false, true};

// vector<bool> is NOT a container of bools!
// It's a specialized bitset
auto bit = vb[0];  
// bit is NOT bool&, it's std::vector<bool>::reference (proxy class)

// Problems:
// 1. Cannot take address: &vb[0] - ERROR
// 2. Type deduction issues with auto
auto proxy = vb[0];
proxy = false;  // Modifies vb[0], but proxy isn't bool

// Solutions:
std::vector<char> flags;           // Use char instead
std::deque<bool> db;               // deque<bool> works normally
boost::container::vector<bool> bb; // Boost alternative
```

### 5. Move semantics with vector
```cpp
std::vector<std::string> createVector() {
    std::vector<std::string> vec;
    vec.push_back("Hello");
    vec.push_back("World");
    return vec;  // NRVO or move, efficient!
}

std::vector<std::string> v1 = {"test"};
std::vector<std::string> v2 = std::move(v1);
// v1 is now in valid but unspecified state
// v1.size() == 0 (typically)
// v2 owns the original data

// Moving individual elements:
std::vector<std::unique_ptr<int>> ptr_vec;
ptr_vec.push_back(std::make_unique<int>(42));
auto ptr = std::move(ptr_vec[0]);  // Transfers ownership
// ptr_vec[0] is now nullptr
```

## 11-20: Iterator Invalidation

### 11. Classical iterator invalidation during erase
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// WRONG: Iterator invalidation
for (auto it = vec.begin(); it != vec.end(); ++it) {
    if (*it % 2 == 0) {
        vec.erase(it);  // it is invalidated!
    }
}

// CORRECT: Use return value of erase
for (auto it = vec.begin(); it != vec.end(); /* no increment */) {
    if (*it % 2 == 0) {
        it = vec.erase(it);  // Returns iterator to next element
    } else {
        ++it;
    }
}

// C++20: std::erase_if
std::erase_if(vec, [](int i) { return i % 2 == 0; });
```

### 12. Insert invalidation and reallocation
```cpp
std::vector<int> vec = {1, 2, 3};
vec.reserve(3);  // capacity() == 3

auto it = vec.begin();
vec.push_back(4);  // Reallocation occurs!

// it is now invalid
// std::cout << *it;  // UNDEFINED BEHAVIOR

// Check if reallocation will happen:
if (vec.size() == vec.capacity()) {
    // Store indices, not iterators
    size_t idx = 1;
    vec.reserve(vec.capacity() * 2);
    it = vec.begin() + idx;  // Recalculate iterator
}
```

### 13. Range-based for loop invalidation
```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// WRONG: Modifying vector during iteration
for (int& val : vec) {
    if (val == 3) {
        vec.push_back(6);  // Potential reallocation!
    }
}

// Also wrong:
for (auto it = vec.begin(); it != vec.end(); ++it) {
    if (*it == 3) {
        vec.erase(it);  // Invalidates current iterator
    }
}

// Use indices or create copy:
auto copy = vec;  // Work on copy
for (size_t i = 0; i < copy.size(); ++i) {
    if (copy[i] == 3) {
        vec.erase(vec.begin() + i);
    }
}
```

### 14. Insert at beginning performance
```cpp
std::vector<int> vec(1000000);

// O(n) - all elements shifted
vec.insert(vec.begin(), 42);

// Alternative: use deque for frequent front insertions
std::deque<int> dq(1000000);
dq.push_front(42);  // O(1) amortized

// Or maintain vector in reverse
std::vector<int> vec_rev;
vec_rev.push_back(42);  // "Front" is actually last element
```

## 21-30: Advanced Operations

### 21. Vector of vectors - memory layout
```cpp
// NOT contiguous in memory
std::vector<std::vector<int>> matrix(3, std::vector<int>(4));
// Each row is separately allocated

// For contiguous 2D array:
std::vector<int> flat(3 * 4);  // Single allocation

// Access with index calculation
auto get = [&](int row, int col) -> int& {
    return flat[row * 4 + col];
};

// Or use a wrapper class
class Matrix2D {
    std::vector<int> data;
    size_t cols;
public:
    Matrix2D(size_t rows, size_t c) : data(rows * c), cols(c) {}
    int& operator()(size_t r, size_t c) { return data[r * cols + c]; }
};
```

### 22. Emplace_back vs push_back
```cpp
struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {
        std::cout << "Constructed\n";
    }
    Point(const Point& other) : x(other.x), y(other.y) {
        std::cout << "Copied\n";
    }
    Point(Point&& other) noexcept : x(other.x), y(other.y) {
        std::cout << "Moved\n";
    }
};

std::vector<Point> vec;
vec.reserve(10);

// push_back with temporary: construct + move
vec.push_back(Point(1, 2));  
// Output: Constructed, Moved

// emplace_back: construct in-place
vec.emplace_back(3, 4);      
// Output: Constructed (only one call!)

// push_back with existing object: copy
Point p(5, 6);
vec.push_back(p);            
// Output: Constructed, Copied
```

### 23. Exception safety with vector
```cpp
class ThrowingCopy {
    static int counter;
public:
    ThrowingCopy() = default;
    ThrowingCopy(const ThrowingCopy&) {
        if (++counter == 3) throw std::runtime_error("Copy failed");
    }
};
int ThrowingCopy::counter = 0;

// Vector provides basic exception guarantee
std::vector<ThrowingCopy> vec(5);
try {
    vec.push_back(ThrowingCopy{});
    // If copy throws during reallocation:
    // - Original vector remains unchanged
    // - Memory is properly freed
    // - No resource leak
} catch (const std::exception& e) {
    // vec is in original state
}
```

### 24. Custom allocator with vector
```cpp
template<typename T>
class StackAllocator {
    std::array<T, 1000> arena;
    size_t used = 0;
public:
    using value_type = T;
    
    T* allocate(size_t n) {
        if (used + n > 1000) throw std::bad_alloc();
        T* ptr = &arena[used];
        used += n;
        return ptr;
    }
    
    void deallocate(T* p, size_t n) {
        // No need to free stack memory
        if (p + n == &arena[used]) {
            used -= n;  // Can reuse if last allocation
        }
    }
    
    template<typename U>
    bool operator==(const StackAllocator<U>&) const { return false; }
};

// Usage: vector with stack allocation
StackAllocator<int> alloc;
std::vector<int, StackAllocator<int>> vec(alloc);
vec.push_back(42);  // Stored on stack, not heap!
```

## 31-40: Performance and Optimization

### 31. Small buffer optimization for vector
```cpp
template<typename T, size_t BufferSize = 64>
class SmallVector {
    union {
        T buffer[BufferSize];
        T* heap_data;
    };
    size_t size_;
    size_t capacity_;
    bool on_heap;
    
public:
    SmallVector() : size_(0), capacity_(BufferSize), on_heap(false) {}
    
    void push_back(const T& value) {
        if (size_ == capacity_) {
            grow();
        }
        if (on_heap) {
            heap_data[size_] = value;
        } else {
            buffer[size_] = value;
        }
        ++size_;
    }
    
private:
    void grow() {
        size_t new_cap = capacity_ * 2;
        T* new_data = new T[new_cap];
        
        // Move existing elements
        T* old_data = on_heap ? heap_data : buffer;
        std::move(old_data, old_data + size_, new_data);
        
        if (on_heap) delete[] heap_data;
        on_heap = true;
        heap_data = new_data;
        capacity_ = new_cap;
    }
};
```

### 32. Sorting vector of custom objects
```cpp
struct Employee {
    std::string name;
    int age;
    double salary;
};

std::vector<Employee> employees = {
    {"Alice", 30, 75000},
    {"Bob", 25, 65000},
    {"Charlie", 35, 85000}
};

// Method 1: Custom comparator
std::sort(employees.begin(), employees.end(),
    [](const Employee& a, const Employee& b) {
        return a.salary < b.salary;  // Sort by salary
    });

// Method 2: Multiple criteria
std::sort(employees.begin(), employees.end(),
    [](const Employee& a, const Employee& b) {
        if (a.age != b.age) return a.age < b.age;
        return a.name < b.name;  // Secondary sort
    });

// Method 3: Partial sort (top N)
std::partial_sort(employees.begin(), 
                  employees.begin() + 2,  // Top 2
                  employees.end(),
                  [](const auto& a, const auto& b) {
                      return a.salary > b.salary;
                  });
```

### 33. Efficient vector comparison
```cpp
std::vector<int> vec1 = {1, 2, 3, 4, 5};
std::vector<int> vec2 = {1, 2, 3, 4, 6};

// Lexicographical comparison (default)
bool less = vec1 < vec2;  // true, 5 < 6

// Element-wise comparison with tolerance
auto equal_with_tolerance = [](const auto& a, const auto& b, double tol = 0.001) {
    return std::equal(a.begin(), a.end(), b.begin(), b.end(),
        [tol](double x, double y) {
            return std::abs(x - y) < tol;
        });
};

// Mismatch: find first difference
auto [it1, it2] = std::mismatch(vec1.begin(), vec1.end(), vec2.begin());
if (it1 != vec1.end()) {
    std::cout << "First difference at position: " 
              << std::distance(vec1.begin(), it1) << '\n';
}
```

### 34. Removing duplicates efficiently
```cpp
std::vector<int> vec = {1, 2, 2, 3, 3, 3, 4, 5, 5};

// Method 1: Requires sorted range
std::sort(vec.begin(), vec.end());
auto last = std::unique(vec.begin(), vec.end());
vec.erase(last, vec.end());  // vec: {1, 2, 3, 4, 5}

// Method 2: Preserve order, O(n) space
std::unordered_set<int> seen;
vec.erase(std::remove_if(vec.begin(), vec.end(),
    [&seen](int x) {
        return !seen.insert(x).second;
    }), vec.end());

// Method 3: Preserve order, less space
auto it = vec.begin();
std::unordered_set<int> seen2;
for (auto src = vec.begin(); src != vec.end(); ++src) {
    if (seen2.insert(*src).second) {
        *it++ = *src;
    }
}
vec.erase(it, vec.end());
```

## 41-50: Tricky Scenarios and Pitfalls

### 41. Vector of references (not allowed)
```cpp
int a = 1, b = 2, c = 3;

// ERROR: Can't have vector of references
// std::vector<int&> ref_vec = {a, b, c};

// Alternative 1: std::reference_wrapper
std::vector<std::reference_wrapper<int>> ref_vec = {a, b, c};
ref_vec[0].get() = 10;  // Modifies 'a'
++ref_vec[1];            // Also works

// Alternative 2: Vector of pointers
std::vector<int*> ptr_vec = {&a, &b, &c};
*ptr_vec[0] = 10;

// Alternative 3: std::ref with vector
std::vector<std::reference_wrapper<int>> v;
v.push_back(std::ref(a));  // Wraps reference
```

### 42. Vector initialization tricks
```cpp
// Common confusion: uniform initialization
std::vector<int> v1{5, 10};      // {5, 10} - two elements
std::vector<int> v2(5, 10);      // {10, 10, 10, 10, 10} - 5 elements

// Constructor resolution with initializer_list
std::vector<std::string> v3{5, "hello"};  // {5, "hello"} - ERROR!
// Ambiguous: initializer_list<string> or size_type + string?

// Copy vs direct initialization
auto v4 = std::vector<int>{1, 2, 3};  // Copy initialization
std::vector<int> v5{1, 2, 3};         // Direct initialization

// Nested vectors
std::vector<std::vector<int>> matrix(3, std::vector<int>(4, 0));
// 3 rows, 4 columns, all zeros
```

### 43. Data() vs &front() subtlety
```cpp
std::vector<int> vec;
vec.reserve(10);

// data() is safe even for empty vector
int* ptr1 = vec.data();  // Might be nullptr or internal buffer

// &front() is UB for empty vector
// int* ptr2 = &vec.front();  // UNDEFINED BEHAVIOR!

// After push_back, both work
vec.push_back(42);
assert(vec.data() == &vec.front());  // Always true
assert(vec.data() == &vec[0]);       // Always true

// Contiguous guarantee (C++11)
// vector data is guaranteed to be contiguous
std::memcpy(arr, vec.data(), vec.size() * sizeof(int));  // Safe
```

### 44. Erase-remove idiom performance
```cpp
std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};

// Remove all 2s efficiently
// Step 1: Compact elements
auto new_end = std::remove(vec.begin(), vec.end(), 2);
// vec now: {1, 3, 4, 5, ?, ?, ?}

// Step 2: Erase the "removed" elements
vec.erase(new_end, vec.end());
// vec: {1, 3, 4, 5}

// With predicate (more complex)
std::vector<std::string> names = {"Alice", "Bob", "Charlie", "David"};
names.erase(
    std::remove_if(names.begin(), names.end(),
        [](const std::string& s) { return s.length() <= 4; }),
    names.end()
);

// C++20: single function call
std::erase(vec, 2);      // Erase all 2s
std::erase_if(names, [](const auto& s) { return s.length() <= 4; });
```

### 45. Thread safety considerations
```cpp
#include <mutex>
#include <shared_mutex>

class ThreadSafeVector {
    mutable std::shared_mutex mtx;
    std::vector<int> vec;
public:
    // Multiple readers allowed
    size_t safe_size() const {
        std::shared_lock lock(mtx);
        return vec.size();
    }
    
    int safe_at(size_t idx) const {
        std::shared_lock lock(mtx);
        return vec.at(idx);  // Throws if out of range
    }
    
    // Single writer
    void safe_push_back(int val) {
        std::unique_lock lock(mtx);
        vec.push_back(val);
    }
    
    // Atomic snapshot
    std::vector<int> safe_copy() const {
        std::shared_lock lock(mtx);
        return vec;  // Returns copy under lock
    }
};

// Common pitfall: iterator invalidation in multi-threaded context
ThreadSafeVector tsv;
// Thread A:
auto size = tsv.safe_size();  // Returns 5
// Thread B modifies vector...
auto val = tsv.safe_at(size - 1);  // Might be out of range!
```

### 46. Vector with aligned storage (SIMD)
```cpp
#include <malloc.h>

template<typename T, size_t Alignment = 64>
struct AlignedAllocator {
    using value_type = T;
    
    T* allocate(size_t n) {
        void* ptr = _aligned_malloc(n * sizeof(T), Alignment);
        if (!ptr) throw std::bad_alloc();
        return static_cast<T*>(ptr);
    }
    
    void deallocate(T* ptr, size_t) {
        _aligned_free(ptr);
    }
};

// For SIMD operations
using AlignedDoubleVec = std::vector<double, AlignedAllocator<double, 32>>;
AlignedDoubleVec vec;
vec.push_back(1.0);

// Verify alignment
assert(reinterpret_cast<uintptr_t>(vec.data()) % 32 == 0);

// Now safe for AVX operations
__m256d data = _mm256_load_pd(vec.data());
```

### 47. Vector of const objects
```cpp
// Vector of const objects
struct Immutable {
    const int value;
    Immutable(int v) : value(v) {}
};

std::vector<Immutable> vec;
vec.emplace_back(1);  // OK: constructs in-place
vec.emplace_back(2);  // OK

// ERROR: can't assign to const member
// vec[0].value = 10;

// ERROR: can't sort - requires assignment
// std::sort(vec.begin(), vec.end());

// Reallocation requires moving, but const can't be moved
// vec.reserve(10);  // Might cause issues if reallocation needed

// Alternative: use const vector
const std::vector<int> const_vec = {1, 2, 3};
// const_vec[0] = 10;  // ERROR: read-only
```

### 48. Vector slicing and subvector operations
```cpp
std::vector<int> original = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// Method 1: Copy constructor with iterators
std::vector<int> slice1(original.begin() + 2, original.begin() + 7);
// {3, 4, 5, 6, 7}

// Method 2: Assign from range
std::vector<int> slice2;
slice2.assign(original.begin() + 2, original.begin() + 7);

// Method 3: Using span (C++20)
#include <span>
std::span<int> view(original.data() + 2, 5);  // Non-owning view
view[0] = 42;  // Modifies original[2]

// Method 4: Efficient swap of sub-ranges
std::vector<int> v1 = {1, 2, 3, 4, 5};
std::vector<int> v2 = {6, 7, 8, 9, 10};
std::swap_ranges(v1.begin(), v1.begin() + 3, v2.begin());
// v1: {6, 7, 8, 4, 5}
// v2: {1, 2, 3, 9, 10}
```

### 49. Exception safety during vector resize
```cpp
class Resource {
    static int active;
    int id;
public:
    Resource() : id(++active) {
        if (id == 7) throw std::runtime_error("Construction failed");
        std::cout << "Created: " << id << '\n';
    }
    ~Resource() { std::cout << "Destroyed: " << id << '\n'; active--; }
    Resource(const Resource& r) : id(++active) {
        if (id == 7) throw std::runtime_error("Copy failed");
        std::cout << "Copied: " << id << '\n';
    }
    Resource(Resource&& r) noexcept : id(r.id) {
        std::cout << "Moved: " << id << '\n';
        r.id = -1;
    }
};
int Resource::active = 0;

void demonstrateExceptionSafety() {
    std::vector<Resource> vec;
    vec.reserve(3);
    vec.emplace_back();  // id=1
    vec.emplace_back();  // id=2
    vec.emplace_back();  // id=3
    
    try {
        // This will trigger reallocation and potentially throw
        vec.resize(10);  
        // If copy #7 throws:
        // - All previously copied elements destroyed
        // - Original 3 elements kept alive
        // - Memory freed
        // - vec unchanged (strong guarantee)
    } catch (const std::exception& e) {
        std::cout << "Caught: " << e.what() << '\n';
        // vec still has original 3 elements
    }
}
```

### 50. Performance comparison: vector vs other containers
```cpp
#include <chrono>
#include <list>
#include <deque>

// Benchmark: Random insert in the middle
template<typename Container>
void benchmark_middle_insert() {
    Container c(100000, 42);  // Fill with elements
    auto it = c.begin();
    std::advance(it, 50000);  // Move to middle
    
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < 1000; ++i) {
        it = c.insert(it, i);  // Insert before middle
    }
    auto end = std::chrono::high_resolution_clock::now();
    
    std::cout << typeid(Container).name() << ": " 
              << std::chrono::duration_cast<std::chrono::microseconds>(
                     end - start).count() << "us\n";
}

// Results (typical):
// vector: 15000us  - Shifts half the elements each insert
// deque:  5000us   - Better, but still shifts
// list:   100us    - O(1) insertion, no shifting

// But for sequential access:
// vector: cache-friendly, fastest iteration
// list: scattered memory, slow iteration

// Decision matrix:
// - Frequent random insert/delete: list or deque
// - Frequent push_back, random access: vector
// - Frequent push_front/push_back: deque
// - Minimal overhead, cache locality: vector
```

**Key Interview Takeaways:**

1. **Iterator invalidation** - Know when and why iterators become invalid
2. **Memory reallocation** - Understand capacity vs size
3. **Move semantics** - Use `emplace_back` over `push_back` for complex types
4. **`vector<bool>` pitfalls** - It's a bitset, not a container
5. **Exception safety** - Vector provides strong exception guarantee for insert operations
6. **Performance characteristics** - O(1) random access, amortized O(1) push_back
7. **Contiguous storage** - Guaranteed by C++11 standard
8. **Custom allocators** - Can optimize for specific use cases

These questions test deep understanding of vector internals, memory management, and performance trade-offs that separate intermediate from advanced C++ developers.