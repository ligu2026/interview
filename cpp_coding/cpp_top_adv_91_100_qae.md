# 🎯 Advanced C++ OOP Questions - Complete Answers & Examples

Here are **in-depth answers** for advanced C++ OOP questions (Questions 91-100 with extra challenging variants):

---

## 91. **Virtual Functions & Polymorphism Deep Dive**

### Question: Explain vtable and vptr. How does dynamic dispatch work?

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    virtual void func1() { cout << "Base::func1" << endl; }
    virtual void func2() { cout << "Base::func2" endl; }
    void func3() { cout << "Base::func3" << endl; }  // Non-virtual
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void func1() override { cout << "Derived::func1" << endl; }
    virtual void func4() { cout << "Derived::func4" << endl; }
};

// Demonstrating vtable behavior
class VTableExplorer {
public:
    virtual void show() { cout << "VTable demo" << endl; }
    virtual ~VTableExplorer() = default;
};

int main() {
    // VTable layout demonstration
    Base* ptr1 = new Base();
    Base* ptr2 = new Derived();
    
    ptr1->func1();  // Calls Base::func1 (index 0 in Base vtable)
    ptr2->func1();  // Calls Derived::func1 (overridden slot)
    ptr2->func2();  // Calls Base::func2 (inherited, same vtable slot)
    ptr2->func3();  // Called statically (not in vtable)
    
    // Size includes vptr (8 bytes on 64-bit)
    cout << "Size of Base: " << sizeof(Base) << endl;      // 8 (vptr only)
    cout << "Size of Derived: " << sizeof(Derived) << endl; // 8 (vptr only)
    
    delete ptr1;
    delete ptr2;
    return 0;
}
```

### Advanced: Multiple Inheritance & Diamond Problem
```cpp
class GrandParent {
public:
    virtual void common() { cout << "GrandParent" << endl; }
    virtual ~GrandParent() = default;
};

class Parent1 : virtual public GrandParent {  // Virtual inheritance
public:
    void common() override { cout << "Parent1" << endl; }
};

class Parent2 : virtual public GrandParent {
public:
    void common() override { cout << "Parent2" << endl; }
};

class Child : public Parent1, public Parent2 {
public:
    void common() override { cout << "Child" << endl; }
};

// Without virtual inheritance, this would have two copies of GrandParent
// With virtual inheritance, single shared instance
```

---

## 92. **Copy Constructor vs Assignment Operator Deep Dive**

### Question: When is each called? Implement rule of three/five.

```cpp
class StringBuffer {
private:
    char* data;
    size_t size;
    
public:
    // Constructor
    StringBuffer(const char* str = "") {
        size = strlen(str);
        data = new char[size + 1];
        strcpy(data, str);
        cout << "Constructed: " << data << endl;
    }
    
    // Copy Constructor - Called when:
    // 1. StringBuffer s2 = s1;  (initialization)
    // 2. Passing by value
    // 3. Returning by value
    StringBuffer(const StringBuffer& other) {
        size = other.size;
        data = new char[size + 1];
        strcpy(data, other.data);
        cout << "Copy constructed: " << data << endl;
    }
    
    // Assignment Operator - Called when:
    // StringBuffer s1, s2; s1 = s2; (existing objects)
    StringBuffer& operator=(const StringBuffer& other) {
        if (this != &other) {
            cout << "Assigning: " << data << " = " << other.data << endl;
            delete[] data;
            size = other.size;
            data = new char[size + 1];
            strcpy(data, other.data);
        }
        return *this;
    }
    
    // Move Constructor
    StringBuffer(StringBuffer&& other) noexcept 
        : data(nullptr), size(0) {
        cout << "Move constructed from: " << other.data << endl;
        data = other.data;
        size = other.size;
        other.data = nullptr;
        other.size = 0;
    }
    
    // Move Assignment
    StringBuffer& operator=(StringBuffer&& other) noexcept {
        if (this != &other) {
            cout << "Move assigned: " << data << " -> " << other.data << endl;
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
    
    // Destructor
    ~StringBuffer() {
        cout << "Destroyed: " << (data ? data : "null") << endl;
        delete[] data;
    }
    
    void print() const { cout << (data ? data : "null") << endl; }
};

// Demonstration
StringBuffer createBuffer() {
    StringBuffer temp("Temporary");
    return temp;  // Move constructor (or RVO)
}

int main() {
    StringBuffer s1("Hello");           // Constructor
    StringBuffer s2 = s1;               // Copy constructor
    StringBuffer s3("World");
    s3 = s1;                            // Assignment operator
    
    StringBuffer s4 = move(s1);         // Move constructor
    s1.print();                         // null (moved-from state)
    
    StringBuffer s5 = createBuffer();   // Move or RVO
    return 0;
}
```

---

## 93. **Smart Pointers Deep Dive**

### Question: Implement custom smart pointer with reference counting.

```cpp
#include <atomic>
#include <iostream>
using namespace std;

template<typename T>
class SharedPtr {
private:
    T* ptr;
    atomic<int>* refCount;
    
public:
    // Constructor
    explicit SharedPtr(T* p = nullptr) : ptr(p), refCount(nullptr) {
        if (ptr) {
            refCount = new atomic<int>(1);
            cout << "SharedPtr created, refCount = 1" << endl;
        }
    }
    
    // Copy constructor
    SharedPtr(const SharedPtr& other) : ptr(other.ptr), refCount(other.refCount) {
        if (refCount) {
            (*refCount)++;
            cout << "SharedPtr copied, refCount = " << *refCount << endl;
        }
    }
    
    // Move constructor
    SharedPtr(SharedPtr&& other) noexcept 
        : ptr(other.ptr), refCount(other.refCount) {
        other.ptr = nullptr;
        other.refCount = nullptr;
        cout << "SharedPtr moved" << endl;
    }
    
    // Assignment operator
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {
            release();
            ptr = other.ptr;
            refCount = other.refCount;
            if (refCount) (*refCount)++;
            cout << "SharedPtr assigned, refCount = " << *refCount << endl;
        }
        return *this;
    }
    
    // Dereference operators
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    
    // Get raw pointer
    T* get() const { return ptr; }
    
    // Get reference count
    int use_count() const { return refCount ? *refCount : 0; }
    
    // Reset
    void reset(T* p = nullptr) {
        release();
        ptr = p;
        if (ptr) {
            refCount = new atomic<int>(1);
        }
    }
    
    // Release reference
    void release() {
        if (refCount) {
            (*refCount)--;
            cout << "Releasing, refCount = " << *refCount << endl;
            if (*refCount == 0) {
                cout << "Deleting resource: " << (void*)ptr << endl;
                delete ptr;
                delete refCount;
            }
            ptr = nullptr;
            refCount = nullptr;
        }
    }
    
    ~SharedPtr() {
        release();
    }
};

// Custom deleter support
template<typename T, typename Deleter>
class UniquePtr {
private:
    T* ptr;
    Deleter deleter;
    
public:
    explicit UniquePtr(T* p = nullptr, Deleter d = Deleter()) 
        : ptr(p), deleter(d) {}
    
