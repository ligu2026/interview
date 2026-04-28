Here's a comprehensive list of 100 tricky C++ interview questions with answers and code examples:

## 1-10: Memory Management & Pointers

### 1. What's the difference between `new` and `malloc()`?
```cpp
// new: calls constructor, type-safe, returns proper type
MyClass* obj1 = new MyClass(10);  // Constructor called

// malloc: doesn't call constructor, returns void*, needs cast
MyClass* obj2 = (MyClass*)malloc(sizeof(MyClass)); // No constructor
// Must use placement new to initialize
new(obj2) MyClass(10);

// Cleanup
delete obj1;    // Calls destructor
obj2->~MyClass(); // Manual destructor call
free(obj2);
```

### 2. What happens when you delete a pointer twice?
```cpp
int* p = new int(5);
delete p;
// delete p; // UNDEFINED BEHAVIOR - crash or corruption

// Best practice
delete p;
p = nullptr;  // Safe to delete nullptr again
delete p;     // No-op, safe
```

### 3. Smart pointers: unique_ptr vs shared_ptr vs weak_ptr
```cpp
#include <memory>

// unique_ptr: exclusive ownership, no overhead
std::unique_ptr<int> uptr = std::make_unique<int>(42);
// std::unique_ptr<int> uptr2 = uptr; // ERROR: can't copy
std::unique_ptr<int> uptr2 = std::move(uptr); // OK: transfer ownership

// shared_ptr: shared ownership, reference counting overhead
std::shared_ptr<int> sptr1 = std::make_shared<int>(100);
std::shared_ptr<int> sptr2 = sptr1; // Both point to same int
// Reference count = 2

// weak_ptr: doesn't increase reference count, prevents circular references
std::weak_ptr<int> wptr = sptr1;
if (auto locked = wptr.lock()) { // Check if still valid
    std::cout << *locked << std::endl;
}
```

### 4. Circular reference problem with shared_ptr
```cpp
struct B; // Forward declaration

struct A {
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

struct B {
    std::weak_ptr<A> a_ptr;  // Use weak_ptr to break cycle
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    a->b_ptr = b;
    b->a_ptr = a;  // Without weak_ptr, memory leak!
    return 0;  // Both destroyed properly
}
```

### 5. Memory leak example and prevention
```cpp
class ResourceLeak {
    int* data;
public:
    ResourceLeak() : data(new int[1000]) {}
    ~ResourceLeak() { delete[] data; }
    // Missing copy constructor and assignment operator!
    // Causes double delete when copied
};

// RAII fix
class ResourceSafe {
    std::vector<int> data;  // Automatically manages memory
public:
    ResourceSafe() : data(1000) {}
    // Rule of Zero: No need for destructor/copy/move
};
```

## 11-20: Object-Oriented Programming

### 11. Virtual destructor importance
```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base destructor\n"; }
};

class Derived : public Base {
    int* data;
public:
    Derived() : data(new int[100]) {}
    ~Derived() { 
        delete[] data;
        std::cout << "Derived destructor\n"; 
    }
};

// Without virtual destructor in Base:
Base* ptr = new Derived();
delete ptr;  // Only calls Base destructor - MEMORY LEAK!
```

### 12. Diamond problem and virtual inheritance
```cpp
class Animal {
public:
    int age;
    virtual void eat() {}
};

class Mammal : virtual public Animal {};  // Virtual inheritance
class Bird : virtual public Animal {};

class Bat : public Mammal, public Bird {
public:
    void eat() override { /* ... */ }
};

Bat bat;
bat.age = 5;  // No ambiguity - single copy of age
```

### 13. Pure virtual function with implementation
```cpp
class Abstract {
public:
    virtual void pure_virtual() = 0;  // Pure virtual
};

void Abstract::pure_virtual() {  // Can have implementation!
    std::cout << "Default implementation\n";
}

class Concrete : public Abstract {
public:
    void pure_virtual() override {
        Abstract::pure_virtual();  // Call base implementation
        std::cout << "Concrete implementation\n";
    }
};
```

