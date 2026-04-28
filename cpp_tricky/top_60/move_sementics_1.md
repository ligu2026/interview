i# Top 60 C++ Interview Questions: Move Semantics & Perfect Forwarding
## Comprehensive Guide with Tricky Examples

---

## **Section 1: Rvalue References (&&) Fundamentals**

### Q1: What exactly is an rvalue reference and how does it differ from lvalue reference?
```cpp
int x = 42;        // x is lvalue
int& lr = x;       // lvalue reference: binds to x
int&& rr = 42;     // rvalue reference: binds to temporary

// Key differences:
// 1. lvalue reference can bind to lvalue, rvalue reference to rvalue
// 2. lvalue reference has identity, rvalue reference represents disposable value
// 3. rvalue reference extends lifetime of temporary
```

### Q2: Why can't you bind an rvalue reference to an lvalue directly?
```cpp
int x = 10;
int&& r = x;  // ERROR: cannot bind rvalue reference to lvalue

// Reason: rvalue reference implies "this value can be modified and will be destroyed"
// lvalues have persistent identity, violating this assumption

// Solution: std::move creates rvalue from lvalue
int&& r2 = std::move(x);  // OK - explicit intent to treat as rvalue
```

### Q3: What happens when you take address of an rvalue reference?
```cpp
int&& rr = 42;
int* p = &rr;     // OK: rr is an lvalue (named reference)
*p = 100;         // Modifies the temporary
cout << rr;       // Prints 100

// Important: Named rvalue reference behaves as lvalue!
void foo(int&& x) {
    int* p = &x;  // OK - x is lvalue inside function
    int&& y = std::move(x);  // Need move to treat as rvalue again
}
```

### Q4: Can you have reference to reference and what does it collapse to?
```cpp
// Reference collapsing rules:
// & &  -> &
// & && -> &
// && & -> &
// && && -> &&

template<typename T>
void collapse_example(T&& t) {
    // If T = int&:  int& && -> int& (lvalue reference)
    // If T = int:   int && -> int&& (rvalue reference)
}

using T1 = int&;
using T2 = T1&&;  // collapses to int&

using T3 = int&&;
using T4 = T3&&;  // collapses to int&&
```

### Q5: What's the difference between `T&&` in template vs concrete type?
```cpp
// Concrete type: ALWAYS rvalue reference
void concrete(int&& x) { }  // Only binds to rvalues

// Template: forwarding reference (universal reference)
template<typename T>
void forwarding(T&& x) {     // Binds to lvalues AND rvalues
    // T's type changes based on argument
}

int a = 5;
concrete(a);       // ERROR
forwarding(a);     // OK - T becomes int&
forwarding(10);    // OK - T becomes int
```

### Q6: How does const affect rvalue references?
```cpp
void foo(const int&& x) {    // Legal but almost useless
    // Cannot move from x because const
    // int y = std::move(x);  // Calls const int& copy constructor!
}

const int&& dangerous() {
    return 42;  // Returns dangling reference if returning local
}

// Rarely useful: const rvalue reference prevents modification
// but also prevents efficient move semantics
```

### Q7: What's the lifetime extension rules for rvalue references?
```cpp
struct Trace {
    Trace() { cout << "Create\n"; }
    ~Trace() { cout << "Destroy\n"; }
};

Trace&& ref = Trace();     // Lifetime extended to ref's scope
cout << "Middle\n";
// Output: Create, Middle, Destroy

// No extension when returned from function
Trace&& bad() {
    Trace t;
    return std::move(t);   // Dangling reference!
}

// Exception: binding to temporary in constructor initializer list
struct Holder {
    Trace&& ref;
    Holder(Trace&& t) : ref(std::move(t)) { }  // Doesn't extend lifetime!
};
```

### Q8: Can you overload on rvalue/lvalue references for arrays?
```cpp
void process(int (&arr)[10]) {        // lvalue array of size 10
    cout << "lvalue array\n";
}

void process(int (&&arr)[10]) {       // rvalue array
    cout << "rvalue array\n";
}

int arr[10];
process(arr);          // lvalue version
process(std::move(arr)); // rvalue version
process({1,2,3,4,5,6,7,8,9,10}); // rvalue version (C++11)
```

### Q9: What's the type of `decltype` on expressions with rvalue references?
```cpp
int x = 5;
int&& r = std::move(x);

decltype(x) a;      // int
decltype((x)) b;    // int& (parentheses matter!)
decltype(r) c;      // int&&
decltype((r)) d;    // int& (r is lvalue in expression)

// Rule: decltype(name) gives declared type
// decltype((name)) gives reference based on value category
```

### Q10: How do rvalue references work with inheritance hierarchies?
```cpp
class Base { 
    virtual void foo() {}
};

class Derived : public Base {
    int extra[1000];  // Derived has more resources
};

void takeBase(Base&& b) {
    Base b2 = std::move(b);  // Slicing if b is Derived!
}

Derived d;
takeBase(std::move(d));  // Object slicing: Derived parts lost

// Solution: Perfect forwarding in templates
template<typename T>
void takeBaseForward(T&& b) {
    Base b2 = std::forward<T>(b);  // Still slices!
    // Need virtual move semantics or CRTP
}
```