    ~UniquePtr() {
        if (ptr) deleter(ptr);
    }
    
    // No copy
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
    
    // Move allowed
    UniquePtr(UniquePtr&& other) noexcept 
        : ptr(other.ptr), deleter(move(other.deleter)) {
        other.ptr = nullptr;
    }
    
    UniquePtr& operator=(UniquePtr&& other) noexcept {
        if (this != &other) {
            if (ptr) deleter(ptr);
            ptr = other.ptr;
            deleter = move(other.deleter);
            other.ptr = nullptr;
        }
        return *this;
    }
    
    T* operator->() { return ptr; }
    T& operator*() { return *ptr; }
    
    T* release() {
        T* temp = ptr;
        ptr = nullptr;
        return temp;
    }
    
    void reset(T* p = nullptr) {
        if (ptr) deleter(ptr);
        ptr = p;
    }
};

// Usage
struct FileDeleter {
    void operator()(FILE* f) const {
        if (f) {
            fclose(f);
            cout << "File closed" << endl;
        }
    }
};

int main() {
    // Custom SharedPtr
    SharedPtr<int> sp1(new int(42));
    SharedPtr<int> sp2 = sp1;  // Copy
    
    cout << "Value: " << *sp1 << ", refCount: " << sp1.use_count() << endl;
    
    // Custom UniquePtr with deleter
    UniquePtr<FILE, FileDeleter> file(fopen("test.txt", "w"));
    // File automatically closed when file goes out of scope
    
    return 0;
}
```

---

## 94. **Move Semantics & Perfect Forwarding**

### Question: Implement perfect forwarding in a factory function.

```cpp
#include <iostream>
#include <utility>
#include <tuple>
#include <memory>
using namespace std;

// Simple class to demonstrate perfect forwarding
class Widget {
private:
    int id;
    string name;
    double value;
    
public:
    // Multiple constructors
    Widget() : id(0), name("default"), value(0.0) {
        cout << "Default constructor" << endl;
    }
    
    Widget(int i) : id(i), name(""), value(0.0) {
        cout << "Constructor: int " << i << endl;
    }
    
    Widget(int i, string n) : id(i), name(n), value(0.0) {
        cout << "Constructor: int, string " << i << ", " << n << endl;
    }
    
    Widget(int i, string n, double v) : id(i), name(n), value(v) {
        cout << "Constructor: int, string, double" << endl;
    }
    
    // Move constructor
    Widget(Widget&& other) noexcept 
        : id(other.id), name(move(other.name)), value(other.value) {
        cout << "Move constructor" << endl;
    }
    
    // Copy constructor
    Widget(const Widget& other) 
        : id(other.id), name(other.name), value(other.value) {
        cout << "Copy constructor" << endl;
    }
    
    void print() const {
        cout << "Widget[" << id << ", " << name << ", " << value << "]" << endl;
    }
};

// Perfect forwarding factory function
template<typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T>(new T(forward<Args>(args)...));
}

// Variadic template with perfect forwarding
class Factory {
public:
    template<typename T, typename... Args>
    static T* create(Args&&... args) {
        cout << "Creating object with perfect forwarding" << endl;
        return new T(forward<Args>(args)...);
    }
    
    template<typename T, typename... Args>
    static T createStack(Args&&... args) {
        return T(forward<Args>(args)...);
    }
};

// Demonstrate forwarding of lvalues and rvalues
void processValue(int& x) { cout << "Lvalue reference: " << x << endl; }
void processValue(int&& x) { cout << "Rvalue reference: " << x << endl; }

template<typename T>
void forwardExample(T&& arg) {
    // arg is universal reference
    processValue(forward<T>(arg));  // Preserves value category
    // processValue(arg);  // Would always treat as lvalue
}

// Real-world example: vector emplace_back implementation
template<typename T>
class MyVector {
private:
    T* data;
    size_t capacity;
    size_t size;
    
public:
    template<typename... Args>
    void emplace_back(Args&&... args) {
        if (size >= capacity) grow();
        new (&data[size]) T(forward<Args>(args)...);
        size++;
    }
    
private:
    void grow() { /* reallocation logic */ }
};

// Move-only type
class MoveOnly {
private:
    unique_ptr<int> ptr;
    
public:
    MoveOnly(int value) : ptr(make_unique<int>(value)) {}
    
    MoveOnly(const MoveOnly&) = delete;
    MoveOnly& operator=(const MoveOnly&) = delete;
    
    MoveOnly(MoveOnly&& other) noexcept : ptr(move(other.ptr)) {}
    MoveOnly& operator=(MoveOnly&& other) noexcept {
        if (this != &other) ptr = move(other.ptr);
        return *this;
    }
    
    int get() const { return *ptr; }
};

int main() {
    // Perfect forwarding examples
    auto w1 = make_unique<Widget>(42, "Hello", 3.14);
    auto w2 = make_unique<Widget>(100);
    auto w3 = make_unique<Widget>();
    
    // Forwarding lvalues and rvalues
    int x = 10;
    forwardExample(x);   // Lvalue
    forwardExample(20);  // Rvalue
    
    // Emplace back demonstration
    MyVector<Widget> vec;
    vec.emplace_back(1, "First");
    vec.emplace_back(2, "Second", 2.5);
    
    // Move-only type
    MoveOnly m1(100);
    MoveOnly m2 = move(m1);  // Transfer ownership
    cout << "Moved value: " << m2.get() << endl;
    
    return 0;
}
```

---

## 95. **Advanced Template Metaprogramming**

### Question: Implement compile-time factorial, type traits, and SFINAE.

```cpp
#include <iostream>
#include <type_traits>
#include <vector>
using namespace std;

// Compile-time factorial
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// Compile-time Fibonacci
template<int N>
struct Fibonacci {
    static constexpr int value = Fibonacci<N - 1>::value + Fibonacci<N - 2>::value;
};

template<>
struct Fibonacci<0> { static constexpr int value = 0; };

template<>
struct Fibonacci<1> { static constexpr int value = 1; };

// Type traits implementation
template<typename T>
struct IsPointer {
    static constexpr bool value = false;
};

template<typename T>
struct IsPointer<T*> {
    static constexpr bool value = true;
};

