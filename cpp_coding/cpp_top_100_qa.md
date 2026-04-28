Below is a curated list of **top 100 C++ coding interview questions**, organized by topic, with concise answers and working examples. This covers **basics, OOP, STL, memory, concurrency, algorithms, and modern C++ (11/14/17/20)**—ideal for senior/algorithmic trading engineer interviews.

---

## Part 1: C++ Basics (1–20)
### 1. What is the difference between `#include <...>` and `#include "..."`?
- `<>`: Searches standard library paths.
- `""`: Searches current directory first, then standard paths.

### 2. What is `main()`?
Entry point of a C++ program. Signature:
```cpp
int main() { return 0; }
// or
int main(int argc, char* argv[]) { ... }
```

### 3. What are `namespace` and `using namespace std`?
- `namespace`: Avoid name collisions.
- `using namespace std`: Imports all `std` identifiers (avoid in headers).

### 4. Data types: `int`, `char`, `float`, `double`, `bool`, `void`
- `bool`: `true`/`false` (1 byte).
- `void`: No type (used for functions returning nothing).

### 5. What is a variable? Scope rules?
- Variable: Named memory location.
- Scope: Local (block), global, static, namespace.

### 6. What is `const`?
- Makes a value read-only.
```cpp
const int x = 5;
void func(const int& a) { ... } // read-only reference
```

### 7. `const` vs `constexpr`
- `const`: Runtime constant.
- `constexpr`: Compile-time constant (C++11+).
```cpp
constexpr int square(int x) { return x*x; }
int arr[square(5)]; // OK (compile-time size)
```

### 8. What is `volatile`?
- Tells compiler not to optimize access (e.g., hardware registers).
```cpp
volatile int sensor;
```

### 9. What is `sizeof`?
- Returns size in bytes of a type/variable.
```cpp
cout << sizeof(int) << endl; // typically 4
```

### 10. What is a pointer?
- Variable storing memory address.
```cpp
int x = 10;
int* p = &x;
cout << *p << endl; // 10
```

### 11. What is a reference? Difference from pointer?
- Alias for a variable; must be initialized; cannot be null.
```cpp
int x = 10;
int& r = x;
r = 20; // x becomes 20
```

### 12. What is `nullptr`?
- C++11 null pointer literal (replaces `NULL`).
```cpp
int* p = nullptr;
```

### 13. What is `auto`?
- Type deduction (C++11+).
```cpp
auto x = 5; // int
auto& r = x; // int&
```

### 14. What is `decltype`?
- Yields type of an expression (C++11+).
```cpp
int x;
decltype(x) y = 10; // int
```

### 15. What is a function? Default arguments?
```cpp
int add(int a, int b = 0) { return a + b; }
add(5); // 5
```

### 16. What is function overloading?
- Same name, different parameters.
```cpp
void print(int);
void print(double);
```

### 17. What is `inline`?
- Suggests compiler replace function call with body (reduce overhead).
```cpp
inline int square(int x) { return x*x; }
```

### 18. What is recursion?
- Function calling itself. Example: factorial.
```cpp
int fact(int n) {
    if (n <= 1) return 1;
    return n * fact(n-1);
}
```

### 19. What is a macro? `#define`?
- Preprocessor text replacement.
```cpp
#define MAX(a,b) ((a)>(b)?(a):(b))
```

### 20. What is `static` (for variables/functions)?
- **Static variable**: Retains value between calls.
- **Static function**: Visible only in current file.
```cpp
void func() {
    static int count = 0;
    count++;
}
```

---

## Part 2: Object-Oriented Programming (21–40)
### 21. What is a class? `class` vs `struct`?
- `class`: Default access `private`.
- `struct`: Default access `public`.
```cpp
struct Point { int x, y; }; // public by default
class Circle { private: int r; public: ... };
```

### 22. What are access specifiers: `public`, `private`, `protected`?
- `public`: Accessible everywhere.
- `private`: Only within class/friends.
- `protected`: Within class, friends, derived classes.

