# Top 50 C++ Interview Questions
## Templates

---

## Table of Contents
1. [Template Fundamentals (Q1–Q10)](#template-fundamentals)
2. [Function Templates (Q11–Q18)](#function-templates)
3. [Class Templates (Q19–Q26)](#class-templates)
4. [Template Specialization (Q27–Q33)](#template-specialization)
5. [Variadic Templates (Q34–Q38)](#variadic-templates)
6. [Concepts & Constraints — C++20 (Q39–Q43)](#concepts--constraints-c20)
7. [Advanced Template Techniques (Q44–Q50)](#advanced-template-techniques)

---

## Template Fundamentals

---

### Q1. What is a template in C++?

**Answer:**  
A template is a compile-time blueprint that lets you write generic code parameterized by types, values, or other templates. The compiler generates concrete code by substituting the template parameters — a process called **instantiation**. Templates are C++'s primary mechanism for generic programming.

```cpp
template<typename T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

std::cout << maximum(3, 7);          // T = int     → Output: 7
std::cout << maximum(3.14, 2.71);    // T = double  → Output: 3.14
std::cout << maximum('a', 'z');      // T = char    → Output: z
```

---

### Q2. What is template instantiation?

**Answer:**  
Template instantiation is the process by which the compiler generates a concrete function or class from a template by substituting actual types or values for the template parameters. This happens at compile time, not runtime.

- **Implicit instantiation**: compiler instantiates when template is used.
- **Explicit instantiation**: programmer forces instantiation for specific types.

```cpp
template<typename T>
void print(T val) { std::cout << val << "\n"; }

print(42);           // implicit: compiler generates print<int>
print(3.14);         // implicit: compiler generates print<double>

// Explicit instantiation (forces generation — used in .cpp files):
template void print<std::string>(std::string); // explicit
```

---

### Q3. What is the difference between `typename` and `class` in a template parameter list?

**Answer:**  
`typename` and `class` are interchangeable when declaring type template parameters. There is **no semantic difference** in this context. `typename` is also used in dependent contexts to tell the compiler a dependent name is a type (where `class` cannot be used).

```cpp
template<typename T>  // same as:
template<class T>     // identical for type parameters

// typename is REQUIRED in dependent type contexts:
template<typename Container>
void printFirst(const Container& c) {
    typename Container::value_type first = *c.begin(); // typename required here
    std::cout << first;
}
```

---

### Q4. What is a non-type template parameter?

**Answer:**  
Template parameters can be compile-time constant values (integers, pointers, enums, etc.) rather than types. C++20 expanded this to include floating-point and class types.

```cpp
template<typename T, int Size>
class FixedArray {
    T data[Size];
public:
    T& operator[](int i) { return data[i]; }
    int size() const { return Size; }
};

FixedArray<int, 5>    intArr;    // array of 5 ints (stack-allocated)
FixedArray<double, 3> dblArr;   // array of 3 doubles

intArr[0] = 42;
std::cout << intArr.size(); // Output: 5

// Template with bool parameter:
template<bool Debug>
void log(const std::string& msg) {
    if constexpr (Debug) std::cout << "[DEBUG] " << msg << "\n";
}

log<true>("test");  // prints
log<false>("test"); // compiled away entirely
```

---

### Q5. What is SFINAE (Substitution Failure Is Not An Error)?

**Answer:**  
SFINAE is a rule that says: if substituting template arguments causes a substitution failure (e.g., a type that doesn't exist), the compiler silently discards that template from the overload set rather than reporting an error. This is the basis for compile-time type-based selection before C++20 Concepts.

```cpp
#include <type_traits>

// Enabled only for integral types:
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
add(T a, T b) {
    std::cout << "(integral) ";
    return a + b;
}

// Enabled only for floating-point types:
template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
add(T a, T b) {
    std::cout << "(float) ";
    return a + b;
}

std::cout << add(1, 2);       // Output: (integral) 3
std::cout << add(1.5, 2.5);   // Output: (float) 4
```

---

### Q6. What is template argument deduction?

**Answer:**  
The compiler automatically deduces template arguments from function call arguments, so you don't need to specify them explicitly. Deduction rules follow specific patterns.

```cpp
template<typename T>
void show(T val) { std::cout << val; }

show(42);          // T deduced as int
show(3.14);        // T deduced as double
show("hello");     // T deduced as const char*

template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

auto result = add(1, 2.5); // T=int, U=double, return type=double
```

---

### Q7. What is Class Template Argument Deduction (CTAD, C++17)?

**Answer:**  
Before C++17, class templates required explicit type arguments. CTAD allows the compiler to deduce class template parameters from constructor arguments, just like function template deduction.

```cpp
// Before C++17 — must specify type:
std::pair<int, std::string> p1(42, "hello");
std::vector<int> v1 = {1, 2, 3};

// C++17 CTAD — types deduced:
std::pair p2(42, "hello");      // deduced: pair<int, const char*>
std::vector v2 = {1, 2, 3};    // deduced: vector<int>

// Custom class with CTAD:
template<typename T>
class Box {
    T value;
public:
    Box(T v) : value(v) {}
};

Box b1(42);        // deduced: Box<int>
Box b2(3.14);      // deduced: Box<double>
Box b3("hello");   // deduced: Box<const char*>
```

---

### Q8. What are deduction guides (C++17)?

**Answer:**  
Deduction guides help the compiler deduce template arguments for CTAD when the constructors alone don't provide enough information.

```cpp
template<typename T>
class Stack {
    std::vector<T> data;
public:
    Stack() = default;
    Stack(std::initializer_list<T> list) : data(list) {}
};

// Deduction guide: when initialized with initializer_list<T>, deduce Stack<T>
template<typename T>
Stack(std::initializer_list<T>) -> Stack<T>;

Stack s = {1, 2, 3}; // Stack<int> deduced via guide
Stack<double> s2;     // explicit still works
```

---

### Q9. What is a template template parameter?

**Answer:**  
A template can accept another template (not just a type or value) as a parameter. This allows writing generic code that works with any container template.

```cpp
template<typename T, template<typename> class Container>
class Adaptor {
    Container<T> data;
public:
    void add(T val) { data.push_back(val); }
    void print() const {
        for (const auto& x : data) std::cout << x << " ";
    }
};

Adaptor<int, std::vector> va;
va.add(1); va.add(2); va.add(3);
va.print(); // Output: 1 2 3

Adaptor<int, std::deque> da;
da.add(10); da.add(20);
da.print(); // Output: 10 20
```

---

### Q10. What is explicit template instantiation and why use it?

**Answer:**  
Explicit template instantiation forces the compiler to generate code for a specific template specialization in a specific translation unit (.cpp file). This reduces compilation time and controls which .cpp file contains the template implementation.

```cpp
// math.h:
template<typename T> T square(T x);

// math.cpp:
template<typename T> T square(T x) { return x * x; }

// Explicit instantiations — only these types generated in math.cpp:
template int    square<int>(int);
template double square<double>(double);
template float  square<float>(float);

// other.cpp:
extern template int square<int>(int); // tells linker: already in math.cpp
```

---

## Function Templates

---

### Q11. How do you write a function template?

**Answer:**  
Use the `template<typename T>` (or `template<class T>`) prefix before the function. The type parameter `T` can appear anywhere in the function signature and body.

```cpp
template<typename T>
T clamp(T value, T low, T high) {
    return (value < low) ? low : (value > high) ? high : value;
}

std::cout << clamp(5, 1, 10);    // Output: 5
std::cout << clamp(-3, 0, 100);  // Output: 0
std::cout << clamp(150, 0, 100); // Output: 100
std::cout << clamp(3.7, 0.0, 5.0); // Output: 3.7
```

---

### Q12. What is explicit template argument specification for functions?

**Answer:**  
You can explicitly specify template arguments when calling a function template, overriding or supplementing deduction.

```cpp
template<typename To, typename From>
To convert(From val) {
    return static_cast<To>(val);
}

auto i = convert<int>(3.99);      // To=int, From=double deduced → 3
auto d = convert<double, int>(5); // both explicit → 5.0
auto f = convert<float>(42);      // To=float, From=int deduced → 42.0f

std::cout << i << " " << d << " " << f;
```

---

### Q13. What is a function template overload?

**Answer:**  
Function templates can be overloaded with other templates or regular functions. The compiler selects the best match using overload resolution rules. Regular (non-template) functions are preferred over templates if they match equally.

```cpp
template<typename T>
void display(T val) {
    std::cout << "Template: " << val << "\n";
}

// Overload for pointers:
template<typename T>
void display(T* val) {
    std::cout << "Pointer template: " << *val << "\n";
}

// Non-template overload — preferred for exact match:
void display(int val) {
    std::cout << "Non-template int: " << val << "\n";
}

display(42);       // Output: Non-template int: 42
display(3.14);     // Output: Template: 3.14
int x = 10;
display(&x);       // Output: Pointer template: 10
```

---

### Q14. What is `std::enable_if` and how does it work?

**Answer:**  
`std::enable_if<Condition, T>` is a metaprogramming utility that makes a type (`T`) available only when `Condition` is true. Used to constrain templates via SFINAE.

```cpp
#include <type_traits>

// Only enabled for arithmetic types:
template<typename T>
std::enable_if_t<std::is_arithmetic_v<T>, T>
doubled(T val) { return val * 2; }

std::cout << doubled(5);      // Output: 10
std::cout << doubled(2.5);    // Output: 5.0
// doubled("hi");             // ERROR: not arithmetic → SFINAE removes it

// C++17 with if constexpr — cleaner alternative:
template<typename T>
auto process(T val) {
    if constexpr (std::is_integral_v<T>)
        return val * 2;
    else
        return val * 2.0;
}
```

---

### Q15. What is `decltype` and `auto` in function template return types?

**Answer:**  
`decltype(expr)` deduces the type of an expression at compile time. Used with trailing return types and `auto` to express return types that depend on template parameters.

```cpp
// Trailing return type with decltype:
template<typename T, typename U>
auto multiply(T a, U b) -> decltype(a * b) {
    return a * b;
}

// C++14: auto return type deduction:
template<typename T, typename U>
auto add(T a, U b) {
    return a + b; // return type deduced from expression
}

auto r1 = multiply(3, 2.5);   // double
auto r2 = add(1, 2);          // int
auto r3 = add(1, 2.0);        // double
std::cout << r1 << " " << r2 << " " << r3;
```

---

### Q16. What is perfect forwarding?

**Answer:**  
Perfect forwarding preserves the value category (lvalue/rvalue) and cv-qualifiers of function arguments when passing them to another function. Uses **forwarding references** (`T&&` where `T` is deduced) and `std::forward<T>`.

```cpp
template<typename T>
void wrapper(T&& arg) {
    // forward preserves lvalue/rvalue nature of arg
    process(std::forward<T>(arg));
}

void process(int& x)  { std::cout << "lvalue\n"; }
void process(int&& x) { std::cout << "rvalue\n"; }

int a = 5;
wrapper(a);             // T=int&  → lvalue forwarded → process(int&)
wrapper(10);            // T=int   → rvalue forwarded → process(int&&)
wrapper(std::move(a));  // T=int   → rvalue forwarded → process(int&&)
```

---

### Q17. What is a forwarding reference (universal reference)?

**Answer:**  
A forwarding reference is `T&&` where `T` is a **deduced** template parameter (or `auto&&`). It binds to both lvalues and rvalues. The reference collapsing rules determine the actual reference type.

```cpp
template<typename T>
void identify(T&& x) {
    if constexpr (std::is_lvalue_reference_v<T>)
        std::cout << "lvalue reference\n";
    else
        std::cout << "rvalue reference\n";
}

int x = 5;
identify(x);            // T=int&  → lvalue reference
identify(5);            // T=int   → rvalue reference
identify(std::move(x)); // T=int   → rvalue reference

// auto&& is also a forwarding reference:
auto&& a = x;  // a is lvalue reference (int&)
auto&& b = 5;  // b is rvalue reference (int&&)
```

---

### Q18. What is `std::forward` vs `std::move`?

**Answer:**  
- `std::move(x)` — unconditionally casts `x` to an rvalue reference.
- `std::forward<T>(x)` — conditionally casts to rvalue only if `T` is not an lvalue reference type. Use in forwarding/wrapper templates; never in ordinary code where `move` suffices.

```cpp
template<typename T>
void forwardExample(T&& val) {
    auto a = std::forward<T>(val); // preserves original category
    auto b = std::move(val);       // always moves — loses lvalue info
}

// std::make_unique implementation style:
template<typename T, typename... Args>
std::unique_ptr<T> myMakeUnique(Args&&... args) {
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...)); // perfect forward
}
```

---

## Class Templates

---

### Q19. How do you write a class template?

**Answer:**  
Prefix the class with `template<typename T>`. All member functions defined outside the class must also carry the template prefix.

```cpp
template<typename T>
class Stack {
    std::vector<T> data;
public:
    void push(const T& val) { data.push_back(val); }
    void push(T&& val)      { data.push_back(std::move(val)); }
    void pop()              { data.pop_back(); }
    T&   top()              { return data.back(); }
    bool empty() const      { return data.empty(); }
    int  size()  const      { return data.size(); }
};

Stack<int>         intStack;
Stack<std::string> strStack;

intStack.push(1); intStack.push(2);
std::cout << intStack.top(); // Output: 2

strStack.push("hello");
std::cout << strStack.top(); // Output: hello
```

---

### Q20. How do you define member functions of a class template outside the class?

**Answer:**  
Each member function definition outside the class body must be prefixed with the full template header and the class name must include the template parameter list.

```cpp
template<typename T>
class Pair {
    T first, second;
public:
    Pair(T a, T b);
    T getFirst() const;
    T getSecond() const;
    void swap();
};

template<typename T>
Pair<T>::Pair(T a, T b) : first(a), second(b) {}

template<typename T>
T Pair<T>::getFirst() const { return first; }

template<typename T>
T Pair<T>::getSecond() const { return second; }

template<typename T>
void Pair<T>::swap() { std::swap(first, second); }

Pair<int> p(10, 20);
p.swap();
std::cout << p.getFirst(); // Output: 20
```

---

### Q21. What is a class template with multiple type parameters?

**Answer:**  
A class template can have any number of type or non-type parameters.

```cpp
template<typename Key, typename Value, int MaxSize = 100>
class BoundedMap {
    std::pair<Key, Value> data[MaxSize];
    int count = 0;
public:
    void insert(Key k, Value v) {
        if (count < MaxSize) data[count++] = {k, v};
    }
    Value* find(const Key& k) {
        for (int i = 0; i < count; ++i)
            if (data[i].first == k) return &data[i].second;
        return nullptr;
    }
};

BoundedMap<std::string, int, 50> m;
m.insert("Alice", 95);
m.insert("Bob",   87);
auto* v = m.find("Alice");
if (v) std::cout << *v; // Output: 95
```

---

### Q22. What are default template arguments?

**Answer:**  
Template parameters can have default values, just like function parameters. They must appear after all non-default parameters.

```cpp
template<typename T, typename Allocator = std::allocator<T>>
class MyVector {
    std::vector<T, Allocator> data;
public:
    void push(T val) { data.push_back(std::move(val)); }
    size_t size() const { return data.size(); }
};

MyVector<int>    v1; // uses default allocator
v1.push(1); v1.push(2);
std::cout << v1.size(); // Output: 2

// Non-type default:
template<typename T, size_t N = 10>
class FixedBuf {
    T buf[N];
public:
    size_t capacity() const { return N; }
};

FixedBuf<int>     fb1; // N=10
FixedBuf<int, 32> fb2; // N=32
```

---

### Q23. Can a class template have static members?

**Answer:**  
Yes. Each instantiation of a class template has its **own** static members — they are not shared across different specializations.

```cpp
template<typename T>
class Counter {
    static int count;
public:
    Counter()  { ++count; }
    ~Counter() { --count; }
    static int getCount() { return count; }
};

template<typename T>
int Counter<T>::count = 0; // definition required

Counter<int>    a, b, c;
Counter<double> x, y;

std::cout << Counter<int>::getCount();    // Output: 3
std::cout << Counter<double>::getCount(); // Output: 2
// int and double counters are separate!
```

---

### Q24. What is a member function template inside a class?

**Answer:**  
A class (template or regular) can have member functions that are themselves templates, providing additional type flexibility at the function level.

```cpp
class Converter {
public:
    template<typename To, typename From>
    static To cast(From val) {
        return static_cast<To>(val);
    }

    template<typename T>
    void print(T val) const {
        std::cout << val << "\n";
    }
};

Converter c;
int   i = Converter::cast<int>(3.99);    // 3
float f = Converter::cast<float, int>(5); // 5.0f
c.print(42);
c.print("hello");
c.print(3.14);
```

---

### Q25. What is `typename` in a dependent type context?

**Answer:**  
When a type depends on a template parameter (a dependent name), the compiler cannot determine at parse time whether it's a type or a value. You must use `typename` to disambiguate.

```cpp
template<typename Container>
class Wrapper {
public:
    // typename required: Container::value_type depends on Container
    using ValueType = typename Container::value_type;

    void printFirst(const Container& c) {
        typename Container::const_iterator it = c.begin(); // typename needed
        if (it != c.end()) std::cout << *it;
    }
};

Wrapper<std::vector<int>> w;
std::vector<int> v = {10, 20, 30};
w.printFirst(v); // Output: 10
```

---

### Q26. What is `template` disambiguation in dependent contexts?

**Answer:**  
When calling a member template of a dependent type, use the `template` keyword to tell the compiler it's a template call, not a less-than operator.

```cpp
template<typename T>
void callMemberTemplate(T& obj) {
    // Without 'template', compiler may parse '<' as less-than:
    obj.template convert<int>(); // 'template' keyword required here
}

struct MyObj {
    template<typename T>
    void convert() { std::cout << "convert<" << typeid(T).name() << ">\n"; }
};

MyObj o;
callMemberTemplate(o); // Output: convert<int> (or int's mangled name)
```

---

## Template Specialization

---

### Q27. What is full (explicit) template specialization?

**Answer:**  
Full specialization provides a completely custom implementation for a specific set of template arguments. The `template<>` prefix with empty angle brackets marks it.

```cpp
// Primary template:
template<typename T>
struct Serializer {
    static std::string serialize(T val) {
        return std::to_string(val);
    }
};

// Full specialization for bool:
template<>
struct Serializer<bool> {
    static std::string serialize(bool val) {
        return val ? "true" : "false"; // custom for bool
    }
};

// Full specialization for std::string:
template<>
struct Serializer<std::string> {
    static std::string serialize(const std::string& val) {
        return "\"" + val + "\""; // quoted
    }
};

std::cout << Serializer<int>::serialize(42);          // Output: 42
std::cout << Serializer<bool>::serialize(true);        // Output: true
std::cout << Serializer<std::string>::serialize("hi"); // Output: "hi"
```

---

### Q28. What is partial template specialization?

**Answer:**  
Partial specialization provides a custom implementation for a **subset** of template arguments (some are still generic). Only available for class templates, not function templates.

```cpp
// Primary template:
template<typename T, typename U>
struct TypeInfo {
    static std::string name() { return "generic<T,U>"; }
};

// Partial specialization: U is a pointer to T:
template<typename T>
struct TypeInfo<T, T*> {
    static std::string name() { return "pointer specialization"; }
};

// Partial specialization: both same type:
template<typename T>
struct TypeInfo<T, T> {
    static std::string name() { return "same-type specialization"; }
};

std::cout << TypeInfo<int, double>::name(); // Output: generic<T,U>
std::cout << TypeInfo<int, int*>::name();   // Output: pointer specialization
std::cout << TypeInfo<int, int>::name();    // Output: same-type specialization
```

---

### Q29. What is specialization for pointer types?

**Answer:**  
A common use of partial specialization is providing a separate implementation when the template type is a pointer.

```cpp
template<typename T>
class SmartRef {
    T value;
public:
    SmartRef(T v) : value(v) {}
    T get() const { return value; }
    void print() const { std::cout << "Value: " << value; }
};

// Partial specialization for pointer types:
template<typename T>
class SmartRef<T*> {
    T* ptr;
public:
    SmartRef(T* p) : ptr(p) {}
    T* get() const { return ptr; }
    void print() const {
        if (ptr) std::cout << "Ptr to: " << *ptr;
        else     std::cout << "nullptr";
    }
};

SmartRef<int>  a(42);      // uses primary template
int x = 99;
SmartRef<int*> b(&x);      // uses pointer specialization
a.print(); // Output: Value: 42
b.print(); // Output: Ptr to: 99
```

---

### Q30. What is `std::is_same`, `std::is_integral`, and other type traits?

**Answer:**  
Type traits in `<type_traits>` are template metaprogramming utilities that provide compile-time information about types, enabling conditional code, SFINAE, and concept definitions.

```cpp
#include <type_traits>

template<typename T>
void analyze() {
    std::cout << "Type: ";
    if constexpr (std::is_integral_v<T>)       std::cout << "integral";
    else if constexpr (std::is_floating_point_v<T>) std::cout << "float";
    else if constexpr (std::is_pointer_v<T>)   std::cout << "pointer";
    else                                        std::cout << "other";
    std::cout << "\n";
    std::cout << "Sizeof: " << sizeof(T) << "\n";
    std::cout << "Same as int: " << std::is_same_v<T, int> << "\n";
}

analyze<int>();    // integral, 4, 1
analyze<double>(); // float, 8, 0
analyze<int*>();   // pointer, 8, 0
```

---

### Q31. What is `if constexpr` (C++17)?

**Answer:**  
`if constexpr` is a compile-time conditional that eliminates branches at compile time. Unlike a regular `if`, the discarded branch is not compiled — allowing branches with code that would otherwise fail to compile for certain types.

```cpp
template<typename T>
std::string toString(T val) {
    if constexpr (std::is_same_v<T, bool>)
        return val ? "true" : "false";
    else if constexpr (std::is_arithmetic_v<T>)
        return std::to_string(val);
    else if constexpr (std::is_convertible_v<T, std::string>)
        return std::string(val);
    else
        return "(unknown)";
}

std::cout << toString(true);      // Output: true
std::cout << toString(42);        // Output: 42
std::cout << toString(3.14);      // Output: 3.140000
std::cout << toString("hello");   // Output: hello
```

---

### Q32. What is template specialization vs. overloading for functions?

**Answer:**  
Function templates can be overloaded (multiple primary templates) but the specialization of function templates can have surprising interaction with overload resolution. It's generally better to use **overloading** for function templates rather than specialization.

```cpp
// Preferred: overloading
template<typename T> void process(T val)  { std::cout << "generic\n"; }
template<typename T> void process(T* val) { std::cout << "pointer\n"; }
void process(int val) { std::cout << "int\n"; } // non-template preferred

// Problematic: specializing after overloads exist
template<> void process(int* val) { std::cout << "int-ptr spec\n"; }

int x = 5;
process(x);   // Output: int (non-template)
process(&x);  // Output: int-ptr spec
process(3.0); // Output: generic
```

---

### Q33. What is explicit specialization of a member function?

**Answer:**  
Individual member functions of a class template can be fully specialized without specializing the entire class.

```cpp
template<typename T>
class Printer {
public:
    void print(T val) {
        std::cout << "Generic: " << val << "\n";
    }
};

// Specialize only the print function for bool:
template<>
void Printer<bool>::print(bool val) {
    std::cout << "Bool: " << (val ? "YES" : "NO") << "\n";
}

Printer<int>  pi; pi.print(42);    // Output: Generic: 42
Printer<bool> pb; pb.print(true);  // Output: Bool: YES
```

---

## Variadic Templates

---

### Q34. What are variadic templates (C++11)?

**Answer:**  
Variadic templates accept any number of template arguments (a parameter pack). They enable type-safe generic functions and classes that work with arbitrary numbers of arguments.

```cpp
// Base case (empty pack):
void print() { std::cout << "\n"; }

// Variadic recursive:
template<typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << " ";
    print(rest...); // recursive: strip first, continue with rest
}

print(1, 2.5, "hello", true);
// Output: 1 2.5 hello 1
```

---

### Q35. What is a fold expression (C++17)?

**Answer:**  
Fold expressions apply a binary operator over all elements of a parameter pack without explicit recursion. Much cleaner than recursive variadic templates.

```cpp
// Unary right fold: (args op ...)
template<typename... T>
auto sum(T... args) { return (... + args); }  // left fold

template<typename... T>
auto product(T... args) { return (args * ...); } // right fold

template<typename... T>
void printAll(T... args) {
    ((std::cout << args << " "), ...); // comma fold
    std::cout << "\n";
}

std::cout << sum(1, 2, 3, 4, 5);       // Output: 15
std::cout << product(1, 2, 3, 4);      // Output: 24
printAll("A", 42, 3.14, true);         // Output: A 42 3.14 1
```

---

### Q36. What is `sizeof...` with variadic templates?

**Answer:**  
`sizeof...(pack)` returns the number of arguments in a parameter pack at compile time.

```cpp
template<typename... Args>
void countArgs(Args... args) {
    std::cout << "Arg count: " << sizeof...(Args) << "\n"; // types
    std::cout << "Val count: " << sizeof...(args) << "\n"; // values
}

countArgs(1, 2.0, "three", true);
// Output: Arg count: 4 / Val count: 4

// Compile-time check:
template<typename... T>
auto safeAdd(T... vals) {
    static_assert(sizeof...(T) >= 2, "Need at least 2 arguments");
    return (... + vals);
}
```

---

### Q37. How do you implement a type-safe `printf` with variadic templates?

**Answer:**  

```cpp
// Base case: no more arguments
void safePrintf(const char* format) {
    while (*format) {
        if (*format == '%') throw std::runtime_error("Extra % in format");
        std::cout << *format++;
    }
}

template<typename T, typename... Args>
void safePrintf(const char* format, T val, Args... rest) {
    while (*format) {
        if (*format == '%') {
            std::cout << val;          // print current value
            safePrintf(format + 1, rest...); // recurse with rest
            return;
        }
        std::cout << *format++;
    }
}

safePrintf("Hello %, you are % years old!\n", "Alice", 30);
// Output: Hello Alice, you are 30 years old!
```

---

### Q38. What is a variadic class template? (tuple-like)

**Answer:**  
Variadic class templates can store or operate on an arbitrary mix of types.

```cpp
// Simple variadic holder (simplified tuple):
template<typename... Types>
struct TypeList {};

template<typename T, typename... Rest>
struct Head { using type = T; };

template<typename... Types>
using HeadType = typename Head<Types...>::type;

// Practical: count types matching a predicate
template<template<typename> class Pred, typename... Types>
struct CountIf {
    static constexpr int value = (0 + ... + (Pred<Types>::value ? 1 : 0));
};

using IntCount = CountIf<std::is_integral, int, double, char, float, long>;
static_assert(IntCount::value == 3); // int, char, long are integral
std::cout << IntCount::value;        // Output: 3
```

---

## Concepts & Constraints — C++20

---

### Q39. What are Concepts in C++20?

**Answer:**  
Concepts are named compile-time predicates that constrain template parameters. They replace complex SFINAE patterns with readable, expressive constraints. A concept is defined with `template<...> concept Name = requirement;`.

```cpp
#include <concepts>

// Define a concept:
template<typename T>
concept Numeric = std::is_arithmetic_v<T>;

// Use it to constrain a function:
template<Numeric T>
T square(T x) { return x * x; }

std::cout << square(5);      // OK: int is Numeric
std::cout << square(3.14);   // OK: double is Numeric
// square("hello");           // ERROR: std::string is not Numeric
```

---

### Q40. What are the ways to apply concepts to templates?

**Answer:**  
Four equivalent syntaxes for constraining templates with concepts:

```cpp
template<typename T>
concept Printable = requires(T t) { std::cout << t; };

// Syntax 1: concept in template parameter list
template<Printable T>
void show1(T val) { std::cout << val; }

// Syntax 2: trailing requires clause
template<typename T>
void show2(T val) requires Printable<T> { std::cout << val; }

// Syntax 3: abbreviated function template (C++20)
void show3(Printable auto val) { std::cout << val; }

// Syntax 4: enable_if style (old way — avoid in new code)
template<typename T, typename = std::enable_if_t<std::is_arithmetic_v<T>>>
void show4(T val) { std::cout << val; }

show1(42);    show2(3.14);    show3("hi");
```

---

### Q41. What is a `requires` expression?

**Answer:**  
A `requires` expression defines ad-hoc constraints by listing requirements a type must satisfy — valid expressions, nested requirements, and type requirements.

```cpp
template<typename T>
concept Container = requires(T c) {
    c.begin();                           // simple: must have begin()
    c.end();                             // must have end()
    { c.size() } -> std::convertible_to<std::size_t>; // type requirement
    typename T::value_type;             // type member requirement
};

template<Container C>
void printContainer(const C& c) {
    for (const auto& x : c) std::cout << x << " ";
}

printContainer(std::vector<int>{1, 2, 3}); // OK
printContainer(std::list<std::string>{"a", "b"}); // OK
// printContainer(42); // ERROR: int is not a Container
```

---

### Q42. What are standard library Concepts (C++20)?

**Answer:**  
`<concepts>` provides many ready-to-use concepts:

```cpp
#include <concepts>

template<std::integral T>
T gcd(T a, T b) { return b == 0 ? a : gcd(b, a % b); }

template<std::floating_point T>
T lerp(T a, T b, T t) { return a + t * (b - a); }

template<std::copyable T>
T duplicate(T val) { return val; }

template<std::equality_comparable T>
bool areEqual(T a, T b) { return a == b; }

std::cout << gcd(48, 18);          // Output: 6
std::cout << lerp(0.0, 10.0, 0.5); // Output: 5.0
std::cout << areEqual(42, 42);      // Output: 1
```

---

### Q43. How do Concepts improve error messages compared to SFINAE?

**Answer:**  
SFINAE errors are often long and cryptic. Concepts produce clear, targeted diagnostic messages explaining exactly which constraint was violated.

```cpp
// SFINAE error (cryptic):
template<typename T>
std::enable_if_t<std::is_arithmetic_v<T>> oldAdd(T a, T b);

// Concept error (clear):
template<typename T>
concept Arithmetic = std::is_arithmetic_v<T>;

template<Arithmetic T>
T newAdd(T a, T b) { return a + b; }

// oldAdd(std::string{"a"}, std::string{"b"});
// Error: long SFINAE substitution failure trace...

// newAdd(std::string{"a"}, std::string{"b"});
// Error: "the associated constraints are not satisfied"
//        "std::string does not satisfy 'Arithmetic'"
//        — immediately clear!
```

---

## Advanced Template Techniques

---

### Q44. What is template metaprogramming (TMP)?

**Answer:**  
Template metaprogramming uses templates to perform computations at compile time, producing values, types, or code. Results are embedded directly in the binary — no runtime overhead.

```cpp
// Compile-time factorial:
template<int N>
struct Factorial {
    static constexpr long long value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr long long value = 1; // base case
};

static_assert(Factorial<10>::value == 3628800);
std::cout << Factorial<10>::value; // Output: 3628800 — computed at compile time

// Compile-time power:
template<int Base, int Exp>
struct Power {
    static constexpr int value = Base * Power<Base, Exp - 1>::value;
};
template<int Base>
struct Power<Base, 0> { static constexpr int value = 1; };

std::cout << Power<2, 10>::value; // Output: 1024
```

---

### Q45. What is CRTP (Curiously Recurring Template Pattern)?

**Answer:**  
CRTP is when a class derives from a template instantiation of itself: `class D : public Base<D>`. It enables **static polymorphism** — virtual dispatch without vtable overhead.

```cpp
template<typename Derived>
class Shape {
public:
    double area() const {
        return static_cast<const Derived*>(this)->computeArea();
    }
    void print() const {
        std::cout << "Area: " << area() << "\n";
    }
};

class Circle : public Shape<Circle> {
    double radius;
public:
    Circle(double r) : radius(r) {}
    double computeArea() const { return 3.14159 * radius * radius; }
};

class Square : public Shape<Square> {
    double side;
public:
    Square(double s) : side(s) {}
    double computeArea() const { return side * side; }
};

Circle c(5.0); c.print(); // Output: Area: 78.5397 — no vtable!
Square s(4.0); s.print(); // Output: Area: 16
```

---

### Q46. What is the Mixin pattern with CRTP?

**Answer:**  
CRTP can inject functionality into derived classes by providing reusable behaviors through base class templates.

```cpp
// Mixin: adds comparison operators from just operator<
template<typename T>
class Comparable {
public:
    bool operator>(const T& o)  const { return o < static_cast<const T&>(*this); }
    bool operator<=(const T& o) const { return !(static_cast<const T&>(*this) > o); }
    bool operator>=(const T& o) const { return !(static_cast<const T&>(*this) < o); }
    bool operator==(const T& o) const { return !(static_cast<const T&>(*this) < o) && !(o < static_cast<const T&>(*this)); }
};

class Weight : public Comparable<Weight> {
    double kg;
public:
    Weight(double k) : kg(k) {}
    bool operator<(const Weight& o) const { return kg < o.kg; }
};

Weight a(70.5), b(80.0);
std::cout << (a < b);  // Output: 1
std::cout << (a > b);  // Output: 0  — injected by Comparable
std::cout << (a == a); // Output: 1  — injected by Comparable
```

---

### Q47. What is a policy-based design with templates?

**Answer:**  
Policy-based design uses template parameters (policies) to inject interchangeable behaviors into a class, enabling compile-time strategy selection without virtual dispatch.

```cpp
// Policies:
struct LogToConsole {
    static void log(const std::string& msg) { std::cout << "[LOG] " << msg << "\n"; }
};
struct LogToFile {
    static void log(const std::string& msg) { /* write to file */ }
};
struct NoLog {
    static void log(const std::string&) {} // no-op
};

// Policy-based class:
template<typename LogPolicy = LogToConsole>
class DataProcessor {
public:
    void process(const std::string& data) {
        LogPolicy::log("Processing: " + data);
        // ... actual processing ...
    }
};

DataProcessor<>           prod;    // default: console logging
DataProcessor<LogToFile>  file;    // file logging
DataProcessor<NoLog>      silent;  // no logging — zero overhead

prod.process("dataset1");   // Output: [LOG] Processing: dataset1
silent.process("dataset2"); // no output, no overhead
```

---

### Q48. What is expression templates?

**Answer:**  
Expression templates are an advanced TMP technique that represents arithmetic expressions as type trees, enabling lazy evaluation and eliminating temporary objects. Used in high-performance math libraries (Eigen, Blaze).

```cpp
// Simplified expression template for vector addition:
template<typename L, typename R>
struct AddExpr {
    const L& lhs;
    const R& rhs;
    AddExpr(const L& l, const R& r) : lhs(l), rhs(r) {}
    double operator[](size_t i) const { return lhs[i] + rhs[i]; }
    size_t size() const { return lhs.size(); }
};

struct Vec {
    std::vector<double> data;
    Vec(std::initializer_list<double> l) : data(l) {}
    double operator[](size_t i) const { return data[i]; }
    size_t size() const { return data.size(); }

    template<typename Expr>
    Vec(const Expr& e) : data(e.size()) {
        for (size_t i = 0; i < data.size(); ++i) data[i] = e[i];
    }
};

template<typename L, typename R>
AddExpr<L,R> operator+(const L& l, const R& r) { return {l, r}; }

Vec a{1.0, 2.0, 3.0}, b{4.0, 5.0, 6.0}, c{7.0, 8.0, 9.0};
Vec result = a + b + c; // No temporaries! a+b+c is one lazy expression
for (auto x : result.data) std::cout << x << " "; // Output: 12 15 18
```

---

### Q49. What is `std::void_t` and detection idiom (C++17)?

**Answer:**  
`std::void_t<T>` maps any valid type expression to `void`. Used with SFINAE to detect whether a type has a member function, typedef, or other property.

```cpp
#include <type_traits>

// Primary template: T does not have a serialize() method
template<typename T, typename = void>
struct has_serialize : std::false_type {};

// Specialization: T has serialize()
template<typename T>
struct has_serialize<T, std::void_t<decltype(std::declval<T>().serialize())>>
    : std::true_type {};

struct WithSerialize {
    std::string serialize() { return "data"; }
};
struct WithoutSerialize {};

static_assert(has_serialize<WithSerialize>::value);
static_assert(!has_serialize<WithoutSerialize>::value);

template<typename T>
void saveObject(const T& obj) {
    if constexpr (has_serialize<T>::value)
        std::cout << "Saving: " << obj.serialize() << "\n";
    else
        std::cout << "Object not serializable\n";
}
```

---

### Q50. What are the best practices for writing templates in C++?

**Answer:**  

1. **Prefer Concepts (C++20) over SFINAE** — clearer constraints and better error messages.
2. **Use `if constexpr` over tag dispatch or SFINAE** for compile-time branching in C++17+.
3. **Mark move constructors/assignments `noexcept`** in template classes.
4. **Follow the Rule of Zero** in template classes — use smart pointers and standard containers.
5. **Define templates in headers** — the compiler needs the full definition at instantiation time.
6. **Use `typename` for dependent types** and `template` for dependent member templates.
7. **Prefer `auto` return types** over complex trailing return types when possible.
8. **Use `std::forward` in forwarding templates**, `std::move` elsewhere.
9. **Keep template interfaces minimal** — only constrain what you actually need.
10. **Test with multiple types** — compile with `int`, `double`, `std::string`, and custom types.

```cpp
// Example of well-written C++20 template:
template<typename T>
concept Summable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

template<Summable T>
class Accumulator {
    T total{};
public:
    void add(T val) { total = total + val; }
    T result() const { return total; }
};

Accumulator<int>         ia; ia.add(1); ia.add(2); ia.add(3);
Accumulator<std::string> sa; sa.add("Hello "); sa.add("World");

std::cout << ia.result() << "\n"; // Output: 6
std::cout << sa.result() << "\n"; // Output: Hello World
```

---

## Quick Reference

```cpp
// --- Template declarations ---
template<typename T>              // type parameter
template<typename T, typename U>  // multiple type params
template<typename T, int N>       // type + non-type
template<typename T = int>        // default type
template<template<typename> class C> // template template param
template<typename... Args>        // variadic

// --- Key techniques ---
// SFINAE:    std::enable_if_t<condition, T>
// C++17:     if constexpr (condition) { ... }
// C++20:     template<MyConcept T> / requires clause

// --- Type traits (common) ---
std::is_integral_v<T>         std::is_floating_point_v<T>
std::is_pointer_v<T>          std::is_same_v<T, U>
std::is_base_of_v<Base, Der>  std::is_convertible_v<From, To>
std::remove_reference_t<T>    std::decay_t<T>
std::conditional_t<B, T, F>   std::enable_if_t<B, T>

// --- Concepts (C++20 standard) ---
std::integral       std::floating_point   std::arithmetic
std::copyable       std::movable          std::default_initializable
std::equality_comparable                  std::totally_ordered
std::invocable<F, Args...>               std::same_as<T, U>

// --- Fold expressions (C++17) ---
(... op pack)   // left fold
(pack op ...)   // right fold
(init op ... op pack) // left fold with init
(pack op ... op init) // right fold with init

// --- CRTP pattern ---
template<typename Derived>
class Base { void func() { static_cast<Derived*>(this)->impl(); } };
class Child : public Base<Child> { void impl() { ... } };
```

---

*End of Top 50 C++ Templates Interview Questions*