### 14. Final and override keywords
```cpp
class Base {
public:
    virtual void func1() {}
    virtual void func2() const {}
};

class Derived : public Base {
public:
    void func1() override {}  // Compiler checks if actually overriding
    // void func2() override {} // ERROR: different const qualifier
    void func2() const override {} // Correct
};

class Further : public Derived {
public:
    // void func1() override {} // ERROR if func1 was final
};

class FinalClass final {  // Cannot be inherited
};
```

## 21-30: Templates

### 21. Template specialization and SFINAE
```cpp
// Primary template
template<typename T>
struct TypeChecker {
    static void print() { std::cout << "General type\n"; }
};

// Specialization for pointers
template<typename T>
struct TypeChecker<T*> {
    static void print() { std::cout << "Pointer type\n"; }
};

// SFINAE: Substitution Failure Is Not An Error
template<typename T>
typename T::value_type getValue(const T& container) {
    return container.front();
}

// If T doesn't have value_type, compilation doesn't fail
// It just removes this overload from consideration
```

### 22. Variadic templates and fold expressions
```cpp
// C++17 fold expressions
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // Unary right fold
}

template<typename... Args>
void printAll(Args... args) {
    (std::cout << ... << args);  // Binary left fold
}

// Recursive variadic template
template<typename T>
T sum_recursive(T t) { return t; }

template<typename T, typename... Args>
T sum_recursive(T first, Args... rest) {
    return first + sum_recursive(rest...);
}
```

### 23. CRTP (Curiously Recurring Template Pattern)
```cpp
template<typename Derived>
class Counter {
    static int count;
public:
    Counter() { count++; }
    ~Counter() { count--; }
    static int getCount() { return count; }
};

template<typename Derived>
int Counter<Derived>::count = 0;

class MyClass : public Counter<MyClass> {};
class YourClass : public Counter<YourClass> {};

// Each derived class gets its own counter
MyClass a, b;
YourClass c;
std::cout << MyClass::getCount();   // 2
std::cout << YourClass::getCount(); // 1
```

## 31-40: Move Semantics

### 31. Move constructor and move assignment
```cpp
class Resource {
    int* data;
    size_t size;
public:
    // Move constructor
    Resource(Resource&& other) noexcept 
        : data(std::exchange(other.data, nullptr))
        , size(std::exchange(other.size, 0)) {}
    
    // Move assignment
    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = std::exchange(other.data, nullptr);
            size = std::exchange(other.size, 0);
        }
        return *this;
    }
};
```

### 32. Perfect forwarding and universal references
```cpp
template<typename T>
void wrapper(T&& arg) {  // Universal reference
    process(std::forward<T>(arg));  // Perfect forwarding
}

// Works for both lvalues and rvalues
int x = 5;
wrapper(x);       // Calls process(int&)
wrapper(42);      // Calls process(int&&)
wrapper(std::move(x)); // Calls process(int&&)
```

## 41-50: Modern C++ Features

### 41. Lambda captures and mutable
```cpp
int x = 10;
int y = 20;

// Capture by value (copy)
auto by_val = [x]() { return x; };  // x is const
// auto by_val2 = [x]() { x++; };   // ERROR: x is read-only

// Mutable lambda
auto mutable_lambda = [x]() mutable { 
    x++;  // OK, but modifies copy only
    return x; 
};

// Capture by reference
auto by_ref = [&x]() { x++; };  // Modifies original

// Generalized capture (C++14)
auto move_capture = [p = std::make_unique<int>(5)]() {
    return *p;  // unique_ptr captured by move
};
```

### 42. constexpr and compile-time evaluation
```cpp
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}

// Computed at compile time
constexpr int fact_5 = factorial(5);  // 120
static_assert(fact_5 == 120, "Factorial must be 120");

// Can also be used at runtime
int runtime_val = factorial(user_input);

// constexpr if (C++17)
template<typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>) {
        return *t;  // Only compiled if T is pointer
    } else {
        return t;
    }
}
```

## 51-60: Exception Safety