### Q11: Can you have a container of rvalue references?
```cpp
// vector<int&&> vec;  // ERROR: cannot form reference to reference

// Workaround: use std::reference_wrapper
vector<reference_wrapper<int>> vec;
int x = 10;
vec.push_back(ref(x));  // Stores lvalue reference

// Or use pointers
vector<int*> ptr_vec;
ptr_vec.push_back(&x);

// Rvalue references can't be stored because they'd dangle
```

### Q12: What's the difference between `std::move` and static cast?
```cpp
template<typename T>
decltype(auto) my_move(T&& param) {
    return static_cast<std::remove_reference_t<T>&&>(param);
}

// They're identical in behavior:
int x = 5;
int&& r1 = static_cast<int&&>(x);
int&& r2 = std::move(x);

// But std::move is:
// 1. More readable
// 2. Less error-prone (automatically deduces type)
// 3. Self-documenting
// 4. Standard practice
```

### Q13: How does rvalue reference work with function pointers?
```cpp
void func(int&& x) { x = 100; }

void (*fp)(int&&) = &func;  // Function pointer to rvalue reference

void caller() {
    int x = 10;
    fp(std::move(x));  // x becomes 100
}

// With function objects
struct Functor {
    void operator()(int&& x) const {
        x = 200;
    }
};

Functor f;
f(std::move(x));
```

### Q14: Can you have rvalue reference to void?
```cpp
void foo() { }
void&& r = foo();  // ERROR: cannot form rvalue reference to void

// But this works:
void(*&&p)() = &foo;  // Rvalue reference to function pointer
p();  // Calls foo
```

### Q15: What are the exception safety implications of rvalue references?
```cpp
class UnsafeMove {
    int* data;
public:
    UnsafeMove(UnsafeMove&& other) {  // No noexcept
        data = other.data;
        other.data = nullptr;
        // If this throws, other is left modified!
    }
};

class SafeMove {
    int* data;
public:
    SafeMove(SafeMove&& other) noexcept {  // Guarantee no throw
        data = other.data;
        other.data = nullptr;
    }
};

// vector::push_back uses move if noexcept, otherwise copy
vector<UnsafeMove> v1;  // Uses copy for safety
vector<SafeMove> v2;    // Uses move for performance
```

---

## **Section 2: `std::move` Deep Dive**

### Q16: Does `std::move` actually move memory around?
```cpp
// std::move implementation (simplified)
template<typename T>
remove_reference_t<T>&& move(T&& t) noexcept {
    return static_cast<remove_reference_t<T>&&>(t);
}

// It's just a cast! No memory operations at all
string s1 = "hello";
string s2 = std::move(s1);  // Now move constructor steals resources

// The move is performed by the constructor, not by std::move
```

### Q17: What state is an object in after being moved from?
```cpp
vector<int> v = {1,2,3,4,5};
vector<int> w = std::move(v);

// v is in "valid but unspecified state"
// Legal operations:
v.clear();           // OK
v = {1,2,3};         // OK
v.size();            // OK (probably 0)
v.empty();           // OK

// Dangerous operations (implementation defined):
cout << v[0];        // May crash, may return something
v.push_back(10);     // OK but behavior unspecified until reinitialized

// Best practice: don't use moved-from objects except to assign/destroy
```

### Q18: Can you move from a `const` object?
```cpp
struct Widget {
    string s;
    Widget(Widget&&) = default;  // Moves s
};

const Widget w1;
Widget w2 = std::move(w1);  // Copies! Not moves!

// Why? The move constructor takes Widget&&, not const Widget&&
// const Widget&& binds to const, but can't steal resources

// Solution: Don't mark move operations if you have const objects
```

### Q19: Why does `std::move` sometimes cause compilation errors?
```cpp
class NoMove {
    NoMove(NoMove&&) = delete;  // Move not allowed
};

NoMove a;
NoMove b = std::move(a);  // ERROR: uses deleted move constructor

// Solution: Use copy or redesign
NoMove c = a;  // OK if copy exists

// Common pitfall with unique_ptr:
unique_ptr<int> p1(new int(5));
unique_ptr<int> p2 = p1;  // ERROR: unique_ptr not copyable
unique_ptr<int> p3 = std::move(p1);  // OK: transfers ownership
```

### Q20: What's the performance cost of `std::move` on different types?
```cpp
// Benchmark comparison
template<typename T>
void compare_move_copy(T&& original) {
    // Copy: O(n) for most types
    auto start = chrono::high_resolution_clock::now();
    T copy = original;
    auto copy_time = chrono::high_resolution_clock::now() - start;
    
    // Move: O(1) for heap-allocated types
    start = chrono::high_resolution_clock::now();
    T moved = std::move(original);
    auto move_time = chrono::high_resolution_clock::now() - start;
    
    cout << "Copy: " << copy_time.count() << "ns\n";
    cout << "Move: " << move_time.count() << "ns\n";
}

// Results:
// vector<int>(1M): Copy ~5,000,000ns, Move ~50ns
// string(1KB): Copy ~1,000ns, Move ~10ns
// int: Copy ~5ns, Move ~5ns (identical)
// array<int,1000>: Copy ~5,000ns, Move ~5,000ns (copy anyway)
```