template<typename T>
struct RemoveReference {
    using type = T;
};

template<typename T>
struct RemoveReference<T&> {
    using type = T;
};

template<typename T>
struct RemoveReference<T&&> {
    using type = T;
};

// SFINAE: Enable only for integral types
template<typename T>
typename enable_if<is_integral<T>::value, bool>::type
isEven(T value) {
    return value % 2 == 0;
}

// Multiple SFINAE overloads
template<typename T>
typename enable_if<is_floating_point<T>::value, bool>::type
isEven(T value) {
    return false;  // Floating point can't be even
}

// Tag dispatching
template<typename T>
void process(T value, true_type) {
    cout << "Processing integral: " << value << endl;
}

template<typename T>
void process(T value, false_type) {
    cout << "Processing non-integral: " << value << endl;
}

template<typename T>
void dispatch(T value) {
    process(value, is_integral<T>());
}

// Variadic templates
template<typename... Args>
struct TypeList {};

template<typename T, typename... Args>
struct FirstType {
    using type = T;
};

template<typename T, typename... Args>
struct CountTypes {
    static constexpr size_t value = 1 + CountTypes<Args...>::value;
};

template<typename T>
struct CountTypes<T> {
    static constexpr size_t value = 1;
};

// Fold expression examples (C++17)
template<typename... Args>
auto sum(Args... args) {
    return (args + ... + 0);  // Right fold
}

template<typename... Args>
auto product(Args... args) {
    return (args * ... * 1);  // Left fold
}

// Type selection at compile time
template<bool Condition, typename TrueType, typename FalseType>
struct Conditional {
    using type = TrueType;
};

template<typename TrueType, typename FalseType>
struct Conditional<false, TrueType, FalseType> {
    using type = FalseType;
};

// Compile-time array
template<typename T, size_t N>
struct CompileTimeArray {
    T data[N];
    
    constexpr T operator[](size_t index) const { return data[index]; }
    constexpr size_t size() const { return N; }
};

// Constexpr functions (C++11/14)
constexpr int fibonacci(int n) {
    return (n <= 1) ? n : fibonacci(n - 1) + fibonacci(n - 2);
}

constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) result *= i;
    return result;
}

// Type trait to check if type has begin() method
template<typename T>
struct has_begin {
private:
    template<typename U>
    static auto check(int) -> decltype(declval<U>().begin(), true_type());
    
    template<typename>
    static false_type check(...);
    
public:
    static constexpr bool value = decltype(check<T>(0))::value;
};

int main() {
    // Compile-time calculations
    cout << "Factorial<5>::value = " << Factorial<5>::value << endl;  // 120
    cout << "Fibonacci<10>::value = " << Fibonacci<10>::value << endl; // 55
    
    // Type traits
    cout << "IsPointer<int*>::value = " << IsPointer<int*>::value << endl;  // true
    cout << "IsPointer<int>::value = " << IsPointer<int>::value << endl;    // false
    
    // SFINAE
    cout << "isEven(4): " << isEven(4) << endl;  // true
    cout << "isEven(3.5): " << isEven(3.5) << endl;  // false
    
    // Tag dispatching
    dispatch(42);    // Integral
    dispatch(3.14);  // Non-integral
    
    // Variadic templates
    cout << "Sum: " << sum(1, 2, 3, 4, 5) << endl;       // 15
    cout << "Product: " << product(1, 2, 3, 4) << endl;  // 24
    
    // Conditional type
    Conditional<true, int, double>::type x = 42;   // int
    Conditional<false, int, double>::type y = 3.14; // double
    
    // Constexpr
    constexpr int fib10 = fibonacci(10);
    constexpr int fact5 = factorial(5);
    cout << "Constexpr fib(10) = " << fib10 << endl;
    
    // Compile-time array
    constexpr CompileTimeArray<int, 3> arr = {1, 2, 3};
    cout << "Compile-time array size: " << arr.size() << endl;
    
    return 0;
}
```

---

## 96. **Singleton Pattern - Thread-Safe Advanced Implementation**

### Question: Implement thread-safe singleton with double-checked locking and Meyers singleton.

```cpp
#include <iostream>
#include <mutex>
#include <atomic>
#include <memory>
#include <thread>
#include <vector>
using namespace std;

// 1. Meyers Singleton (C++11 thread-safe)
class MeyersSingleton {
private:
    MeyersSingleton() {
        cout << "MeyersSingleton created in thread " 
             << this_thread::get_id() << endl;
    }
    
    ~MeyersSingleton() {
        cout << "MeyersSingleton destroyed" << endl;
    }
    
    MeyersSingleton(const MeyersSingleton&) = delete;
    MeyersSingleton& operator=(const MeyersSingleton&) = delete;
    
public:
    static MeyersSingleton& getInstance() {
        static MeyersSingleton instance;  // Thread-safe in C++11+
        return instance;
    }
    
    void doWork() {
        cout << "MeyersSingleton working in thread " 
             << this_thread::get_id() << endl;
    }
};

// 2. Double-Checked Locking Singleton (with atomic)
class DCLSingleton {
private:
    static atomic<DCLSingleton*> instance;
    static mutex mtx;
    
    DCLSingleton() {
        cout << "DCLSingleton created" << endl;
    }
    
    ~DCLSingleton() = default;
    
public:
    static DCLSingleton* getInstance() {
        DCLSingleton* tmp = instance.load(memory_order_acquire);
        if (tmp == nullptr) {
            lock_guard<mutex> lock(mtx);
            tmp = instance.load(memory_order_relaxed);
            if (tmp == nullptr) {
                tmp = new DCLSingleton();
                instance.store(tmp, memory_order_release);
            }
        }
        return tmp;
    }
    
    void doWork() {
        cout << "DCLSingleton working" << endl;
    }
};

atomic<DCLSingleton*> DCLSingleton::instance(nullptr);
mutex DCLSingleton::mtx;

// 3. Smart pointer singleton
class SmartSingleton {
private:
    static weak_ptr<SmartSingleton> weakInstance;
    static mutex mtx;
    
    SmartSingleton() {
        cout << "SmartSingleton created" << endl;
    }
    
public:
    static shared_ptr<SmartSingleton> getInstance() {
        auto instance = weakInstance.lock();
        if (!instance) {
            lock_guard<mutex> lock(mtx);
            instance = weakInstance.lock();
            if (!instance) {
                instance = shared_ptr<SmartSingleton>(new SmartSingleton());
                weakInstance = instance;
            }
        }
        return instance;
    }
    
    void doWork() {
        cout << "SmartSingleton working" << endl;
    }
};