### 51. RAII and exception safety guarantees
```cpp
class SafeFile {
    FILE* file;
public:
    SafeFile(const char* name) : file(fopen(name, "r")) {
        if (!file) throw std::runtime_error("Cannot open file");
    }
    
    ~SafeFile() { if (file) fclose(file); }
    
    // No-throw swap for strong exception guarantee
    friend void swap(SafeFile& a, SafeFile& b) noexcept {
        std::swap(a.file, b.file);
    }
    
    SafeFile(const SafeFile&) = delete;
    SafeFile& operator=(const SafeFile&) = delete;
};

// Strong exception safety with copy-and-swap
class SafeArray {
    int* data;
    size_t size;
public:
    SafeArray& operator=(const SafeArray& other) {
        SafeArray temp(other);  // Might throw
        swap(*this, temp);      // No-throw
        return *this;
    }
};
```

## 61-70: Thread Safety

### 61. Atomic operations and memory ordering
```cpp
#include <atomic>
#include <thread>

std::atomic<bool> ready(false);
std::atomic<int> data(0);

void producer() {
    data.store(42, std::memory_order_relaxed);
    ready.store(true, std::memory_order_release);  // Sync point
}

void consumer() {
    while (!ready.load(std::memory_order_acquire)); // Sync point
    int val = data.load(std::memory_order_relaxed);
    // Guaranteed to see data = 42
}
```

### 62. Lock-free programming with compare_exchange
```cpp
class LockFreeStack {
    struct Node {
        int data;
        Node* next;
        Node(int d) : data(d), next(nullptr) {}
    };
    
    std::atomic<Node*> head{nullptr};
    
public:
    void push(int value) {
        Node* new_node = new Node(value);
        new_node->next = head.load();
        
        while (!head.compare_exchange_weak(
                   new_node->next,  // Expected
                   new_node         // Desired
               )) {
            // CAS failed, new_node->next is updated to current head
        }
    }
    
    bool pop(int& value) {
        Node* old_head = head.load();
        
        while (old_head && 
               !head.compare_exchange_weak(old_head, old_head->next)) {
            // Retry on failure
        }
        
        if (old_head) {
            value = old_head->data;
            delete old_head;
            return true;
        }
        return false;
    }
};
```

## 71-80: Design Patterns in C++

### 71. Singleton with thread safety (C++11 style)
```cpp
class Singleton {
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
public:
    static Singleton& getInstance() {
        static Singleton instance;  // Thread-safe in C++11
        return instance;
    }
};
```

### 72. Factory pattern with smart pointers
```cpp
class Product {
public:
    virtual void use() = 0;
    virtual ~Product() = default;
};

class ConcreteProductA : public Product {
public:
    void use() override { std::cout << "Using A\n"; }
};

class ConcreteProductB : public Product {
public:
    void use() override { std::cout << "Using B\n"; }
};

class Factory {
public:
    enum class Type { A, B };
    
    static std::unique_ptr<Product> create(Type type) {
        switch (type) {
            case Type::A: return std::make_unique<ConcreteProductA>();
            case Type::B: return std::make_unique<ConcreteProductB>();
        }
        return nullptr;
    }
};
```

## 81-90: Tricky Code Snippets

### 81. Most Vexing Parse
```cpp
class Timer {
public:
    Timer() {}
};

// This is a function declaration, not object creation!
Timer t();  // Function returning Timer

// Solutions:
Timer t1;           // Default construction
Timer t2{};         // Uniform initialization (C++11)
```

### 82. Dangling reference
```cpp
const std::string& getString() {
    return "temporary";  // UNDEFINED BEHAVIOR!
    // const char* converted to temporary std::string
    // temporary destroyed when function returns
}

const std::string& good() {
    static std::string str = "safe";
    return str;  // OK: static variable exists globally
}
```

### 83. Order of evaluation quirks
```cpp
int x = 0;
int f1() { x += 1; return x; }
int f2() { x += 2; return x; }

// Unspecified order of evaluation (pre-C++17)
int result = f1() + f2();  
// Could be 1+3=4 or 3+1=4, but x could be 1 or 3 after

// C++17: evaluation order specified
// Left-to-right: f1() then f2()
```

