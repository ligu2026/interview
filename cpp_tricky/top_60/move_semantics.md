# Top 60 C++ Interview Questions: Move Semantics & Perfect Forwarding

## **Section 1: Rvalue References (&&) - Questions 1-15**

### Q1: What's the difference between `T&&` and `const T&`?
```cpp
void foo(const int& x) { }  // binds to anything
void bar(int&& x) { }        // binds only to rvalues

int a = 10;
foo(a);     // OK
foo(20);    // OK
bar(a);     // Error! a is lvalue
bar(20);    // OK
bar(std::move(a)); // OK
```

### Q2: What are rvalues and lvalues?
```cpp
int x = 5;        // x is lvalue, 5 is rvalue
int& y = x;       // y is lvalue reference
int&& z = 10;     // z is rvalue reference to rvalue 10

// lvalue: has address, can appear on left of =
// rvalue: temporary, no address, appears on right
```

### Q3: Can you bind an rvalue reference to an lvalue?
```cpp
int x = 42;
int&& r1 = x;        // ERROR: cannot bind rvalue ref to lvalue
int&& r2 = std::move(x);  // OK: move casts lvalue to rvalue
int&& r3 = 42;       // OK: bind to rvalue
```

### Q4: What is a forwarding reference (universal reference)?
```cpp
template<typename T>
void forward_ref(T&& param) {  // NOT rvalue reference, but forwarding reference
    // T&& behaves differently based on T
}

// Rules:
// - If T is lvalue reference, T&& becomes lvalue reference
// - If T is non-reference, T&& becomes rvalue reference
```

### Q5: What happens with const rvalue references?
```cpp
void foo(const int&& x) { }  // Legal but rarely useful

const int&& r = 42;  // OK, but pointless
// const rvalue references can't be moved from efficiently
```

### Q6: Can functions be overloaded on rvalue/lvalue?
```cpp
void process(int& x) { cout << "lvalue\n"; }
void process(int&& x) { cout << "rvalue\n"; }

int a = 5;
process(a);    // lvalue
process(10);   // rvalue
process(std::move(a)); // rvalue
```

### Q7: What's the lifetime of a temporary bound to rvalue reference?
```cpp
struct Big { ~Big() { cout << "destroyed\n"; } };

void demo() {
    Big&& ref = Big();  // temporary lifetime extended to ref's scope
    cout << "using ref\n";
}  // destroyed here, not at semicolon
```

### Q8: Can you have arrays of rvalue references?
```cpp
// Arrays of references (including rvalue references) are illegal
int&& arr[10];  // ERROR: array of references

// But you can have:
vector<int&&> vec;  // Also illegal - same reason
```

### Q9: What's the type of `decltype(std::move(x))`?
```cpp
int x = 5;
decltype(std::move(x)) y = 10;  // y is int&&

auto&& z = std::move(x);  // z is int&&
```

### Q10: How do rvalue references interact with inheritance?
```cpp
class Base { 
public:
    virtual ~Base() {}
};

class Derived : public Base {};

void take(Base&& b) { }

Derived d;
take(std::move(d));  // OK: Derived&& converts to Base&&
```

### Q11: What's the difference between `int&&*` and `int*&&`?
```cpp
int x = 10;
int* p = &x;

int&&* rp = &x;    // ERROR: cannot create pointer to rvalue reference
int*&& rp2 = p;    // OK: rvalue reference to pointer
```

### Q12: Can rvalue references be reassigned?
```cpp
int a = 10, b = 20;
int&& ref = std::move(a);
ref = 30;           // OK: modifies a
ref = std::move(b); // OK: reassigns reference, now refers to b
```

### Q13: What happens with `auto&&` deduction?
```cpp
int x = 42;
auto&& a1 = x;        // a1 is int& (lvalue)
auto&& a2 = 42;       // a2 is int&& (rvalue)
auto&& a3 = std::move(x); // a2 is int&&

// auto&& is always a forwarding reference
```

### Q14: Can you have rvalue reference to function?
```cpp
void func() { }
void (*pfunc)() = func;

// Function rvalue references
void (&&rfunc)() = func;  // OK, but unusual
auto&& r = func;  // r is lvalue reference to function
```