weak_ptr<SmartSingleton> SmartSingleton::weakInstance;
mutex SmartSingleton::mtx;

// 4. Singleton with automatic cleanup
template<typename T>
class Singleton {
private:
    static T* instance;
    static mutex mtx;
    
    struct Deleter {
        ~Deleter() {
            delete Singleton<T>::instance;
            Singleton<T>::instance = nullptr;
        }
    };
    static Deleter deleter;
    
protected:
    Singleton() = default;
    virtual ~Singleton() = default;
    
public:
    static T& getInstance() {
        if (!instance) {
            lock_guard<mutex> lock(mtx);
            if (!instance) {
                instance = new T();
            }
        }
        return *instance;
    }
};

template<typename T>
T* Singleton<T>::instance = nullptr;

template<typename T>
mutex Singleton<T>::mtx;

// 5. Lazy initialization with call_once
class CallOnceSingleton {
private:
    static unique_ptr<CallOnceSingleton> instance;
    static once_flag initFlag;
    
    CallOnceSingleton() {
        cout << "CallOnceSingleton created" << endl;
    }
    
public:
    static CallOnceSingleton& getInstance() {
        call_once(initFlag, []() {
            instance.reset(new CallOnceSingleton());
        });
        return *instance;
    }
    
    void doWork() {
        cout << "CallOnceSingleton working" << endl;
    }
};

unique_ptr<CallOnceSingleton> CallOnceSingleton::instance(nullptr);
once_flag CallOnceSingleton::initFlag;

// Demonstration with multiple threads
void threadFunction(int id) {
    // All singletons are thread-safe
    MeyersSingleton::getInstance().doWork();
    DCLSingleton::getInstance()->doWork();
    SmartSingleton::getInstance()->doWork();
    Singleton<CallOnceSingleton>::getInstance().doWork();
    CallOnceSingleton::getInstance().doWork();
}

int main() {
    vector<thread> threads;
    
    // Launch multiple threads
    for (int i = 0; i < 5; i++) {
        threads.emplace_back(threadFunction, i);
    }
    
    // Wait for all threads
    for (auto& t : threads) {
        t.join();
    }
    
    // Reference semantics
    MeyersSingleton& ms1 = MeyersSingleton::getInstance();
    MeyersSingleton& ms2 = MeyersSingleton::getInstance();
    cout << "Same instance? " << (&ms1 == &ms2) << endl;
    
    return 0;
}
```

---

## 97. **RAII - Resource Acquisition Is Initialization Advanced**

### Question: Implement RAII wrappers for multiple resource types.

```cpp
#include <iostream>
#include <mutex>
#include <fstream>
#include <cstdio>
#include <memory>
#include <functional>
using namespace std;

// 1. RAII for file handling
class FileRAII {
private:
    FILE* file;
    string filename;
    
public:
    FileRAII(const string& name, const string& mode) 
        : filename(name), file(nullptr) {
        file = fopen(name.c_str(), mode.c_str());
        if (!file) {
            throw runtime_error("Cannot open file: " + name);
        }
        cout << "File opened: " << name << endl;
    }
    
    ~FileRAII() {
        if (file) {
            fclose(file);
            cout << "File closed: " << filename << endl;
        }
    }
    
    // Write to file
    void write(const string& data) {
        if (file) {
            fputs(data.c_str(), file);
        }
    }
    
    // Read line
    string readLine() {
        char buffer[1024];
        if (file && fgets(buffer, sizeof(buffer), file)) {
            return string(buffer);
        }
        return "";
    }
    
    // Move constructor
    FileRAII(FileRAII&& other) noexcept 
        : file(other.file), filename(move(other.filename)) {
        other.file = nullptr;
    }
    
    // No copy
    FileRAII(const FileRAII&) = delete;
    FileRAII& operator=(const FileRAII&) = delete;
};

// 2. RAII for mutex locking
class MutexRAII {
private:
    mutex& mtx;
    bool locked;
    
public:
    explicit MutexRAII(mutex& m) : mtx(m), locked(true) {
        mtx.lock();
        cout << "Mutex locked" << endl;
    }
    
    ~MutexRAII() {
        if (locked) {
            mtx.unlock();
            cout << "Mutex unlocked" << endl;
        }
    }
    
    void unlock() {
        if (locked) {
            mtx.unlock();
            locked = false;
        }
    }
    
    void lock() {
        if (!locked) {
            mtx.lock();
            locked = true;
        }
    }
    
    // No copy or move
    MutexRAII(const MutexRAII&) = delete;
    MutexRAII& operator=(const MutexRAII&) = delete;
};

// 3. RAII for dynamic memory with custom cleanup
template<typename T, typename Deleter = function<void(T*)>>
class ScopedResource {
private:
    T* resource;
    Deleter deleter;
    
public:
    ScopedResource(T* res, Deleter del = [](T* p) { delete p; })
        : resource(res), deleter(del) {
        cout << "Resource acquired: " << (void*)resource << endl;
    }
    
    ~ScopedResource() {
        if (resource) {
            deleter(resource);
            cout << "Resource released: " << (void*)resource << endl;
        }
    }
    
    T* get() { return resource; }
    const T* get() const { return resource; }
    
    T* operator->() { return resource; }
    T& operator*() { return *resource; }
    
    // Release ownership
    T* release() {
        T* temp = resource;
        resource = nullptr;
        return temp;
    }
    
    void reset(T* res = nullptr) {
        if (resource) {
            deleter(resource);
        }
        resource = res;
    }
    
    // Move constructor
    ScopedResource(ScopedResource&& other) noexcept
        : resource(other.resource), deleter(move(other.deleter)) {
        other.resource = nullptr;
    }
    
    // No copy
    ScopedResource(const ScopedResource&) = delete;
    ScopedResource& operator=(const ScopedResource&) = delete;
};

// 4. RAII for database connection
class DBConnection {
private:
    string connectionString;
    bool connected;
    
public:
    DBConnection(const string& connStr) 
        : connectionString(connStr), connected(false) {
        // Simulate connection
        connected = true;
        cout << "Database connected: " << connectionString << endl;
    }
    
    ~DBConnection() {
        if (connected) {
            // Simulate disconnection
            connected = false;
            cout << "Database disconnected" << endl;
        }
    }
    
    void query(const string& sql) {
        if (connected) {
            cout << "Executing query: " << sql << endl;
        }
    }
    
    bool isConnected() const { return connected; }
    
    // Move constructor
    DBConnection(DBConnection&& other) noexcept
        : connectionString(move(other.connectionString)), 
          connected(other.connected) {
        other.connected = false;
    }
    
    // No copy
    DBConnection(const DBConnection&) = delete;
    DBConnection& operator=(const DBConnection&) = delete;
};

