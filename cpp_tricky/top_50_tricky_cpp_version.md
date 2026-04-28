Here are **50 tricky C++ interview questions** focusing on **Modern C++ Features (C++11/14/17/20)** — covering `auto`, `nullptr`, range-based for, `constexpr`, `std::optional`, `std::variant`, structured bindings, and more — with answers and code examples.

---

## 1. What is the difference between `auto` and `decltype`?

**Answer:**  
`auto` deduces type from initializer, removing references and `const`. `decltype` returns exact declared type.

```cpp
int x = 5, &rx = x;
auto a = rx;    // int
decltype(rx) d = rx; // int&
```

---

## 2. What is `nullptr`? Why not `NULL` or `0`?

**Answer:**  
`nullptr` is type-safe null pointer constant of type `std::nullptr_t`. `NULL` can be `0` → ambiguous overloads.

```cpp
void f(int);
void f(char*);
f(NULL);    // calls f(int) — error prone
f(nullptr); // calls f(char*)
```

---

## 3. What is the output of this range-based for loop?

```cpp
std::vector<int> v{1,2,3};
for (auto x : v) x *= 2;
for (auto x : v) cout << x;
```

**Answer:**  
`123` — loop iterates over copies, not references.

**Fix:** `for (auto& x : v) x *= 2;`

---

## 4. Can you modify a container while iterating with range-based for?

**Answer:**  
Dangerous — invalidating iterators leads to UB. Resizing `vector` invalidates.

---

## 5. When should you use `constexpr` vs `const`?

**Answer:**  
`constexpr` guarantees compile-time evaluation (for constants, functions, constructors). `const` only read-only runtime.

```cpp
constexpr int square(int x) { return x*x; }
int arr[square(5)]; // OK (compile-time)
```

---

## 6. What are `constexpr` limitations in C++11 vs C++14?

**Answer:**  
C++11: single return, no loops, no mutation. C++14: loops, multiple statements, mutation.

```cpp
// C++14 constexpr
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) result *= i;
    return result;
}
```

---

## 7. What is `std::optional`? When to use?

**Answer:**  
Wrapper for "nullable" value without sentinel (like `-1`). C++17.

```cpp
std::optional<int> find(string s) {
    if (s.empty()) return std::nullopt;
    return s.size();
}
if (auto len = find("hi")) cout << *len;
```

---

## 8. What is `std::optional::value_or()`?

**Answer:**  
Returns value if present, otherwise default.

```cpp
cout << find("").value_or(0); // 0
```

---

## 9. What is the output?

```cpp
std::optional<int> o;
cout << o.has_value();
o = 10;
cout << o.value();
```

**Answer:**  
`010` (false printed as 0, then 10)

---

## 10. What is `std::variant`? Type-safe union?

**Answer:**  
Holds one of several types, knows active type. C++17.

```cpp
std::variant<int, float, string> v = "hello";
v = 42;
cout << std::get<int>(v);
```

---

## 11. What happens if `std::get` with wrong type in `variant`?

**Answer:**  
Throws `std::bad_variant_access`.

---

## 12. What is `std::visit`? When used?

**Answer:**  
Applies visitor to active member of `variant`.

```cpp
std::visit([](auto&& arg) { cout << arg; }, v);
```

---

## 13. What is structured binding? Example.

**Answer:**  
Unpacks struct/tuple/pair into separate variables. C++17.

```cpp
std::map<int, string> m = {{1,"a"}};
for (auto [key, val] : m) cout << key << val;
```

---

## 14. Output of this structured binding with modifiers?

```cpp
std::pair<int, int> p{1,2};
auto& [a,b] = p;
a = 10;
cout << p.first;
```

**Answer:**  
`10` — binding references modify original.

---

## 15. Can you apply structured binding to `std::array`?

**Answer:**  
Yes — works with aggregates.

```cpp
std::array<int,3> arr{1,2,3};
auto [x,y,z] = arr; // x=1,y=2,z=3
```

---

## 16. What is `std::string_view`? Why use?

**Answer:**  
Non-owning view of string, avoids copy. C++17.