### Q15: Are bitfields compatible with rvalue references?
```cpp
struct Flags {
    unsigned int flag1 : 1;
    unsigned int flag2 : 1;
};

Flags f;
// Cannot bind rvalue reference to bitfield
// unsigned int&& r = std::move(f.flag1); // ERROR
```

## **Section 2: std::move - Questions 16-30**

### Q16: What does `std::move` actually do?
```cpp
template<typename T>
decltype(auto) move(T&& param) {
    return static_cast<remove_reference_t<T>&&>(param);
}
// It's just a cast! No moving happens here.
```

### Q17: Does `std::move` move anything?
```cpp
vector<int> v1 = {1,2,3};
vector<int> v2 = std::move(v1);  // move constructor called
// std::move itself does nothing, just casts

v1.size();  // unspecified but valid state (typically empty)
```

### Q18: What happens after moving from an object?
```cpp
string s1 = "hello";
string s2 = std::move(s1);

// s1 is in valid but unspecified state
// Safe operations: assignment, destruction
s1 = "new value";  // OK
cout << s1.size(); // OK, but value unspecified
// s1.clear();     // OK
```

### Q19: Can you `std::move` a const object?
```cpp
const string s = "hello";
string s2 = std::move(s);  // Copies, doesn't move!

// Because move constructor takes string&&, not const string&&
// Const rvalue references bind to const, but can't modify
```

### Q20: Is `std::move` expensive?
```cpp
// std::move is O(1) - just a cast
// The actual move operation can be fast (pointer swaps)
vector<int> v1(1000000);
vector<int> v2 = std::move(v1);  // O(1) - just swaps pointers
```

### Q21: Can you move from a return value?
```cpp
vector<int> createVector() {
    vector<int> v = {1,2,3};
    return std::move(v);  // BAD! Prevents RVO/NRVO
}

vector<int> createVectorGood() {
    vector<int> v = {1,2,3};
    return v;  // Good: RVO or implicit move
}
```

### Q22: Does `std::move` work with `std::unique_ptr`?
```cpp
unique_ptr<int> p1 = make_unique<int>(10);
unique_ptr<int> p2 = std::move(p1);  // Ownership transferred
// p1 is now nullptr
// Good: unique_ptr is move-only
```

### Q23: What's wrong with this code?
```cpp
void process(string s) { }

string getString() { return "hello"; }

void bad() {
    string x = getString();
    process(std::move(x));  // Unnecessary, x is already expiring
    // Can't use x after this (unspecified state)
}
```

### Q24: Can you `std::move` an element from vector?
```cpp
vector<string> v = {"a", "b", "c"};
string s = std::move(v[1]);  // Moves element, v[1] now empty
// v[1] is in valid but unspecified state
v.erase(v.begin() + 1);  // Safe to remove
```

### Q25: Does `std::move` work with built-in types?
```cpp
int a = 10;
int b = std::move(a);  // Same as b = a (copy)
// No benefit, built-in types don't have move semantics
```

### Q26: What's the difference between `std::move` and `std::forward`?
```cpp
template<typename T>
void wrapper(T&& arg) {
    foo(std::move(arg));  // Always casts to rvalue
    bar(std::forward<T>(arg));  // Preserves original value category
}
```

### Q27: Why not just use `static_cast<T&&>` directly?
```cpp
// You could, but std::move is:
// 1. More readable
// 2. Less error-prone (auto deduces correctly)
// 3. Standard practice

void f() {
    int x = 5;
    int&& r = static_cast<int&&>(x);  // Works
    int&& r2 = std::move(x);          // Better
}
```

### Q28: Does `std::move` invalidate the source?
```cpp
struct MyType {
    int* data;
    MyType(MyType&& other) noexcept {
        data = other.data;
        other.data = nullptr;  // Must do this!
    }
};
// std::move doesn't invalidate; the move operation does
```