### Q21: Can you `std::move` from `std::initializer_list`?
```cpp
auto il = {1,2,3,4,5};
vector<int> v1 = std::move(il);  // ERROR: initializer_list has no move

// initializer_list elements are const, can't move from them
// Reason: initializer_list stores references to temporaries

// Workaround: Use array or vector directly
vector<int> source = {1,2,3,4,5};
vector<int> dest = std::move(source);  // OK
```

### Q22: How does `std::move` interact with `std::async` and futures?
```cpp
void process(string&& s) { }

string expensive_string() { return string(1000000, 'x'); }

int main() {
    // Moving into async
    auto future = std::async(std::launch::async, 
        [](string s) { process(std::move(s)); },
        expensive_string()  // Temporary, moves into lambda
    );
    
    // Moving from future
    string result = future.get();  // May be moved depending on implementation
    
    // Don't do this:
    string s = expensive_string();
    auto bad = std::async(process, std::move(s));  // s may be moved twice!
}
```

### Q23: What's wrong with returning `std::move(local)`?
```cpp
string bad_return() {
    string local = "hello";
    return std::move(local);  // BAD: prevents RVO/NRVO
}

string good_return() {
    string local = "hello";
    return local;  // GOOD: RVO or implicit move
}

// Even worse with multiple returns:
string conditional(bool flag) {
    string a = "hello", b = "world";
    if (flag) return std::move(a);  // Still bad
    else return std::move(b);       // Still bad
}

// Only use std::move in returns when you can't use RVO:
string conditional_good(bool flag) {
    string result;
    if (flag) result = "hello";
    else result = "world";
    return result;  // NRVO possible
}
```

### Q24: Does `std::move` work with `std::ref` and `std::cref`?
```cpp
int x = 10;
auto ref = std::ref(x);  // reference_wrapper<int>
auto moved = std::move(ref);  // Moves the wrapper, not x

// reference_wrapper is trivially copyable, move is same as copy
// To move the referenced object:
auto moved_value = std::move(ref.get());

// With std::cref (const reference):
const int y = 20;
auto cref = std::cref(y);
auto moved_cref = std::move(cref);  // Wrapper moved, y unchanged
```

### Q25: Can you use `std::move` in constexpr contexts?
```cpp
// C++20: std::move is constexpr
constexpr int square(int x) { return x * x; }

constexpr int&& r = std::move(square(5));  // C++20 OK

// But rvalue reference to temporary in constexpr has limited use
// constexpr objects must be destructible at compile time

// Practical constexpr move:
struct ConstexprString {
    char* data;
    constexpr ConstexprString(ConstexprString&& other) noexcept
        : data(other.data) {
        other.data = nullptr;  // Can't modify in constexpr? Actually C++14 allows
    }
};
```

### Q26: What's the output of this program?
```cpp
struct Detective {
    string name;
    Detective(string n) : name(n) {}
    Detective(const Detective& other) : name(other.name + "_copy") {}
    Detective(Detective&& other) noexcept : name(std::move(other.name) + "_move") {}
};

int main() {
    Detective d1("Sherlock");
    Detective d2 = d1;                    // Copy
    Detective d3 = std::move(d1);         // Move
    Detective d4 = Detective("Watson");   // Move (temporary)
    
    cout << d1.name << endl;    // "" (moved-from)
    cout << d2.name << endl;    // "Sherlock_copy"
    cout << d3.name << endl;    // "Sherlock_move"
    cout << d4.name << endl;    // "Watson_move"
}
```

### Q27: How does move behave with polymorphic types?
```cpp
class Base {
    vector<int> data = {1,2,3};
public:
    virtual ~Base() = default;
    Base(Base&&) = default;
};

class Derived : public Base {
    vector<int> more_data = {4,5,6};
public:
    Derived(Derived&&) = default;  // Moves both parts
};

Derived d1;
Derived d2 = std::move(d1);  // OK: moves entire object

// But slicing with Base reference:
Base&& bref = std::move(d1);  // Rvalue reference to Base portion
Base b = std::move(bref);     // Slicing! Only moves Base part
```

### Q28: Can `std::move` be used with function parameters to avoid copies?
```cpp
// Bad: overload for lvalue and rvalue
class Widget {
    void setData(const string& s) { data = s; }  // Copy
    void setData(string&& s) { data = std::move(s); }  // Move
    string data;
};

// Better: pass by value, then move
class WidgetBetter {
    void setData(string s) { data = std::move(s); }  
    // For lvalue: copy into param, then move
    // For rvalue: move into param, then move again
    string data;
};

// Even better: perfect forwarding
class WidgetPerfect {
    template<typename T>
    void setData(T&& s) { 
        data = std::forward<T>(s);  // Preserves category
    }
    string data;
};
```