```cpp
void print(std::string_view sv) { cout << sv; }
print("hello"); // no temporary string
```

---

## 17. What is the danger of `string_view`?

**Answer:**  
Dangling view if original string destroyed.

```cpp
std::string_view sv = std::string("temp"); // dangling
```

---

## 18. What is `std::any`? When to use?

**Answer:**  
Type-safe container for single value of any type. C++17.

```cpp
std::any a = 42;
cout << std::any_cast<int>(a);
```

---

## 19. What happens if `std::any_cast` wrong type?

**Answer:**  
Throws `std::bad_any_cast`.

---

## 20. What is `std::byte`? Why not `unsigned char`?

**Answer:**  
Safer for raw memory — no arithmetic, only bitwise ops. C++17.

```cpp
std::byte b{0x0F};
b <<= 1; // OK
// b = b + 1; // error
```

---

## 21. What is a lambda capture `[=, this]` (C++20)?

**Answer:**  
Explicit `this` capture required for clarity, avoids implicit `this` in `[=]`.

```cpp
[=, this] { return member; }; // OK in C++20
```

---

## 22. What is generic lambda?

**Answer:**  
`auto` parameters (C++14) — templated lambda.

```cpp
auto add = [](auto a, auto b) { return a + b; };
cout << add(1,2) << add(1.5,0.5);
```

---

## 23. What is init-capture in lambda (C++14)?

**Answer:**  
Capture with initialization — moves-only types.

```cpp
auto p = std::make_unique<int>(10);
auto func = [p = std::move(p)] { return *p; };
```

---

## 24. What is `decltype(auto)`? When needed?

**Answer:**  
Deduces exact type including references — for perfect forwarding.

```cpp
int x = 5, &rx = x;
decltype(auto) a = rx; // int&
auto b = rx; // int
```

---

## 25. What's the output?

```cpp
auto f() { int x = 10; return x; }
decltype(auto) g() { int x = 10; return x; }
// both return int
decltype(auto) h() { int x = 10; return (x); } // returns int& (dangling)
```

---

## 26. What is `std::make_unique` vs `std::unique_ptr{new T}`?

**Answer:**  
`make_unique` exception-safe and more concise.

```cpp
auto p = std::make_unique<int>(10); // preferred
```

---

## 27. What is `std::make_shared` advantage over `shared_ptr{new T}`?

**Answer:**  
Single allocation (control block + object) → fewer cache misses.

---

## 28. What is `std::invoke` (C++17)?

**Answer:**  
Unified call for functions, member pointers, lambdas.

```cpp
std::invoke(&std::string::size, "hello"); // calls member
```

---

## 29. What is `constinit` (C++20)?

**Answer:**  
Guarantees constant initialization (before dynamic init), avoids static init order fiasco.

```cpp
constinit int x = 42; // guaranteed init at compile-time
```

---

## 30. What is `consteval` (C++20)?

**Answer:**  
Immediate function — must be evaluated at compile-time.

```cpp
consteval int sq(int x) { return x*x; }
constexpr int y = sq(5); // OK
int z = sq(5); // error (must be constexpr context)
```

---

## 31. What is `std::span` (C++20)?

**Answer:**  
Non-owning view of contiguous sequence (like `string_view` for arrays/vectors).

```cpp
void print(std::span<int> sp) { for (int x : sp) cout << x; }
std::vector<int> v{1,2,3};
print(v);
print({v.data(), 2});
```

---

## 32. What is `std::to_chars` (C++17)? Benefit over `sprintf`?

**Answer:**  
Locale-independent, fastest conversion, no allocation.

```cpp
char buf[10];
auto [ptr, ec] = std::to_chars(buf, buf+10, 42);
std::string_view result(buf, ptr - buf);
```

---

## 33. What is `if constexpr`? Why compile-time branching?

**Answer:**  
Template-only compile-time condition — discards false branch.

```cpp
template<typename T>
auto length(T& t) {
    if constexpr (std::is_array_v<T>) return sizeof(t);
    else return t.size(); // only compiled if T has size()
}
```

---

## 34. Output of this `constexpr` lambda (C++17)?