### Q29: Can `std::move` be used in constexpr context?
```cpp
constexpr int square(int x) { return x * x; }
constexpr int&& r = std::move(square(5));  // Not useful, but allowed

// C++20: std::move is constexpr
```

### Q30: What's the output?
```cpp
struct Noisy {
    Noisy() { cout << "construct\n"; }
    Noisy(const Noisy&) { cout << "copy\n"; }
    Noisy(Noisy&&) { cout << "move\n"; }
};

int main() {
    Noisy a;
    Noisy b = std::move(a);
    Noisy c = static_cast<Noisy&&>(a);
}
// Output: construct, move, move (static_cast also casts to rvalue)
```

## **Section 3: std::forward & Perfect Forwarding - Questions 31-40**

### Q31: What problem does `std::forward` solve?
```cpp
template<typename T>
void wrapper(T&& param) {
    // We want to call foo with same value category as original
    foo(param);  // Always passes as lvalue - WRONG
    foo(std::forward<T>(param));  // Preserves original category
}
```

### Q32: How does `std::forward` work?
```cpp
template<typename T>
T&& forward(remove_reference_t<T>& param) {
    return static_cast<T&&>(param);
}

// If T is int&: returns int& (lvalue)
// If T is int: returns int&& (rvalue)
```

### Q33: When to use `std::forward` vs `std::move`?
```cpp
class Widget {
    template<typename T>
    void setValue(T&& newValue) {
        // Use forward when value might be used again
        data = std::forward<T>(newValue);
        
        // Use move when we're done with the object
        internalBuffer = std::move(localData);
    }
};
```

### Q34: What's perfect forwarding failure?
```cpp
void foo(int&) { }
void foo(int&&) { }

template<typename T>
void wrapper(T&& param) {
    foo(std::forward<T>(param));  // Perfect
}

int x = 5;
wrapper(x);     // Calls foo(int&)
wrapper(10);    // Calls foo(int&&)

// But fails with braced initializers, bitfields, nullptr_t
```

### Q35: Can you perfect forward multiple arguments?
```cpp
template<typename... Args>
void wrapper(Args&&... args) {
    target(std::forward<Args>(args)...);  // Pack expansion
}

// Example
void target(int a, double b, string c) { }

wrapper(10, 3.14, "hello");
```

### Q36: What happens with `std::forward` and `auto`?
```cpp
auto&& x = someFunction();
auto y = std::forward<decltype(x)>(x);  // Preserves category

// Better: use decltype(auto)
decltype(auto) z = std::forward<decltype(x)>(x);
```

### Q37: Why can't you forward `this` in member functions?
```cpp
class Widget {
    // C++23: Deducing this
    template<typename Self>
    void func(this Self&& self) {
        // Can forward self now
        helper(std::forward<Self>(self));
    }
    
    // C++17 and earlier: need lvalue/rvalue overloads
};
```

### Q38: Does `std::forward` work with non-forwarding references?
```cpp
void process(int& x) { }
void process(int&& x) { }

int x = 5;
int& lr = x;
int&& rr = 10;

process(std::forward<int&>(lr));  // lvalue
process(std::forward<int>(rr));   // rvalue (but rr destroyed after)
```

### Q39: What's wrong with this perfect forwarding?
```cpp
template<typename T>
void wrapper(T param) {  // Not forwarding reference (by value)
    foo(std::forward<T>(param));  // WRONG: param is always lvalue
}

template<typename T>
void wrapper(T&& param) {  // Forwarding reference
    foo(std::forward<T>(param));  // Correct
}
```

### Q40: How to perfect forward return values?
```cpp
template<typename... Args>
decltype(auto) wrapper(Args&&... args) {
    return target(std::forward<Args>(args)...);
}

// Or with trailing return type
template<typename... Args>
auto wrapper(Args&&... args) -> decltype(target(std::forward<Args>(args)...)) {
    return target(std::forward<Args>(args)...);
}
```

## **Section 4: Move Operations & Rule of Five - Questions 41-50**