### 23. What is a constructor? Default/copy/move?
- **Default**: No arguments.
- **Copy**: `Class(const Class&)`.
- **Move**: `Class(Class&&)` (C++11+).
```cpp
class A {
public:
    A() {} // default
    A(const A&) {} // copy
    A(A&&) {} // move
};
```

### 24. What is a destructor? `virtual` destructor?
- Cleans up resources.
- **Virtual destructor**: Ensures derived destructor runs when deleting via base pointer.
```cpp
class Base {
public:
    virtual ~Base() {}
};
class Derived : public Base { ... };
Base* b = new Derived;
delete b; // OK (Derived destructor called)
```

### 25. What is `this` pointer?
- Points to current object.
```cpp
class A {
    int x;
public:
    void set(int x) { this->x = x; }
};
```

### 26. What is inheritance? Types?
- **Public**: Public/protected members stay.
- **Protected**: Public→protected.
- **Private**: Public/protected→private.
```cpp
class Derived : public Base { ... };
```

### 27. What is polymorphism?
- One interface, multiple implementations.
- **Compile-time**: Overloading, templates.
- **Runtime**: Virtual functions.

### 28. What is a virtual function? `vtable`?
- Enables runtime polymorphism.
- Compiler creates a **vtable** (array of function pointers) per class; object has a **vptr** to it.
```cpp
class Base {
public:
    virtual void show() { cout << "Base\n"; }
};
class Derived : public Base {
public:
    void show() override { cout << "Derived\n"; }
};
```

### 29. What is a pure virtual function? Abstract class?
- Pure virtual: `virtual void f() = 0;`
- Abstract class: Cannot be instantiated; used as interface.
```cpp
class Shape {
public:
    virtual double area() = 0; // pure virtual
};
```

### 30. What is `override`? `final`? (C++11+)
- `override`: Explicitly marks overriding (compiler checks).
- `final`: Prevents further overriding/inheritance.
```cpp
class Derived : public Base {
    void show() override final { ... }
};
```

### 31. What is a friend function/class?
- Grants access to private/protected members.
```cpp
class A {
    int x;
    friend void func(A&);
};
void func(A& a) { a.x = 5; } // can access private x
```

### 32. What is operator overloading?
- Redefine operators for user-defined types.
```cpp
class Point {
    int x, y;
public:
    Point operator+(const Point& p) {
        return {x+p.x, y+p.y};
    }
};
```

### 33. What is copy elision?
- Compiler optimizes away unnecessary copies (e.g., return value optimization).

### 34. What is object slicing?
- Assigning derived → base by value loses derived parts.
```cpp
Derived d;
Base b = d; // slices Derived parts
```

### 35. What is multiple inheritance? Diamond problem?
- A class inherits from two+ bases.
- **Diamond**: Ambiguity when two bases share a common ancestor.
- Solve with **virtual inheritance**:
```cpp
class A {};
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};
```

### 36. What is a static member (variable/function)?
- Belongs to class, not object; shared across all instances.
```cpp
class A {
    static int count;
public:
    static void inc() { count++; }
};
int A::count = 0;
```

### 37. What is a singleton? Implement one.
- Only one instance.
```cpp
class Singleton {
    Singleton() {}
public:
    static Singleton& get() {
        static Singleton instance;
        return instance;
    }
};
```

### 38. What is RAII?
- **Resource Acquisition Is Initialization**: Bind resource lifetime to object lifetime (e.g., `std::lock_guard`, smart pointers).

### 39. What is a copy-and-swap idiom?
- Safe copy assignment:
```cpp
class A {
    int* data;
public:
    A& operator=(A other) {
        swap(*this, other);
        return *this;
    }
    friend void swap(A& a, A& b) { ... }
};
```

### 40. What is `explicit`?
- Prevents implicit conversions for constructors.
```cpp
class A {
public:
    explicit A(int x) { ... }
};
A a = 5; // error (no implicit conversion)
```

---

## Part 3: Memory Management (41–55)
### 41. `new`/`delete` vs `malloc`/`free`?
- `new`/`delete`: C++ operators; call constructor/destructor; type-safe.
- `malloc`/`free`: C functions; raw memory; no construction.
```cpp
int* p = new int(5);
delete p;
int* q = (int*)malloc(sizeof(int));
free(q);
```