```cpp
constexpr auto f = [](int x) { return x*x; };
int arr[f(5)]; // OK
```

**Answer:**  
Lambda is `constexpr` by default in C++17 if eligible.

---

## 35. What is `[[nodiscard]]`? When used?

**Answer:**  
Compiler warning if return value ignored. C++17.

```cpp
[[nodiscard]] int allocate() { return 0; }
allocate(); // warning
```

---

## 36. What is `[[maybe_unused]]`?

**Answer:**  
Suppresses unused variable/parameter warnings.

```cpp
[[maybe_unused]] int debug = 10;
```

---

## 37. What is `std::gcd` and `std::lcm` (C++17)?

**Answer:**  
Greatest common divisor / least common multiple in `<numeric>`.

```cpp
cout << std::gcd(12, 18); // 6
cout << std::lcm(12, 18); // 36
```

---

## 38. What is `std::clamp` (C++17)?

**Answer:**  
Clamps value between boundaries.

```cpp
int x = 50;
cout << std::clamp(x, 0, 100); // 50
cout << std::clamp(x, 60, 100); // 60 (clamped)
```

---

## 39. What is `std::size` (C++17)?

**Answer:**  
Unified size for containers and built-in arrays.

```cpp
int arr[10];
std::vector<int> v(10);
cout << std::size(arr); // 10
cout << std::size(v); // 10
```

---

## 40. What is `std::as_const` (C++17)?

**Answer:**  
Convenience to add `const` qualification.

```cpp
auto& constRef = std::as_const(obj);
```

---

## 41. What is `std::midpoint` (C++20)?

**Answer:**  
Safely computes midpoint without overflow.

```cpp
std::midpoint(10, 20); // 15
std::midpoint(1, 9999999999LL); // no overflow
```

---

## 42. What is `std::bind_front` (C++20)?

**Answer:**  
Partial function application (better than `std::bind`).

```cpp
auto add = [](int a, int b) { return a + b; };
auto inc = std::bind_front(add, 1);
cout << inc(5); // 6
```

---

## 43. What is designator initializer (C++20)?

**Answer:**  
Named initializers for aggregates.

```cpp
struct Point { int x, y, z; };
Point p = { .x = 1, .z = 3 }; // y = 0
```

---

## 44. What is `using enum` (C++20)?

**Answer:**  
Brings enum enumerators into scope.

```cpp
enum class Color { Red, Green, Blue };
void f() {
    using enum Color;
    Color c = Red; // instead of Color::Red
}
```

---

## 45. What is `std::string::starts_with`, `ends_with` (C++20)?

**Answer:**  
String prefix/suffix checks.

```cpp
std::string s = "hello world";
s.starts_with("hello"); // true
s.ends_with("world"); // true
```

---

## 46. What is `std::remove_cvref` (C++20)?

**Answer:**  
Removes `const`, `volatile`, and reference.

```cpp
std::remove_cvref_t<const int&> // int
```

---

## 47. What is `std::bit_cast` (C++20)?

**Answer:**  
Type-punning without UB (strict-aliasing safe).

```cpp
float f = 3.14;
int i = std::bit_cast<int>(f); // reinterprets bits
```

---

## 48. Range-based for with initializer (C++20)?

**Answer:**  
Initializer before loop.

```cpp
for (std::vector v = getVec(); auto& e : v) { ... }
```

---

## 49. What is `std::source_location` (C++20)?

**Answer:**  
Replaces `__FILE__`, `__LINE__` with type-safe equivalent.

```cpp
void log(std::string_view msg, 
         std::source_location loc = std::source_location::current()) {
    cout << loc.file_name() << ":" << loc.line() << " " << msg;
}
```

---

## 50. Output of `char8_t` (C++20) pitfall?

```cpp
const char8_t* u8str = u8"hello";
std::cout << u8str; // error: no overload (char8_t not convertible to char)
```

**Answer:**  
`char8_t` distinct type → `std::cout` won't accept. Need reinterpret or convert.

---

Let me know if you'd like the full **400+ questions** across all 8 topics compiled into a single document, categorized for interview prep or self-testing.