// 5. RAII for C-style array
template<typename T>
class ArrayRAII {
private:
    T* arr;
    size_t size;
    
public:
    ArrayRAII(size_t n) : size(n) {
        arr = new T[n];
        cout << "Array allocated: " << n << " elements" << endl;
    }
    
    ~ArrayRAII() {
        delete[] arr;
        cout << "Array deallocated" << endl;
    }
    
    T& operator[](size_t index) {
        if (index >= size) throw out_of_range("Index out of range");
        return arr[index];
    }
    
    const T& operator[](size_t index) const {
        if (index >= size) throw out_of_range("Index out of range");
        return arr[index];
    }
    
    size_t getSize() const { return size; }
    
    // Move constructor
    ArrayRAII(ArrayRAII&& other) noexcept
        : arr(other.arr), size(other.size) {
        other.arr = nullptr;
        other.size = 0;
    }
    
    // No copy
    ArrayRAII(const ArrayRAII&) = delete;
    ArrayRAII& operator=(const ArrayRAII&) = delete;
};

// 6. Composite RAII - managing multiple resources
class ResourceManager {
private:
    FileRAII logFile;
    MutexRAII mutexGuard;
    ScopedResource<int> intResource;
    ArrayRAII<double> doubleArray;
    
public:
    ResourceManager(const string& logPath, mutex& m, int intVal, size_t arraySize)
        : logFile(logPath, "w"), 
          mutexGuard(m),
          intResource(new int(intVal)),
          doubleArray(arraySize) {
        cout << "All resources acquired successfully" << endl;
    }
    
    // Destructor automatically cleans up all resources in reverse order
};

// Demonstration
mutex globalMutex;

void threadSafeWork() {
    MutexRAII lock(globalMutex);
    // Critical section
    cout << "Thread " << this_thread::get_id() << " is working" << endl;
    // Automatic unlock on scope exit
}

int main() {
    // File RAII
    try {
        FileRAII file("test.txt", "w");
        file.write("Hello RAII\n");
        file.write("Second line\n");
        // File automatically closed
    } catch (const exception& e) {
        cerr << "Error: " << e.what() << endl;
    }
    
    // Mutex RAII
    {
        MutexRAII lock(globalMutex);
        cout << "Protected operation" << endl;
    }  // Automatic unlock
    
    // Custom resource with custom deleter
    {
        ScopedResource<FILE, function<void(FILE*)>> 
            file(fopen("data.txt", "r"), [](FILE* f) {
                if (f) fclose(f);
                cout << "Custom file closed" << endl;
            });
        
        if (file.get()) {
            cout << "File opened successfully" << endl;
        }
    }  // Automatic fclose
    
    // Database connection
    {
        DBConnection db("localhost:3306/mydb");
        db.query("SELECT * FROM users");
        // Automatic disconnect
    }
    
    // Array RAII
    {
        ArrayRAII<int> arr(100);
        for (int i = 0; i < 10; i++) {
            arr[i] = i * i;
        }
        // Automatic deletion
    }
    
    // Composite RAII
    {
        ResourceManager manager("log.txt", globalMutex, 42, 50);
        // All resources managed together
    }  // Resources cleaned up in reverse order
    
    // Thread safety demonstration
    vector<thread> threads;
    for (int i = 0; i < 3; i++) {
        threads.emplace_back(threadSafeWork);
    }
    for (auto& t : threads) t.join();
    
    return 0;
}
```

---

## 98. **Const Correctness Deep Dive**

### Question: Explain const overloading, mutable keyword, and const_iterator.

```cpp
#include <iostream>
#include <vector>
#include <cstring>
using namespace std;

// 1. Const member functions and overloading
class StringBuffer {
private:
    char* data;
    size_t len;
    mutable size_t accessCount;  // Can be modified in const methods
    
public:
    StringBuffer(const char* str = "") {
        len = strlen(str);
        data = new char[len + 1];
        strcpy(data, str);
        accessCount = 0;
    }
    
    ~StringBuffer() { delete[] data; }
    
    // Const version - returns const reference
    const char& operator[](size_t index) const {
        accessCount++;  // Allowed because mutable
        return data[index];
    }
    
    // Non-const version - returns reference (allows modification)
    char& operator[](size_t index) {
        accessCount++;
        return data[index];
    }
    
    // Const correctness in return types
    const char* c_str() const { return data; }
    char* c_str() { return data; }
    
    // Const method guarantees no modification
    size_t length() const { return len; }
    size_t getAccessCount() const { return accessCount; }
    
    // Mutable allows modification in const methods
    void resetCounter() const {
        accessCount = 0;  // Allowed because mutable
    }
};

// 2. Const iterator implementation
template<typename T>
class SimpleVector {
private:
    T* data;
    size_t size;
    size_t capacity;
    
public:
    class Iterator {
    private:
        T* ptr;
        
    public:
        Iterator(T* p) : ptr(p) {}
        
        T& operator*() { return *ptr; }
        const T& operator*() const { return *ptr; }
        
        Iterator& operator++() { ptr++; return *this; }
        bool operator!=(const Iterator& other) const { return ptr != other.ptr; }
    };
    
    class ConstIterator {
    private:
        const T* ptr;
        
    public:
        ConstIterator(const T* p) : ptr(p) {}
        
        const T& operator*() const { return *ptr; }
        
        ConstIterator& operator++() { ptr++; return *this; }
        bool operator!=(const ConstIterator& other) const { return ptr != other.ptr; }
    };
    
    // Const and non-const iterators
    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + size); }
    
    ConstIterator begin() const { return ConstIterator(data); }
    ConstIterator end() const { return ConstIterator(data + size); }
    
    ConstIterator cbegin() const { return ConstIterator(data); }
    ConstIterator cend() const { return ConstIterator(data + size); }
};

// 3. Const overloading for optimization
class Matrix {
private:
    vector<vector<int>> data;
    
public:
    // Const version for reading
    int get(int row, int col) const {
        return data[row][col];
    }
    
    // Non-const version for writing (with bounds checking)
    int& get(int row, int col) {
        if (row >= data.size() || col >= data[0].size()) {
            throw out_of_range("Index out of range");
        }
        return data[row][col];
    }
    
    // Const correctness in return types
    const vector<int>& operator[](int row) const {
        return data[row];
    }
    
    vector<int>& operator[](int row) {
        return data[row];
    }
};

// 4. Const parameters and const return values
class Rational {
private:
    int numerator;
    int denominator;
    
public:
    Rational(int num = 0, int den = 1) : numerator(num), denominator(den) {}
    