### 42. What is a memory leak? How to avoid?
- Allocated memory not freed.
- Avoid with **smart pointers** (`unique_ptr`, `shared_ptr`).

### 43. What is `std::unique_ptr`?
- Exclusive ownership; cannot copy; move-only.
```cpp
#include <memory>
unique_ptr<int> p = make_unique<int>(5);
```

### 44. What is `std::shared_ptr`?
- Shared ownership; reference count.
```cpp
shared_ptr<int> p1 = make_shared<int>(5);
shared_ptr<int> p2 = p1; // ref count = 2
```

### 45. What is `std::weak_ptr`?
- Non-owning reference to `shared_ptr`; breaks circular references.
```cpp
weak_ptr<int> wp = p1;
if (auto sp = wp.lock()) { ... }
```

### 46. What is `make_unique`/`make_shared`? (C++14/11)
- Preferred over `new`; safer, more efficient.
```cpp
auto p = make_unique<vector<int>>();
```

### 47. What is a dangling pointer?
- Pointer to freed/invalid memory.
- Avoid with smart pointers or nulling after delete.

### 48. What is a wild pointer?
- Uninitialized pointer.
- Always initialize: `int* p = nullptr;`

### 49. What is stack vs heap?
- **Stack**: Automatic variables; fast; limited size; LIFO.
- **Heap**: Dynamic allocation (`new`); slower; large; manual management.

### 50. What is memory alignment?
- Data stored at addresses multiples of its size (e.g., `int` at 4-byte boundaries).
- Use `alignas` (C++11):
```cpp
alignas(16) int x;
```

### 51. What is a memory pool?
- Preallocate a block of memory for fast allocation/deallocation (reduces fragmentation).

### 52. What is `placement new`?
- Construct object at a pre-allocated address.
```cpp
char buf[sizeof(int)];
int* p = new(buf) int(5);
p->~int(); // explicit destructor call
```

### 53. What is a memory leak detector?
- Tools: Valgrind, AddressSanitizer (ASAN).

### 54. What is `noexcept`? (C++11)
- Specifies function does not throw; enables optimizations.
```cpp
void func() noexcept { ... }
```

### 55. What is exception handling? `try`/`catch`/`throw`?
```cpp
try {
    throw runtime_error("error");
} catch (const exception& e) {
    cout << e.what() << endl;
}
```

---

## Part 4: STL (Standard Template Library) (56–70)
### 56. What is STL? Components?
- Containers, iterators, algorithms, functors, adaptors.

### 57. Containers: `vector`, `list`, `deque`, `array`
- `vector`: Dynamic array; random access; O(1) access, O(n) insert middle.
- `list`: Doubly linked list; O(1) insert/erase; no random access.
- `deque`: Double-ended queue; random access; O(1) push/pop front/back.
- `array`: Fixed-size array (C++11).

### 58. Associative containers: `map`, `set`, `unordered_map`, `unordered_set`
- `map`/`set`: Ordered (red-black tree); O(log n).
- `unordered_map`/`unordered_set`: Hash table; average O(1).

### 59. What is an iterator? Categories?
- Pointer-like object to traverse containers.
- Categories: input, output, forward, bidirectional, random access.

### 60. `begin()`/`end()` vs `cbegin()`/`cend()`?
- `cbegin()`/`cend()`: Const iterators (read-only).

### 61. What is `std::sort`? Time complexity?
- Sorts range; O(n log n).
```cpp
vector<int> v = {3,1,2};
sort(v.begin(), v.end());
```

### 62. What is `std::find`?
- Linear search; returns iterator to first match.
```cpp
auto it = find(v.begin(), v.end(), 2);
```

### 63. What is `std::binary_search`?
- Requires sorted range; O(log n).
```cpp
bool found = binary_search(v.begin(), v.end(), 2);
```

### 64. What is `std::reverse`?
```cpp
reverse(v.begin(), v.end());
```

### 65. What is `std::accumulate`?
- Sum range (in `<numeric>`).
```cpp
int sum = accumulate(v.begin(), v.end(), 0);
```