### Q41: What is the Rule of Five?
```cpp
class Resource {
    // If you define any of these, define all five
public:
    Resource();                          // Constructor
    ~Resource();                         // Destructor
    Resource(const Resource&);           // Copy constructor
    Resource& operator=(const Resource&);// Copy assignment
    Resource(Resource&&) noexcept;       // Move constructor
    Resource& operator=(Resource&&) noexcept; // Move assignment
};
```

### Q42: How to implement move constructor?
```cpp
class Buffer {
    int* data;
    size_t size;
public:
    // Move constructor
    Buffer(Buffer&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;  // Leave in valid state
    }
    
    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data;      // Cleanup current
            data = other.data;  // Steal resources
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

### Q43: Why `noexcept` on move operations?
```cpp
vector<MyType> vec;
vec.push_back(MyType());  // If MyType move is noexcept, vector uses move
// Otherwise, vector uses copy for strong exception guarantee
```

### Q44: What's the default move behavior?
```cpp
struct DefaultMove {
    string s;    // movable
    vector<int> v; // movable
    // Default move does memberwise move
};

// Implicit move constructor generated
// Implicit move assignment generated

struct NoMove {
    NoMove(const NoMove&) = default;  // Copy prevents move generation
    string s;  // Even though string is movable
};
// No move operations generated!
```

### Q45: Can you force move generation?
```cpp
class ForceMove {
    string s;
public:
    ForceMove(ForceMove&&) = default;   // Forces move generation
    ForceMove& operator=(ForceMove&&) = default;
    
    // But must also handle copy if needed
    ForceMove(const ForceMove&) = default;
    ForceMove& operator=(const ForceMove&) = default;
};
```

### Q46: When is move not faster than copy?
```cpp
// Move is same as copy for:
int x;              // Trivial types
array<int, 1000>;   // No dynamic allocation
struct Small { int a,b,c; };  // Small, trivially copyable

// Move is faster for:
vector<string>;     // Just swaps pointers
unique_ptr<int>;    // Transfers ownership
string;             // Swaps pointer to heap data
```

### Q47: How to delete move operations?
```cpp
class Immovable {
public:
    Immovable(Immovable&&) = delete;
    Immovable& operator=(Immovable&&) = delete;
    // Must also handle copy
};
```

### Q48: What's object slicing with move?
```cpp
class Base {
    string data;
public:
    Base(Base&&) = default;
};

class Derived : public Base {
    string extra;
public:
    // Move constructor slices if not careful
    Derived(Derived&& other) 
        : Base(std::move(other)),  // Must move base part
          extra(std::move(other.extra)) { }
};
```

### Q49: When to use `std::move` in return statements?
```cpp
// DON'T: Prevents RVO
string bad() {
    string s = "hello";
    return std::move(s);  // Bad
}

// DO: Let compiler optimize
string good1() {
    string s = "hello";
    return s;  // Good: RVO or implicit move
}

// DO: When returning different objects
string conditional(bool flag) {
    string a = "hello", b = "world";
    return std::move(flag ? a : b);  // OK: can't use RVO here
}
```

### Q50: How to move with containers?
```cpp
vector<unique_ptr<int>> vec;
vec.push_back(make_unique<int>(10));

// Transfer ownership
auto ptr = std::move(vec[0]);

// Move entire container
vector<unique_ptr<int>> vec2 = std::move(vec);
// vec is now empty
```

## **Section 5: Copy Elision & Performance - Questions 51-60**

### Q51: What is Copy Elision (RVO/NRVO)?
```cpp
struct Expensive { Expensive(const Expensive&) { sleep(1); } };

Expensive create() {
    return Expensive();  // RVO: constructs directly in return location
}

Expensive create_named() {
    Expensive local;
    return local;  // NRVO: may elide copy (C++17 guarantees in some cases)
}

// C++17 guarantees elision for prvalues
```

### Q52: Guaranteed Copy Elision in C++17?
```cpp
class NoCopy {
    NoCopy(const NoCopy&) = delete;  // Not copyable
public:
    NoCopy() = default;
};

NoCopy factory() {
    return NoCopy();  // C++17: OK, guaranteed elision
}