## 91-100: Advanced Concepts

### 91. SFINAE with enable_if
```cpp
template<typename T>
typename std::enable_if<std::is_integral<T>::value, bool>::type
is_odd(T value) {
    return value % 2 != 0;
}

template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, bool>::type
is_odd(T value) {
    return std::fmod(value, 2.0) != 0.0;
}

// C++14/17 cleaner syntax
template<typename T>
std::enable_if_t<std::is_integral_v<T>, bool>
is_odd_v2(T value) {
    return value % 2 != 0;
}
```

### 92. Custom allocators and memory pools
```cpp
template<typename T, size_t PoolSize = 1024>
class PoolAllocator {
    struct Block {
        typename std::aligned_storage<sizeof(T), alignof(T)>::type data;
        Block* next;
    };
    
    Block pool[PoolSize];
    Block* free_list;
    
public:
    using value_type = T;
    
    PoolAllocator() noexcept {
        for (size_t i = 0; i < PoolSize - 1; ++i) {
            pool[i].next = &pool[i + 1];
        }
        pool[PoolSize - 1].next = nullptr;
        free_list = &pool[0];
    }
    
    T* allocate(size_t n) {
        if (n != 1 || !free_list) throw std::bad_alloc();
        Block* block = free_list;
        free_list = free_list->next;
        return reinterpret_cast<T*>(block);
    }
    
    void deallocate(T* p, size_t n) {
        Block* block = reinterpret_cast<Block*>(p);
        block->next = free_list;
        free_list = block;
    }
};
```

### 93. Type erasure example
```cpp
class FunctionWrapper {
    struct Concept {
        virtual void call() = 0;
        virtual ~Concept() = default;
    };
    
    template<typename T>
    struct Model : Concept {
        T func;
        Model(T f) : func(std::move(f)) {}
        void call() override { func(); }
    };
    
    std::unique_ptr<Concept> ptr;
    
public:
    template<typename T>
    FunctionWrapper(T f) : ptr(std::make_unique<Model<T>>(std::move(f))) {}
    
    void operator()() { ptr->call(); }
};

// Usage
FunctionWrapper f = []() { std::cout << "Hello Type Erasure\n"; };
f();  // Works with any callable
```

### 94. Structured bindings and tuple-like types
```cpp
struct Point3D {
    int x, y, z;
    
    // Enable structured bindings
    template<size_t I>
    auto& get() {
        if constexpr (I == 0) return x;
        else if constexpr (I == 1) return y;
        else if constexpr (I == 2) return z;
    }
};

namespace std {
    template<> struct tuple_size<Point3D> : std::integral_constant<size_t, 3> {};
    template<size_t I> struct tuple_element<I, Point3D> { using type = int; };
}

// Usage
Point3D p{1, 2, 3};
auto [a, b, c] = p;  // Structured binding
```

### 95. Volatile and atomic interaction
```cpp
#include <atomic>

// Wrong: volatile doesn't guarantee atomicity
struct Wrong {
    volatile int flag;  // Not atomic, no memory ordering guarantees
};

// Correct: use std::atomic
struct Correct {
    std::atomic<int> flag;  // Atomic with memory ordering
};

void signal_handler() {
    static Correct signal_flag;  // Safe for signal handlers
    signal_flag.flag.store(1, std::memory_order_release);
}
```

### 96. The PImpl idiom (Compiler firewall)
```cpp
// Header file - no implementation details visible
class Widget {
public:
    Widget();
    ~Widget();
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;
    
    void doSomething();
    
private:
    class Impl;  // Forward declaration only
    std::unique_ptr<Impl> pImpl;
};

// Implementation file - hidden from clients
class Widget::Impl {
    std::string name;
    std::vector<int> data;
public:
    void doSomething() { /* complex implementation */ }
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() = default;  // Must be defined after Impl definition
Widget::Widget(Widget&&) = default;
```