### 66. What is `std::for_each`?
```cpp
for_each(v.begin(), v.end(), [](int x) { cout << x; });
```

### 67. What is a functor?
- Function object (overloads `operator()`).
```cpp
struct Add {
    int operator()(int a, int b) { return a+b; }
};
```

### 68. What is a lambda? (C++11)
- Anonymous function.
```cpp
auto sum = [](int a, int b) { return a+b; };
```

### 69. What is `std::function`?
- Wraps any callable (function, lambda, functor).
```cpp
function<int(int,int)> f = [](int a, int b) { return a+b; };
```

### 70. What is `std::bind`?
- Bind arguments to a callable.
```cpp
auto add5 = bind(plus<int>(), placeholders::_1, 5);
add5(3); // 8
```

---

## Part 5: Modern C++ (C++11/14/17/20) (71–80)
### 71. What is move semantics? `std::move`?
- Transfer ownership of resources instead of copying.
- `std::move`: Cast to rvalue reference.
```cpp
string a = "hello";
string b = move(a); // a is now empty
```

### 72. What is an rvalue reference? `&&`?
- Binds to temporary objects; enables moving.
```cpp
void func(string&& s) { ... }
func("temp");
```

### 73. What is perfect forwarding? `std::forward`?
- Preserve value category (lvalue/rvalue) in templates.
```cpp
template <typename T>
void wrapper(T&& arg) {
    func(forward<T>(arg));
}
```

### 74. What is `decltype(auto)`? (C++14)
- Deduces exact type (including references).
```cpp
decltype(auto) f() { int x; return (x); } // int&
```

### 75. What is `constexpr` functions/constructors?
- Evaluate at compile time.
```cpp
constexpr int square(int x) { return x*x; }
constexpr int s = square(5); // 25 at compile time
```

### 76. What is `std::variant`? (C++17)
- Type-safe union.
```cpp
variant<int, string> v = 5;
v = "hello";
```

### 77. What is `std::optional`? (C++17)
- May or may not hold a value.
```cpp
optional<int> o;
o = 5;
if (o) cout << *o;
```

### 78. What is `std::any`? (C++17)
- Holds any type.
```cpp
any a = 5;
a = string("hello");
```

### 79. What is structured bindings? (C++17)
```cpp
pair<int, string> p = {1, "a"};
auto [i, s] = p;
```

### 80. What is `concepts`? (C++20)
- Constrains template arguments.
```cpp
template <Integral T>
T add(T a, T b) { return a+b; }
```

---

## Part 6: Concurrency & Multithreading (81–90)
### 81. What is `std::thread`? (C++11)
```cpp
#include <thread>
void func() { ... }
thread t(func);
t.join();
```

### 82. What is `std::mutex`? `lock_guard`?
- `mutex`: Mutual exclusion.
- `lock_guard`: RAII lock.
```cpp
mutex m;
void func() {
    lock_guard<mutex> lock(m);
    // critical section
}
```

### 83. What is `std::unique_lock`?
- Flexible lock (can lock/unlock manually; use with condition variables).
```cpp
unique_lock<mutex> lock(m);
lock.unlock();
```

### 84. What is `std::condition_variable`?
- Wait for a condition.
```cpp
condition_variable cv;
unique_lock<mutex> lock(m);
cv.wait(lock, [] { return ready; });
```

### 85. What is a race condition? How to prevent?
- Concurrent unsynchronized access to shared data.
- Prevent with mutexes, atomics, or immutable data.

### 86. What is `std::atomic`?
- Atomic operations (no mutex needed for simple types).
```cpp
atomic<int> count = 0;
count++; // atomic
```

### 87. What is a deadlock? How to avoid?
- Circular wait for locks.
- Avoid: lock ordering, `try_lock`, reduce lock scope.

### 88. What is `std::async`? `std::future`?
- Launch async task; get result via `future`.
```cpp
future<int> f = async(launch::async, [] { return 5; });
int res = f.get();
```

### 89. What is `std::promise`?
- Set value for a `future`.
```cpp
promise<int> p;
future<int> f = p.get_future();
p.set_value(5);
```