    // Const return prevents modification
    const Rational operator+(const Rational& other) const {
        return Rational(
            numerator * other.denominator + other.numerator * denominator,
            denominator * other.denominator
        );
    }
    
    // Const reference parameters
    bool operator==(const Rational& other) const {
        return numerator * other.denominator == other.numerator * denominator;
    }
    
    void print() const {
        cout << numerator << "/" << denominator << endl;
    }
};

// 5. Const_cast usage (not recommended but sometimes necessary)
class ConstCastExample {
private:
    int value;
    
public:
    ConstCastExample(int v) : value(v) {}
    
    // Old C API that modifies but shouldn't
    void processData(const int* ptr) {
        // Bad practice - removing const
        int* modifiable = const_cast<int*>(ptr);
        *modifiable = 100;
    }
    
    // Better approach: use mutable for cache
    class ExpensiveComputation {
    private:
        mutable int cache;
        mutable bool cached;
        
    public:
        int compute() const {
            if (!cached) {
                cache = expensiveCalculation();
                cached = true;
            }
            return cache;
        }
        
    private:
        int expensiveCalculation() const {
            // Simulate expensive computation
            return 42;
        }
    };
};

// 6. Const and polymorphism
class Base {
public:
    virtual void print() const {
        cout << "Base const" << endl;
    }
    
    virtual void print() {
        cout << "Base non-const" << endl;
    }
};

class Derived : public Base {
public:
    void print() const override {
        cout << "Derived const" << endl;
    }
    
    void print() override {
        cout << "Derived non-const" << endl;
    }
};

int main() {
    // String buffer demonstration
    StringBuffer sb("Hello");
    const StringBuffer& csb = sb;
    
    sb[0] = 'h';        // Non-const version
    cout << csb[0] << endl;  // Const version
    
    // Const iteration
    const SimpleVector<int> constVec = SimpleVector<int>();
    // for (auto& val : constVec) - Error! Can't get non-const iterator
    
    // Matrix example
    Matrix mat;
    mat.get(0, 0) = 42;  // Non-const
    int val = mat.get(0, 0);  // Const version if mat is const
    
    // Pointer const-ness
    int x = 10, y = 20;
    const int* ptr1 = &x;  // ptr to const int
    // *ptr1 = 30;  // Error!
    ptr1 = &y;  // OK
    
    int* const ptr2 = &x;  // Const pointer to int
    *ptr2 = 30;  // OK
    // ptr2 = &y;  // Error!
    
    const int* const ptr3 = &x;  // Const pointer to const int
    // *ptr3 = 30;  // Error!
    // ptr3 = &y;   // Error!
    
    // Polymorphic const
    Base* b1 = new Base();
    Base* b2 = new Derived();
    
    b1->print();  // Base non-const
    b2->print();  // Derived non-const
    
    const Base* cb1 = new Base();
    const Base* cb2 = new Derived();
    
    cb1->print();  // Base const
    cb2->print();  // Derived const
    
    delete b1; delete b2; delete cb1; delete cb2;
    
    return 0;
}
```

---

## 99. **Explicit Constructor and Type Conversion**

### Question: When and why to use explicit constructors?

```cpp
#include <iostream>
#include <vector>
#include <memory>
using namespace std;

// 1. Basic explicit constructor
class Integer {
private:
    int value;
    
public:
    // Implicit constructor - DANGEROUS!
    // Integer(int v) : value(v) {}
    
    // Explicit constructor - SAFE
    explicit Integer(int v) : value(v) {
        cout << "Integer created: " << value << endl;
    }
    
    int getValue() const { return value; }
};

void printInteger(const Integer& i) {
    cout << "Value: " << i.getValue() << endl;
}

// 2. Complex class with multiple constructors
class String {
private:
    char* data;
    size_t length;
    
public:
    // Constructor from const char*
    String(const char* str) {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        cout << "String created from C-string: " << data << endl;
    }
    
    // Explicit size constructor - prevents implicit conversion from int
    explicit String(size_t size) {
        length = size;
        data = new char[size + 1];
        memset(data, 0, size + 1);
        cout << "String created with size: " << size << endl;
    }
    
    // Copy constructor
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
        cout << "String copied: " << data << endl;
    }
    
    ~String() {
        delete[] data;
    }
    
    void print() const {
        cout << (data ? data : "null") << endl;
    }
};

// 3. Problem without explicit (demonstration)
class DangerousArray {
private:
    vector<int> data;
    
public:
    // BAD: Implicit constructor from size
    DangerousArray(int size) : data(size) {
        cout << "Created array of size: " << size << endl;
    }
    
    // Another constructor
    DangerousArray(const vector<int>& vec) : data(vec) {
        cout << "Created array from vector" << endl;
    }
    
    // This would cause ambiguity!
    void process() {
        cout << "Processing array of size: " << data.size() << endl;
    }
};

// 4. Safe version with explicit
class SafeArray {
private:
    vector<int> data;
    
public:
    // GOOD: explicit constructor
    explicit SafeArray(int size) : data(size) {
        cout << "Created array of size: " << size << endl;
    }
    
    SafeArray(const vector<int>& vec) : data(vec) {
        cout << "Created array from vector" << endl;
    }
    
    void process() const {
        cout << "Processing array of size: " << data.size() << endl;
    }
};

// 5. Conversion operators (also can be explicit C++11)
class Number {
private:
    double value;
    
public:
    Number(double v) : value(v) {}
    
    // Implicit conversion to double
    operator double() const {
        return value;
    }
    
    // Explicit conversion to int (C++11)
    explicit operator int() const {
        return static_cast<int>(value);
    }
};

// 6. Real-world example: vector emplacement
class Point {
private:
    int x, y;
    
public:
    Point(int x, int y) : x(x), y(y) {
        cout << "Point created: (" << x << ", " << y << ")" << endl;
    }
    
    // Explicit single-argument constructor
    explicit Point(int val) : x(val), y(val) {
        cout << "Point created from single value: " << val << endl;
    }
    
    void print() const {
        cout << "(" << x << ", " << y << ")" << endl;
    }
};

// 7. Factory pattern with explicit constructors
class Product {
private:
    Product() { cout << "Product created" << endl; }
    Product(int id) : id_(id) { cout << "Product created with id: " << id << endl; }
    
    int id_;
    
public:
    // Factory methods instead of public constructors
    static Product createDefault() {
        return Product();
    }
    
    static Product createWithId(int id) {
        return Product(id);
    }
    
    int getId() const { return id_; }
};

// 8. Tag dispatching to avoid explicit constructors
struct CreateWithSize {};
struct CreateWithDefault {};