### 97. Expression templates (Delay evaluation)
```cpp
template<typename L, typename R>
struct VectorSum {
    const L& left;
    const R& right;
    
    VectorSum(const L& l, const R& r) : left(l), right(r) {}
    
    double operator[](size_t i) const {
        return left[i] + right[i];
    }
};

class Vector {
    std::vector<double> data;
public:
    Vector(std::initializer_list<double> il) : data(il) {}
    
    size_t size() const { return data.size(); }
    double operator[](size_t i) const { return data[i]; }
    
    template<typename T>
    Vector& operator=(const T& expression) {
        data.resize(expression.size());
        for (size_t i = 0; i < data.size(); ++i) {
            data[i] = expression[i];  // Calculate on assignment
        }
        return *this;
    }
};

template<typename L, typename R>
auto operator+(const L& left, const R& right) {
    return VectorSum<L, R>(left, right);
}

// Usage: No temporary vectors created!
Vector a{1, 2, 3}, b{4, 5, 6}, c{7, 8, 9};
Vector result = a + b + c;  // Single loop, no temporaries
```

### 98. Compile-time state machine
```cpp
template<typename State>
struct TransitionTo {
    using type = State;
};

class Idle;
class Running;
class Paused;

class Idle {
public:
    using on_start = TransitionTo<Running>;
};

class Running {
public:
    using on_pause = TransitionTo<Paused>;
    using on_stop = TransitionTo<Idle>;
};

class Paused {
public:
    using on_resume = TransitionTo<Running>;
    using on_stop = TransitionTo<Idle>;
};

template<typename Current>
class StateMachine {
    Current state;
public:
    template<typename Event>
    void handle(const Event& event) {
        using NextState = typename Current::template Event::type;
        static_assert(!std::is_same_v<NextState, void>, "Invalid transition");
        transition(NextState{});
    }
};
```

### 99. Template metaprogramming: Type lists
```cpp
template<typename... Types>
struct TypeList {};

// Length
template<typename List>
struct Length;

template<typename... Types>
struct Length<TypeList<Types...>> {
    static constexpr size_t value = sizeof...(Types);
};

// Index
template<typename List, size_t Index>
struct TypeAt;

template<typename T, typename... Types>
struct TypeAt<TypeList<T, Types...>, 0> {
    using type = T;
};

template<typename T, typename... Types, size_t Index>
struct TypeAt<TypeList<T, Types...>, Index> {
    using type = typename TypeAt<TypeList<Types...>, Index - 1>::type;
};

// Usage
using MyTypes = TypeList<int, double, std::string>;
static_assert(Length<MyTypes>::value == 3);
using SecondType = TypeAt<MyTypes, 1>::type;  // double
```

### 100. `std::launder` and object lifetime
```cpp
#include <new>

struct X {
    const int n;
    X(int val) : n(val) {}
};

void tricky_object_reuse() {
    X x1{42};
    X x2{100};
    
    // End lifetime of x1, start lifetime of x2 at same address
    new (&x1) X{200};
    
    // x1.n might still be 42 (compiler optimization of const)
    // std::launder prevents UB
    const int val = std::launder(&x1)->n;  // Guaranteed to be 200
}

// More realistic use case
struct alignas(64) AlignedBuffer {
    int data[16];
};

AlignedBuffer buffer;
// Use placement new for aligned objects
auto ptr = ::new (&buffer) std::string("Hello");
// std::launder might be needed if you access through original pointer
```

**Key takeaways for C++ interviews:**

1. **Understand move semantics deeply** - Most modern C++ interviews focus heavily on this
2. **RAII and exception safety** - Show understanding of resource management
3. **Template metaprogramming** - Especially SFINAE and concepts
4. **Smart pointers** - When to use each type
5. **Memory model** - Understanding happens-before relationships
6. **Undefined behavior** - Ability to spot potential UB
7. **Compile-time programming** - constexpr, templates, concepts

These questions cover the most challenging aspects of C++ that interviewers often test. The key is not just memorizing answers but understanding the underlying principles.