### 90. Implement a thread-safe queue.
```cpp
template <typename T>
class ThreadSafeQueue {
    queue<T> q;
    mutable mutex mtx;
    condition_variable cv;
public:
    void push(T val) {
        lock_guard<mutex> lock(mtx);
        q.push(move(val));
        cv.notify_one();
    }
    bool pop(T& val) {
        unique_lock<mutex> lock(mtx);
        cv.wait(lock, [this] { return !q.empty(); });
        val = move(q.front());
        q.pop();
        return true;
    }
};
```

---

## Part 7: Algorithms & Coding Problems (91–100)
### 91. Reverse a string.
```cpp
string reverse(string s) {
    int l = 0, r = s.size()-1;
    while (l < r) swap(s[l++], s[r--]);
    return s;
}
```

### 92. Check palindrome.
```cpp
bool isPalindrome(string s) {
    int l = 0, r = s.size()-1;
    while (l < r) if (s[l++] != s[r--]) return false;
    return true;
}
```

### 93. Two-sum (find indices).
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> m;
    for (int i = 0; i < nums.size(); i++) {
        if (m.count(target - nums[i]))
            return {m[target-nums[i]], i};
        m[nums[i]] = i;
    }
    return {};
}
```

### 94. Maximum subarray (Kadane’s algorithm).
```cpp
int maxSubArray(vector<int>& nums) {
    int cur = 0, res = nums[0];
    for (int x : nums) {
        cur = max(x, cur + x);
        res = max(res, cur);
    }
    return res;
}
```

### 95. Merge two sorted arrays.
```cpp
vector<int> merge(vector<int>& a, vector<int>& b) {
    vector<int> res;
    int i = 0, j = 0;
    while (i < a.size() && j < b.size())
        res.push_back(a[i] < b[j] ? a[i++] : b[j++]);
    while (i < a.size()) res.push_back(a[i++]);
    while (j < b.size()) res.push_back(b[j++]);
    return res;
}
```

### 96. Implement LRU cache.
```cpp
class LRUCache {
    list<pair<int, int>> cache;
    unordered_map<int, list<pair<int, int>>::iterator> map;
    int cap;
public:
    LRUCache(int capacity) : cap(capacity) {}
    int get(int key) {
        if (!map.count(key)) return -1;
        cache.splice(cache.begin(), cache, map[key]);
        return map[key]->second;
    }
    void put(int key, int val) {
        if (map.count(key)) {
            cache.splice(cache.begin(), cache, map[key]);
            map[key]->second = val;
            return;
        }
        if (cache.size() == cap) {
            map.erase(cache.back().first);
            cache.pop_back();
        }
        cache.emplace_front(key, val);
        map[key] = cache.begin();
    }
};
```

### 97. Find median of two sorted arrays.
```cpp
double findMedianSortedArrays(vector<int>& a, vector<int>& b) {
    // O(log(min(n,m))) solution (standard)
}
```

### 98. Binary tree inorder traversal (iterative).
```cpp
vector<int> inorder(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    while (root || !st.empty()) {
        while (root) {
            st.push(root);
            root = root->left;
        }
        root = st.top(); st.pop();
        res.push_back(root->val);
        root = root->right;
    }
    return res;
}
```

### 99. Validate binary search tree (BST).
```cpp
bool isValidBST(TreeNode* root, long min = LONG_MIN, long max = LONG_MAX) {
    if (!root) return true;
    if (root->val <= min || root->val >= max) return false;
    return isValidBST(root->left, min, root->val) &&
           isValidBST(root->right, root->val, max);
}
```

### 100. Word break (DP).
```cpp
bool wordBreak(string s, vector<string>& dict) {
    unordered_set<string> st(dict.begin(), dict.end());
    vector<bool> dp(s.size()+1, false);
    dp[0] = true;
    for (int i = 1; i <= s.size(); i++)
        for (int j = 0; j < i; j++)
            if (dp[j] && st.count(s.substr(j, i-j))) {
                dp[i] = true;
                break;
            }
    return dp[s.size()];
}
```

---

Would you like me to turn this list into a **one-page cheat sheet** (condensed, interview-ready) or generate a **C++ senior-level mock interview** with follow-up questions for trading engineering roles?