class FlexibleArray {
private:
    vector<int> data;
    
public:
    // Ambiguous constructor avoided with tags
    FlexibleArray(CreateWithSize, size_t size) : data(size) {}
    FlexibleArray(CreateWithDefault) : data(100) {}
    FlexibleArray(const vector<int>& vec) : data(vec) {}
};

int main() {
    // Problem demonstration
    cout << "=== Without explicit ===" << endl;
    // If DangerousArray had implicit constructor:
    // DangerousArray arr1 = 10;  // Implicit conversion from int
    // arr1.process();
    
    // DangerousArray arr2 = {1,2,3};  // Ambiguous!
    
    // With explicit constructor
    cout << "\n=== With explicit ===" << endl;
    // SafeArray arr1 = 10;  // ERROR: cannot convert int to SafeArray
    SafeArray arr2(10);      // OK: direct initialization
    SafeArray arr3({1,2,3});  // OK: explicit constructor call
    
    // Integer example
    cout << "\n=== Integer example ===" << endl;
    Integer i1(42);          // OK: direct initialization
    // Integer i2 = 42;      // ERROR: explicit constructor prevents this
    // printInteger(100);    // ERROR: cannot convert int to Integer
    printInteger(i1);         // OK
    
    // String example
    cout << "\n=== String example ===" << endl;
    String s1("Hello");       // OK
    String s2(100);           // OK: explicit call
    // String s3 = 100;       // ERROR: explicit constructor
    
    // Number conversion operators
    cout << "\n=== Conversion operators ===" << endl;
    Number num(3.14);
    double d = num;           // Implicit conversion to double (OK)
    // int n = num;           // ERROR: conversion to int is explicit
    int n = static_cast<int>(num);  // OK: explicit cast
    
    // Point example with vector emplacement
    cout << "\n=== Emplacement example ===" << endl;
    vector<Point> points;
    points.emplace_back(10, 20);    // OK: calls Point(int,int)
    // points.emplace_back(5);      // ERROR: ambiguous without explicit
    // With explicit: still ambiguous? Actually calls Point(int)
    // But could be confusing
    
    // Factory pattern
    cout << "\n=== Factory pattern ===" << endl;
    auto p1 = Product::createDefault();
    auto p2 = Product::createWithId(100);
    
    // Tag dispatching
    cout << "\n=== Tag dispatching ===" << endl;
    FlexibleArray fa1(CreateWithSize(), 50);
    FlexibleArray fa2(CreateWithDefault());
    FlexibleArray fa3({1,2,3,4});
    
    // Best practices summary:
    cout << "\n=== Best Practices ===" << endl;
    cout << "1. Use explicit for single-argument constructors" << endl;
    cout << "2. Avoid implicit conversions that lose information" << endl;
    cout << "3. Use factory methods for complex construction" << endl;
    cout << "4. Consider tag dispatching for multiple construction patterns" << endl;
    cout << "5. Make conversion operators explicit when lossy" << endl;
    
    return 0;
}
```

---

## 100. **Deep vs Shallow Copy - Comprehensive Guide**

### Question: Implement deep copy, shallow copy, and copy-on-write.

```cpp
#include <iostream>
#include <cstring>
#include <memory>
#include <atomic>
using namespace std;

// 1. Shallow copy (DANGEROUS)
class ShallowString {
private:
    char* data;
    size_t length;
    
public:
    ShallowString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        cout << "ShallowString constructed: " << data << endl;
    }
    
    // Shallow copy - just copies pointer
    ShallowString(const ShallowString& other) 
        : data(other.data), length(other.length) {
        cout << "ShallowString COPY - sharing memory: " << data << endl;
    }
    
    ~ShallowString() {
        cout << "ShallowString destroyed: " << (data ? data : "null") << endl;
        delete[] data;
    }
    
    void modify(const char* newStr) {
        delete[] data;
        length = strlen(newStr);
        data = new char[length + 1];
        strcpy(data, newStr);
        cout << "ShallowString modified to: " << data << endl;
    }
    
    void print() const { cout << "Data: " << data << endl; }
    
    // This will cause double delete!
};

// 2. Deep copy - correct implementation
class DeepString {
private:
    char* data;
    size_t length;
    
public:
    DeepString(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
        cout << "DeepString constructed: " << data << endl;
    }
    
    // Deep copy - copies actual data
    DeepString(const DeepString& other) : length(other.length) {
        data = new char[length + 1];
        strcpy(data, other.data);
        cout << "DeepString COPY - new memory: " << data << endl;
    }
    
    // Assignment operator - deep copy
    DeepString& operator=(const DeepString& other) {
        if (this != &other) {
            cout << "DeepString assignment: " << data << " = " << other.data << endl;
            delete[] data;
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        return *this;
    }
    
    ~DeepString() {
        cout << "DeepString destroyed: " << (data ? data : "null") << endl;
        delete[] data;
    }
    
    void modify(const char* newStr) {
        delete[] data;
        length = strlen(newStr);
        data = new char[length + 1];
        strcpy(data, newStr);
        cout << "DeepString modified to: " << data << endl;
    }
    
    void print() const { cout << "Data: " << data << endl; }
};

// 3. Copy-on-write (COW) optimization
class CowString {
private:
    struct RefCounted {
        char* data;
        size_t length;
        atomic<int> refCount;
        
        RefCounted(const char* str) : length(strlen(str)), refCount(1) {
            data = new char[length + 1];
            strcpy(data, str);
            cout << "RefCounted created: " << data << " (ref=" << refCount << ")" << endl;
        }
        
        ~RefCounted() {
            cout << "RefCounted destroyed: " << (data ? data : "null") << endl;
            delete[] data;
        }
        
        void addRef() { 
            refCount++;
            cout << "Reference added: " << data << " (ref=" << refCount << ")" << endl;
        }
        
        void releaseRef() {
            refCount--;
            cout << "Reference released: " << data << " (ref=" << refCount << ")" << endl;
            if (refCount == 0) {
                delete this;
            }
        }
    };
    
    RefCounted* m_data;
    
public:
    CowString(const char* str = "") : m_data(new RefCounted(str)) {}
    
    // Copy constructor - shares reference
    CowString(const CowString& other) : m_data(other.m_data) {
        m_data->addRef();
    }
    
    // Assignment operator
    CowString& operator=(const CowString& other) {
        if (this != &other) {
            if (m_data) m_data->releaseRef();
            m_data = other.m_data;
            m_data->addRef();
        }
        return *this;
    }
    
    // Detach before writing (copy-on-write)
    void detach() {
        if (m_data->refCount > 1) {
            cout << "COW: Detaching before write..." << endl;
            RefCounted* old = m_data;
            m_data = new RefCounted(old->data);  // Create new copy
            old->releaseRef();
        }
    }
    