### Q29: What's the interaction between `std::move` and `noexcept`?
```cpp
void risky_move(string&& s) {  // Could throw
    // some operation that might throw
}

void safe_move(string&& s) noexcept {  // Guarantees no throw
    // Only noexcept operations
}

template<typename T>
void move_into_vector(vector<T>& vec, T&& elem) {
    // vector's strong exception guarantee requires move to be noexcept
    vec.push_back(std::move(elem));  // Uses copy if move can throw
}

// Mark moves noexcept when possible:
struct Good {
    int* ptr;
    Good(Good&& other) noexcept : ptr(other.ptr) {
        other.ptr = nullptr;
    }
};
```

### Q30: How to properly move from a function parameter?
```cpp
void process(unique_ptr<int> p) {
    // p is a local variable here
}

void caller() {
    unique_ptr<int> ptr = make_unique<int>(10);
    
    // Option 1: Move at call site
    process(std::move(ptr));  // ptr becomes nullptr
    
    // Option 2: Move inside function (if passed by value)
    // Not applicable here
    
    // Don't: use after move
    // cout << *ptr;  // CRASH: ptr is nullptr
}

// For multiple parameters:
void multi(unique_ptr<int> a, unique_ptr<int> b) { }

auto a = make_unique<int>(10);
auto b = make_unique<int>(20);
multi(std::move(a), std::move(b));  // Order of evaluation unspecified!
```

---

## **Section 3: `std::forward` & Perfect Forwarding**

### Q31: What problem does `std::forward` solve that `std::move` doesn't?
```cpp
template<typename T>
void wrapper(T&& param) {
    // Using std::move always casts to rvalue
    foo(std::move(param));  // Always treats as rvalue - wrong for lvalues!
    
    // Using std::forward preserves value category
    bar(std::forward<T>(param));  // Preserves original category
}

void foo(int& x) { cout << "lvalue\n"; }
void foo(int&& x) { cout << "rvalue\n"; }

int x = 5;
wrapper(x);   // foo gets rvalue? No! Should be lvalue
wrapper(10);  // foo gets rvalue correctly

// Solution: Use forward in forwarding references
template<typename T>
void wrapper_correct(T&& param) {
    foo(std::forward<T>(param));  // Perfect forwarding
}
```

### Q32: How does `std::forward` actually work internally?
```cpp
// Simplified implementation of std::forward
template<typename T>
T&& forward(typename remove_reference<T>::type& param) {
    return static_cast<T&&>(param);
}

// With reference collapsing:
// Case 1: T = int& (from lvalue)
// forward<int&> returns int& (lvalue reference)

// Case 2: T = int (from rvalue)
// forward<int> returns int&& (rvalue reference)

// Example:
template<typename T>
void wrapper(T&& arg) {
    // If arg came from lvalue, T is int&, forward returns int&
    // If arg came from rvalue, T is int, forward returns int&&
    target(std::forward<T>(arg));
}
```

### Q33: Can you perfect forward with multiple arguments?
```cpp
// Variadic template perfect forwarding
template<typename... Args>
auto perfect_forward(Args&&... args) {
    // Forward each argument with its category preserved
    return target(std::forward<Args>(args)...);
}

// Example with different categories
void target(int& a, int&& b, const string& c) {
    cout << "lvalue=" << a << ", rvalue=" << b << ", const=" << c << endl;
}

int main() {
    int x = 10;
    string s = "hello";
    
    perfect_forward(x, 20, s);  
    // x -> lvalue, 20 -> rvalue, s -> const lvalue
    
    perfect_forward(30, std::move(x), std::move(s));
    // 30 -> rvalue, x -> rvalue, s -> rvalue
}
```

### Q34: What's wrong with this perfect forwarding attempt?
```cpp
// Wrong: passing by value
template<typename T>
void bad_forward(T arg) {
    foo(std::forward<T>(arg));  // arg is always lvalue!
}

// Wrong: not using template
void bad_forward2(int&& arg) {
    foo(std::forward<int>(arg));  // Always rvalue reference
}

// Wrong: using wrong type for forward
template<typename T>
void bad_forward3(T&& arg) {
    foo(std::forward<decltype(arg)>(arg));  // Close but not quite right
}

// Correct:
template<typename T>
void good_forward(T&& arg) {
    foo(std::forward<T>(arg));  // Perfect
}
```

### Q35: How does perfect forwarding work with initializer lists?
```cpp
void take_list(std::initializer_list<int> list) { }

template<typename T>
void forward_to_list(T&& arg) {
    take_list(std::forward<T>(arg));  // ERROR: can't forward initializer_list
}

int main() {
    // Can't forward braced initializers
    forward_to_list({1,2,3});  // ERROR: can't deduce type
    
    // Workaround: explicit type
    forward_to_list(std::initializer_list<int>{1,2,3});  // OK
    
    // Or use auto
    auto il = {1,2,3};
    forward_to_list(il);  // OK, but il is lvalue
}
```

### Q36: What's the difference between forward and move in return statements?
```cpp
template<typename T>
decltype(auto) forward_and_move(T&& param) {
    if constexpr (std::is_lvalue_reference_v<T>) {
        return std::forward<T>(param);  // Returns lvalue reference
    } else {
        return std::move(param);  // Returns rvalue reference
    }
}

// Better: just use forward
template<typename T>
decltype(auto) better_forward(T&& param) {
    return std::forward<T>(param);  // Works for both
}

// Even better: decltype(auto) with perfect forwarding
template<typename F, typename... Args>
decltype(auto) invoke(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

### Q37: Can you perfect forward `this` in member functions (C++23)?
```cpp
// C++23: Deducing this
class Widget {
    // Old way: need two overloads
    void process() & { cout << "lvalue\n"; }
    void process() && { cout << "rvalue\n"; }
    