NoCopy make() {
    NoCopy x;
    return x;  // Error: copy constructor deleted (not guaranteed)
}
```

### Q53: Why is move faster than copy?
```cpp
// Copy: O(n) deep copy
vector<int> copy_vec(const vector<int>& v) {
    vector<int> result;  // New allocation
    result = v;          // Copies all elements O(n)
    return result;
}

// Move: O(1) pointer swaps
vector<int> move_vec(vector<int>&& v) {
    vector<int> result;
    result = std::move(v);  // Just swaps pointers O(1)
    return result;
}
```

### Q54: What's the performance penalty of unnecessary copies?
```cpp
void process(vector<int> v) { }  // Copies if lvalue passed

vector<int> data(1000000);
process(data);           // O(1M) copy
process(std::move(data)); // O(1) move (data becomes empty)

void process_improved(const vector<int>& v) { 
    vector<int> local = v;  // Explicit copy when needed
}
```

### Q55: How does RVO interact with move?
```cpp
struct Widget {
    Widget() = default;
    Widget(Widget&&) = delete;  // No move
};

Widget create() {
    return Widget();  // Still works - RVO elides move
}

// Without RVO, would try to use deleted move constructor
```

### Q56: What's the cost of `std::move` on containers?
```cpp
vector<string> v(10000, "hello");
string s = std::move(v[5000]);  // O(1) - just swap pointers
// But v[5000] now empty string (unspecified)

// Moving entire vector: O(1)
vector<string> v2 = std::move(v);
```

### Q57: Why use `std::move` with lambda captures?
```cpp
unique_ptr<int> ptr = make_unique<int>(10);

auto lambda1 = [ptr = std::move(ptr)]() {  // C++14 init capture
    return *ptr;
};
// ptr now nullptr

auto lambda2 = [ptr = std::move(ptr)]() mutable {  // Need mutable
    ptr.reset(new int(20));
};
```

### Q58: Move semantics with SSO (Small String Optimization)?
```cpp
string s1 = "small";  // Uses SSO (typically 16-22 bytes)
string s2 = std::move(s1);  // Still cheap, but might copy if small

string s3 = "very long string that exceeds SSO buffer";
string s4 = std::move(s3);  // Always O(1) pointer swap

// SSO implementation varies by stdlib
```

### Q59: What's the performance difference for primitive arrays?
```cpp
array<int, 1000> arr1;  // On stack
array<int, 1000> arr2 = std::move(arr1);  // Still copies all 1000 ints

vector<int> vec1(1000);
vector<int> vec2 = std::move(vec1);  // O(1) - just swaps pointers

// std::array has no move advantage over copy
// Use vector for dynamic arrays to benefit from move
```

### Q60: Benchmark: Move vs Copy
```cpp
#include <chrono>

template<typename T>
auto benchmark_copy(T&& original) {
    auto start = chrono::steady_clock::now();
    T copy = original;  // Copy
    auto end = chrono::steady_clock::now();
    return end - start;
}

template<typename T>
auto benchmark_move(T&& original) {
    auto start = chrono::steady_clock::now();
    T moved = std::move(original);  // Move
    auto end = chrono::steady_clock::now();
    return end - start;
}

// Results (typical):
// vector<int>(1000000): copy ~10ms, move ~0.001ms
// string(1000 chars): copy ~0.001ms, move ~0.0001ms
// int: same for both (just copy)
```

## **Key Takeaways**

1. **Rvalue references (`&&`)** bind to temporaries and enable move semantics
2. **`std::move`** is just a cast to rvalue; no moving happens until construction/assignment
3. **`std::forward`** preserves value categories for perfect forwarding in templates
4. **Move operations** steal resources (O(1)) vs copy (O(n)) for heap-allocated data
5. **Copy elision** (RVO/NRVO) eliminates copies/moves entirely, prefer over explicit `std::move`
6. **Rule of Five**: Define all or none of copy/move/dtor for resource-managing classes
7. **`noexcept` moves** enable optimal container operations
8. **After move**, source is in valid but unspecified state (only assign/destroy)
9. **Don't `std::move` return values** - prevents RVO
10. **Perfect forwarding** enables generic code that preserves argument categories