    void modify(const char* newStr) {
        detach();  // Make sure we have exclusive access
        delete[] m_data->data;
        m_data->length = strlen(newStr);
        m_data->data = new char[m_data->length + 1];
        strcpy(m_data->data, newStr);
        cout << "CowString modified to: " << m_data->data << endl;
    }
    
    void print() const { 
        cout << "CowString: " << m_data->data 
             << " (refCount=" << m_data->refCount << ")" << endl;
    }
    
    ~CowString() {
        if (m_data) m_data->releaseRef();
    }
};

// 4. Template-based deep copy
template<typename T>
class DeepCopyPtr {
private:
    T* ptr;
    
public:
    DeepCopyPtr(T* p = nullptr) : ptr(p) {}
    
    DeepCopyPtr(const DeepCopyPtr& other) {
        ptr = other.ptr ? new T(*other.ptr) : nullptr;
        cout << "DeepCopyPtr: Deep copied" << endl;
    }
    
    DeepCopyPtr& operator=(const DeepCopyPtr& other) {
        if (this != &other) {
            delete ptr;
            ptr = other.ptr ? new T(*other.ptr) : nullptr;
        }
        return *this;
    }
    
    ~DeepCopyPtr() { delete ptr; }
    
    T* operator->() { return ptr; }
    const T* operator->() const { return ptr; }
    T& operator*() { return *ptr; }
    
    // Move constructor
    DeepCopyPtr(DeepCopyPtr&& other) noexcept : ptr(other.ptr) {
        other.ptr = nullptr;
    }
};

// 5. Clone pattern for polymorphic classes
class Cloneable {
public:
    virtual ~Cloneable() = default;
    virtual Cloneable* clone() const = 0;  // Deep copy
    virtual void print() const = 0;
};

class Dog : public Cloneable {
private:
    string name;
    int age;
    
public:
    Dog(const string& n, int a) : name(n), age(a) {}
    
    Dog* clone() const override {
        cout << "Cloning Dog: " << name << endl;
        return new Dog(*this);
    }
    
    void print() const override {
        cout << "Dog: " << name << ", age: " << age << endl;
    }
};

class Cat : public Cloneable {
private:
    string name;
    string color;
    
public:
    Cat(const string& n, const string& c) : name(n), color(c) {}
    
    Cat* clone() const override {
        cout << "Cloning Cat: " << name << endl;
        return new Cat(*this);
    }
    
    void print() const override {
        cout << "Cat: " << name << ", color: " << color << endl;
    }
};

// Demonstration
int main() {
    cout << "=== SHALLOW COPY (DANGEROUS) ===" << endl;
    {
        ShallowString s1("Hello");
        ShallowString s2 = s1;  // Shallow copy - same pointer
        
        s1.print();  // Hello
        s2.print();  // Hello
        
        s1.modify("World");  // s1 gets new memory
        
        s1.print();  // World
        s2.print();  // Points to old memory that was deleted! Dangling pointer!
    }  // Double delete! Undefined behavior
    
    cout << "\n=== DEEP COPY (SAFE) ===" << endl;
    {
        DeepString s1("Hello");
        DeepString s2 = s1;  // Deep copy - new memory
        
        s1.modify("World");
        
        s1.print();  // World
        s2.print();  // Hello (unchanged)
    }  // Clean destruction
    
    cout << "\n=== COPY-ON-WRITE (OPTIMIZED) ===" << endl;
    {
        CowString s1("Hello");
        CowString s2 = s1;   // Shares reference
        CowString s3 = s1;   // Shares reference
        
        s1.print();  // Hello (ref=3)
        s2.print();  // Hello (ref=3)
        
        s1.modify("World");  // Detaches and creates new copy
        
        s1.print();  // World (ref=1)
        s2.print();  // Hello (ref=2)
        s3.print();  // Hello (ref=2)
    }
    
    cout << "\n=== DEEP COPY POINTER TEMPLATE ===" << endl;
    {
        DeepCopyPtr<string> ptr1(new string("Hello"));
        DeepCopyPtr<string> ptr2 = ptr1;  // Deep copy
        
        *ptr1 = "World";  // Modify one
        
        cout << "ptr1: " << *ptr1 << endl;  // World
        cout << "ptr2: " << *ptr2 << endl;  // Hello
    }
    
    cout << "\n=== CLONE PATTERN ===" << endl;
    {
        vector<Cloneable*> animals;
        animals.push_back(new Dog("Rex", 5));
        animals.push_back(new Cat("Whiskers", "black"));
        
        // Deep copy using clone
        vector<Cloneable*> clones;
        for (auto a : animals) {
            clones.push_back(a->clone());
        }
        
        // Original vs clones (independent)
        for (auto a : animals) a->print();
        for (auto a : clones) a->print();
        
        // Cleanup
        for (auto a : animals) delete a;
        for (auto a : clones) delete a;
    }
    
    cout << "\n=== MEMORY LAYOUT COMPARISON ===" << endl;
    cout << "Shallow copy: Stack pointer -> Shared heap memory" << endl;
    cout << "Deep copy:    Stack pointer -> Independent heap memory" << endl;
    cout << "COW:          Stack pointer -> Shared heap + refCount" << endl;
    
    return 0;
}
```

---

## 📚 **Summary & Best Practices**

| Concept | Best Practice | Common Pitfall |
|---------|--------------|----------------|
| **Virtual Functions** | Always use `override` keyword | Forgetting virtual destructor |
| **Copy/Move** | Follow Rule of 5 | Shallow copy of owning pointers |
| **Smart Pointers** | Prefer `unique_ptr`, then `shared_ptr` | Circular references with `shared_ptr` |
| **Move Semantics** | Mark move ops `noexcept` | Using moved-from objects |
| **Templates** | Use `forward` for perfect forwarding | Slicing with universal references |
| **Singleton** | Use Meyers singleton | Double-checked locking without atomic |
| **RAII** | Acquire in constructor, release in destructor | Manual resource management |
| **Const Correctness** | Mark member functions `const` when possible | Modifying logical state in const methods |
| **Explicit** | Use for single-argument constructors | Unintended implicit conversions |
| **Deep Copy** | Implement for owning pointers | Double delete or memory leaks |

### Advanced Topics Not Covered (Ask for expansion):
- CRTP (Curiously Recurring Template Pattern)
- std::variant vs union
- Exception safety guarantees
- Pimpl idiom
- Type erasure
- SFINAE vs Concepts (C++20)

Would you like me to expand on **any of these advanced topics** or provide **more test cases and edge cases** for specific OOP questions?