    // C++23: Deducing this
    template<typename Self>
    void process_modern(this Self&& self) {
        // Can forward self
        helper(std::forward<Self>(self));
    }
    
    // Even better: explicit object parameter
    void process_cpp23(this Widget&& self) {  // rvalue version
        helper(std::move(self));
    }
    
    void process_cpp23(this const Widget& self) {  // const lvalue
        helper(self);
    }
};
```

### Q38: How does perfect forwarding interact with overload resolution?
```cpp
void foo(int&& x) { cout << "rvalue\n"; }
void foo(int& x) { cout << "lvalue\n"; }
void foo(const int& x) { cout << "const lvalue\n"; }

template<typename T>
void forwarder(T&& arg) {
    foo(std::forward<T>(arg));
}

int main() {
    int x = 5;
    const int cx = 10;
    
    forwarder(x);   // Calls foo(int&)
    forwarder(cx);  // Calls foo(const int&)
    forwarder(15);  // Calls foo(int&&)
    forwarder(std::move(x));  // Calls foo(int&&)
    forwarder(static_cast<const int&&>(20));  // Calls foo(const int&)
}
```

### Q39: What's the performance cost of perfect forwarding?
```cpp
// Perfect forwarding is zero-cost abstraction
template<typename F, typename... Args>
auto invoke(F&& f, Args&&... args) {
    // All forwarding happens at compile time
    // No runtime overhead
    return std::forward<F>(f)(std::forward<Args>(args)...);
}

// Generated code is identical to direct call:
int add(int a, int b) { return a + b; }

// Direct call
int result1 = add(5, 10);

// Forwarded call (compiles to same assembly)
int result2 = invoke(add, 5, 10);
```

### Q40: How to perfect forward when you need to modify arguments?
```cpp
template<typename T>
void modify_and_forward(T&& arg) {
    // Need to modify arg before forwarding
    arg += 10;  // Modify the value
    
    // But now arg is lvalue! Need to recapture category
    if constexpr (std::is_rvalue_reference_v<T&&>) {
        foo(std::move(arg));  // Forward as rvalue
    } else {
        foo(arg);  // Forward as lvalue
    }
}

// Alternative: store original category
template<typename T>
void modify_and_forward2(T&& arg) {
    using category = decltype(std::forward<T>(arg));
    arg += 10;
    foo(std::forward<category>(arg));  // Forward with original category
}
```

---

## **Section 4: Move Operations & The Rule of Five**

### Q41: When are move operations automatically generated?
```cpp
struct AutoMove {
    string s;
    vector<int> v;
    // Move constructor/assignment auto-generated
};

struct NoAutoMove1 {
    string s;
    NoAutoMove1(const NoAutoMove1&) = default;  // User-declared copy
    // Move not generated (rule of zero violated)
};

struct NoAutoMove2 {
    string s;
    NoAutoMove2(NoAutoMove2&&) = delete;  // Explicitly deleted
    // No move
};

struct NoAutoMove3 {
    string s;
    ~NoAutoMove3() {}  // User-declared destructor
    // Move not generated (C++11), generated but deprecated (C++20)
};

// Best practice: Rule of Zero or Rule of Five
```

### Q42: How to implement move operations for a resource-managing class?
```cpp
class DynamicBuffer {
    char* data;
    size_t size;
    
public:
    // Constructor
    DynamicBuffer(size_t sz) : data(new char[sz]), size(sz) {}
    
    // Destructor
    ~DynamicBuffer() { delete[] data; }
    
    // Copy operations
    DynamicBuffer(const DynamicBuffer& other) 
        : data(new char[other.size]), size(other.size) {
        std::copy(other.data, other.data + size, data);
    }
    
    DynamicBuffer& operator=(const DynamicBuffer& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new char[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
    
    // Move operations
    DynamicBuffer(DynamicBuffer&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
    
    DynamicBuffer& operator=(DynamicBuffer&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

### Q43: What's the copy-and-swap idiom and how does it relate to move?
```cpp
class Widget {
    string name;
    vector<int> data;
    
public:
    // Unified assignment operator using copy-and-swap
    Widget& operator=(Widget other) {  // Pass by value (copy or move)
        swap(*this, other);
        return *this;
    }
    
    friend void swap(Widget& a, Widget& b) {
        using std::swap;
        swap(a.name, b.name);
        swap(a.data, b.data);
    }
};

// How it works:
Widget a, b;
a = b;        // Pass by value copies b -> other, then swap
a = std::move(b); // Pass by value moves b -> other, then swap
```

### Q44: Why should move operations be `noexcept`?
```cpp
class NoexceptMove {
    vector<int> data;
public:
    NoexceptMove(NoexceptMove&&) noexcept = default;
};

class ThrowingMove {
    vector<int> data;
public:
    ThrowingMove(ThrowingMove&&) /* not noexcept */ {}
};

// vector reallocation behavior:
vector<NoexceptMove> v1;
v1.reserve(1);
v1.emplace_back();  // No reallocation yet
v1.emplace_back();  // Reallocation: uses move (fast)

vector<ThrowingMove> v2;
v2.reserve(1);
v2.emplace_back();
v2.emplace_back();  // Reallocation: uses copy (slow!) for safety

// Never throw from move operations if possible
```

### Q45: What's the difference between move and swap?
```cpp
struct MyType {
    vector<int> data;
    
    // Move: transfers ownership, source becomes empty
    MyType(MyType&& other) noexcept 
        : data(std::move(other.data)) { }
    
    // Swap: exchanges values, both remain valid
    void swap(MyType& other) noexcept {
        data.swap(other.data);
    }
};

// Performance:
// Move: O(1) for heap data
// Swap: O(1) for heap data (similar performance)

// Semantic difference:
vector<int> a = {1,2,3};
vector<int> b = {4,5,6};

auto c = std::move(a);  // a becomes empty
std::swap(a, b);        // a gets {4,5,6}, b gets {1,2,3}
```

### Q46: How to handle move operations with inheritance?
```cpp
class Base {
    vector<int> base_data;
public:
    Base(Base&& other) noexcept 
        : base_data(std::move(other.base_data)) { }
    Base& operator=(Base&& other) noexcept {
        if (this != &other) {
            base_data = std::move(other.base_data);
        }
        return *this;
    }
};

class Derived : public Base {
    vector<int> derived_data;
public:
    Derived(Derived&& other) noexcept 
        : Base(std::move(other)),  // Move base part
          derived_data(std::move(other.derived_data)) { }
    
    Derived& operator=(Derived&& other) noexcept {
        if (this != &other) {
            Base::operator=(std::move(other));  // Move base part
            derived_data = std::move(other.derived_data);
        }
        return *this;
    }
};
```

### Q47: Can you have virtual move operations?
```cpp
// C++ doesn't have virtual move constructors/assignments
class Base {
public:
    virtual ~Base() = default;
    // No virtual move constructor
};

class Derived : public Base {
    string* data;
public:
    Derived(Derived&& other) noexcept 
        : Base(std::move(other)), data(other.data) {
        other.data = nullptr;
    }
};

// Workaround: virtual clone method
class Cloneable {
public:
    virtual unique_ptr<Cloneable> clone() const = 0;
    virtual unique_ptr<Cloneable> move_clone() = 0;
};

class Concrete : public Cloneable {
    string* data;
public:
    unique_ptr<Cloneable> clone() const override {
        return make_unique<Concrete>(*this);
    }
    
    unique_ptr<Cloneable> move_clone() override {
        return make_unique<Concrete>(std::move(*this));
    }
};
```

### Q48: What's the default move behavior for members?
```cpp
struct S {
    int a;           // Trivially movable (copy)
    string b;        // Has move constructor
    const int c;     // const member prevents move assignment
    int& d;          // Reference member - special handling
};

// Generated move constructor does member-wise move
// For each member:
// - If movable, move it
// - If not, copy it (if possible)
// - If can't copy, delete move

struct ConstMember {
    const int id;
    string name;
    // Move assignment deleted because can't assign to const
    // Move constructor OK (initializes const)
};
```

### Q49: How to conditionally enable move operations?
```cpp
template<typename T>
class Optional {
    alignas(T) unsigned char storage[sizeof(T)];
    bool has_value;
    
public:
    // Enable move if T is movable
    Optional(Optional&& other) noexcept(
        std::is_nothrow_move_constructible_v<T>
    ) requires std::is_move_constructible_v<T> {
        if (other.has_value) {
            new(storage) T(std::move(*other));
            has_value = true;
            other.reset();
        }
    }
    
    // Disable move if T isn't movable
    Optional(Optional&&) = delete;
};

// Using SFINAE or concepts to control move availability
```

### Q50: What's the lifetime of moved-from objects in containers?
```cpp
vector<string> vec = {"a", "b", "c"};

// Moving from element
string s = std::move(vec[1]);  // vec[1] now empty string

// Valid operations on moved-from element:
vec[1] = "new";      // Assignment works
vec[1].clear();      // Clear works
cout << vec[1];      // Outputs empty string

// Invalid:
// vec[1].c_str();     // OK but returns pointer to empty string
// No undefined behavior if implementation follows standard

// Erasing moved-from elements:
vec.erase(vec.begin() + 1);  // Safe, erases empty string

// Better: move entire container
vector<string> vec2 = std::move(vec);  // vec now empty
vec.clear();  // Still OK
```

---

## **Section 5: Copy Elision & Performance Optimization**

### Q51: What are RVO and NRVO and when do they apply?
```cpp
// RVO (Return Value Optimization) - for unnamed temporaries
string make_string_rvo() {
    return string("hello");  // Constructs directly in return location
    // No copy/move happens
}

// NRVO (Named Return Value Optimization) - for named locals
string make_string_nrvo() {
    string result = "hello";
    return result;  // May elide copy/move
    // Compiler optimizes name 'result' directly into return slot
}

// When RVO/NRVO fails:
string no_rvo(bool flag) {
    string a = "hello";
    string b = "world";
    return flag ? a : b;  // Conditional return - NRVO fails
    // Implicit move in C++11 and later
}

// C++17: Guaranteed elision for prvalues
struct NoCopy {
    NoCopy(const NoCopy&) = delete;
    NoCopy() = default;
};

NoCopy factory() {
    return NoCopy();  // C++17: guaranteed to work despite deleted copy
}
```

### Q52: Why is move faster than copy for heap-allocated types?
```cpp
// Copy: Deep copy all elements
class CopyableVector {
    int* data;
    size_t size;
public:
    CopyableVector(const CopyableVector& other) 
        : size(other.size), data(new int[size]) {
        // O(n) loop
        for (size_t i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
    }
};

// Move: Just swap pointers
class MovableVector {
    int* data;
    size_t size;
public:
    MovableVector(MovableVector&& other) noexcept
        : data(other.data), size(other.size) {
        // O(1) pointer assignment
        other.data = nullptr;
        other.size = 0;
    }
};

// Performance: O(n) vs O(1) for large n
```

### Q53: When is move not faster than copy?
```cpp
// 1. Trivially copyable types
int x = 5;
int y = std::move(x);  // Same as copy

// 2. Small objects with SSO (Small String Optimization)
string s1 = "tiny";
string s2 = std::move(s1);  // May still copy if size < SSO threshold

// 3. std::array (no dynamic allocation)
array<int, 1000> arr1;
array<int, 1000> arr2 = std::move(arr1);  // Still copies all elements

// 4. Types with no dynamic resources
struct Point { int x, y, z; };
Point p1{1,2,3};
Point p2 = std::move(p1);  // Just copies ints

// 5. When move constructor isn't noexcept and container needs safety
vector<ThrowingMove> vec;
vec.push_back(ThrowingMove());  // May use copy instead of move
```

### Q54: How does copy elision interact with move operations?
```cpp
struct Tracker {
    Tracker() { cout << "Construct\n"; }
    Tracker(const Tracker&) { cout << "Copy\n"; }
    Tracker(Tracker&&) { cout << "Move\n"; }
    ~Tracker() { cout << "Destroy\n"; }
};

Tracker create() {
    Tracker t;
    return t;  // May do: Construct, Move, Destroy (if no NRVO)
    // With NRVO: Construct, Destroy only
}

Tracker create_rvo() {
    return Tracker();  // Guaranteed: Construct, Destroy
}

int main() {
    cout << "Without optimization:\n";
    Tracker t1 = create();  // Construct, [Move], Destroy, Destroy
    
    cout << "\nWith RVO:\n";
    Tracker t2 = create_rvo();  // Construct, Destroy (only 2 operations)
}
```

### Q55: What optimization opportunities does move semantics enable?
```cpp
// 1. Swapping without temporary
vector<int> a(1000000), b(1000000);
// Old way: copy to temporary
auto temp = a; a = b; b = temp;  // O(n) copies
// New way: move
a = std::move(b);  // O(1) pointer swaps

// 2. Inserting temporaries
vector<string> vec;
vec.push_back(string("expensive"));  // Move (not copy)
vec.emplace_back("expensive");       // Even better: construct in place

// 3. Sorting move-only types
vector<unique_ptr<int>> ptrs;
// Can sort efficiently because unique_ptr is movable

// 4. Returning large objects from functions
vector<int> create_large_vector() {
    vector<int> result(1000000);
    return result;  // NRVO or move
}
```

### Q56: How to measure move vs copy performance?
```cpp
class Benchmark {
    static chrono::nanoseconds measure_copy(const string& s) {
        auto start = chrono::high_resolution_clock::now();
        string copy = s;  // Copy
        auto end = chrono::high_resolution_clock::now();
        return end - start;
    }
    
    static chrono::nanoseconds measure_move(string&& s) {
        auto start = chrono::high_resolution_clock::now();
        string moved = std::move(s);  // Move
        auto end = chrono::high_resolution_clock::now();
        return end - start;
    }
    
public:
    static void compare() {
        string large(1000000, 'x');
        
        auto copy_time = measure_copy(large);
        auto move_time = measure_move(std::move(large));
        
        cout << "Copy: " << copy_time.count() << " ns\n";
        cout << "Move: " << move_time.count() << " ns\n";
        cout << "Speedup: " << (double)copy_time.count() / move_time.count() << "x\n";
        // Typical on modern hardware: Copy ~1,000,000ns, Move ~100ns, Speedup ~10,000x
    }
};
```

### Q57: What's the cost of `std::move` on different STL containers?
```cpp
void benchmark_containers() {
    // vector: O(1) move
    vector<int> v1(1000000);
    vector<int> v2 = std::move(v1);  // ~50ns
    
    // deque: O(1) move
    deque<int> d1(1000000);
    deque<int> d2 = std::move(d1);   // ~50ns
    
    // list: O(1) move (just swaps head/tail pointers)
    list<int> l1(1000000);
    list<int> l2 = std::move(l1);    // ~50ns
    
    // map/set/unordered: O(1) move
    map<int,int> m1;
    // fill m1...
    map<int,int> m2 = std::move(m1); // ~50ns
    
    // array: O(n) copy (no move benefit)
    array<int, 1000000> a1;
    array<int, 1000000> a2 = std::move(a1);  // Copies all elements
    
    // string: O(1) if heap-allocated, O(SSO) if small
    string s1(1000000, 'x');
    string s2 = std::move(s1);  // ~50ns
}
```

### Q58: How to write move-aware code for maximum performance?
```cpp
class OptimizedBuffer {
    unique_ptr<char[]> data;
    size_t size;
    
public:
    // Pass by value for sink parameters
    void take_ownership(string s) {  // Copy or move into parameter
        data = make_unique<char[]>(s.size());
        size = s.size();
        memcpy(data.get(), s.c_str(), size);
    }
    
    // Return by value (RVO will optimize)
    static OptimizedBuffer create(size_t sz) {
        OptimizedBuffer result;
        result.data = make_unique<char[]>(sz);
        result.size = sz;
        return result;  // NRVO
    }
    
    // Use forwarding references for generic code
    template<typename T>
    void assign(T&& value) {
        data = std::forward<T>(value);  // Perfect forwarding
    }
    
    // Always provide noexcept move operations
    OptimizedBuffer(OptimizedBuffer&& other) noexcept = default;
    OptimizedBuffer& operator=(OptimizedBuffer&& other) noexcept = default;
    
    // For container operations
    void swap(OptimizedBuffer& other) noexcept {
        data.swap(other.data);
        std::swap(size, other.size);
    }
};
```

### Q59: What are common anti-patterns that prevent move optimization?
```cpp
// 1. Unnecessary std::move in return
string bad_return() {
    string s = "hello";
    return std::move(s);  // Prevents NRVO
}

// 2. std::move on const objects
void bad_move_const(const string& s) {
    string s2 = std::move(s);  // Copies, doesn't move
}

// 3. Using moved-from object
void use_after_move() {
    vector<int> v = {1,2,3};
    auto w = std::move(v);
    v.push_back(4);  // Dangerous: v is in unspecified state
}

// 4. Not using emplace_back
void no_emplace() {
    vector<pair<int,int>> vec;
    vec.push_back({1,2});  // Creates temporary, then moves
    vec.emplace_back(1,2); // Better: constructs in-place
}

// 5. Returning std::move of local in conditional
string bad_conditional(bool flag) {
    string a = "hello", b = "world";
    return std::move(flag ? a : b);  // Prevents RVO
}
```

### Q60: Ultimate performance comparison: Copy vs Move vs RVO
```cpp
struct Heavy {
    static constexpr int SIZE = 1000000;
    int* data;
    
    Heavy() : data(new int[SIZE]) {
        fill(data, data + SIZE, 42);
    }
    
    Heavy(const Heavy& other) : data(new int[SIZE]) {
        copy(other.data, other.data + SIZE, data);
        cout << "Copy\n";
    }
    
    Heavy(Heavy&& other) noexcept : data(other.data) {
        other.data = nullptr;
        cout << "Move\n";
    }
    
    ~Heavy() { delete[] data; }
};

Heavy factory_rvo() {
    return Heavy();  // RVO: No copy/move
}

Heavy factory_no_rvo() {
    Heavy h;
    return h;  // NRVO usually elides
}

Heavy factory_forced_move() {
    Heavy h;
    return std::move(h);  // Forces move (slower than NRVO)
}

int main() {
    cout << "RVO: ";
    Heavy h1 = factory_rvo();      // Output: (nothing)
    
    cout << "NRVO: ";
    Heavy h2 = factory_no_rvo();   // Output: (nothing if optimized)
    
    cout << "Forced move: ";
    Heavy h3 = factory_forced_move(); // Output: Move
    
    cout << "Explicit move: ";
    Heavy h4;
    Heavy h5 = std::move(h4);      // Output: Move
    
    cout << "Copy: ";
    Heavy h6;
    Heavy h7 = h6;                 // Output: Copy
}
```

---

## **Quick Reference Summary**

| Concept | Key Point | Example |
|---------|-----------|---------|
| **Rvalue Reference** | Binds to temporaries only | `int&& r = 42;` |
| **`std::move`** | Cast to rvalue (doesn't move) | `std::move(lvalue)` |
| **`std::forward`** | Preserves value category | `std::forward<T>(arg)` |
| **Move Constructor** | Steals resources (O(1)) | `T(T&& other) noexcept` |
| **Perfect Forwarding** | Template with `T&&` + `std::forward` | `template<typename T> void f(T&& arg)` |
| **Copy Elision** | Eliminates copies/moves | `return T();` |
| **Rule of Five** | Define all or none | Copy, move, dtor |
| **`noexcept` Move** | Enables optimized containers | Operations that don't throw |

**Best Practices:**
1. Use `std::move` only when you're done with the object
2. Don't `std::move` return values (prevents RVO)
3. Mark move operations `noexcept` when possible
4. Use perfect forwarding for generic code
5. Prefer pass-by-value for sink parameters
6. Follow the Rule of Zero or Rule of Five
7. Don't use moved-from objects except to reassign/destroy