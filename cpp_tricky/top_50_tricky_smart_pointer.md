Here are **50 tricky C++ interview questions** focusing on **smart pointers (`unique_ptr`, `shared_ptr`, `weak_ptr`), `std::move`, move semantics, rvalue references, and related pitfalls**, with answers and code examples.

---

## 1. What problem do smart pointers solve?

**Answer:**  
Automatic memory management — prevent leaks, double deletions, and dangling pointers.

---

## 2. What is `std::unique_ptr`?

**Answer:**  
Exclusive ownership smart pointer. Cannot be copied, only moved.

```cpp
std::unique_ptr<int> p1(new int(5));
std::unique_ptr<int> p2 = std::move(p1); // OK
// std::unique_ptr<int> p3 = p1; // Error
```

---

## 3. What happens when a `unique_ptr` goes out of scope?

**Answer:**  
Its destructor deletes the managed object.

---

## 4. Can two `shared_ptr` manage same object?

**Answer:**  
Yes — reference counting.

```cpp
std::shared_ptr<int> p1(new int(10));
std::shared_ptr<int> p2 = p1; // ref count = 2
```

---

## 5. What is a `weak_ptr`?

**Answer:**  
Non-owning observer of `shared_ptr`; prevents circular references.

```cpp
std::weak_ptr<int> wp = p1;
if (auto sp = wp.lock()) { /* use sp */ }
```

---

## 6. Output:

```cpp
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;
sp.reset();
if (wp.expired()) cout << "expired";
```

**Answer:**  
`expired`

---

## 7. What is circular reference with `shared_ptr`?

**Answer:**  
Two objects hold `shared_ptr` to each other → never deleted.

```cpp
struct A { std::shared_ptr<B> b; };
struct B { std::shared_ptr<A> a; };
```

**Fix:** Use `weak_ptr` in one side.

---

## 8. What is `std::make_unique` vs `new`?

**Answer:**  
`make_unique` is exception-safe, cleaner.

```cpp
auto p = std::make_unique<int>(10); // preferred
std::unique_ptr<int> p2(new int(10));
```

---

## 9. What is `std::make_shared` advantage?

**Answer:**  
Single allocation for control block + object; faster, exception-safe.

---

## 10. Can you put a `unique_ptr` in a container?

**Answer:**  
Yes, but cannot copy; must move.

```cpp
std::vector<std::unique_ptr<int>> v;
v.push_back(std::make_unique<int>(5));
```

---

## 11. What is `std::move`?

**Answer:**  
Cast to rvalue; enables move semantics but doesn't move anything by itself.

---

## 12. Output:

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);
std::cout << s1.size();
```

**Answer:**  
`0` (moved-from string is valid but unspecified; typically empty).

---

## 13. What happens after moving from an object?

**Answer:**  
Object is in valid but unspecified state. Only safe to destroy or assign new value.

---

## 14. What is a move constructor?

**Answer:**  
Transfers ownership from rvalue instead of copying.

```cpp
A(A&& other) noexcept : data(other.data) { other.data = nullptr; }
```

---

## 15. Why mark move constructor `noexcept`?

**Answer:**  
So containers (`vector`) can use strong exception guarantee and actually move instead of copy.

---

## 16. Can `unique_ptr` be returned from a function?

**Answer:**  
Yes — move happens automatically (RVO + move).

```cpp
std::unique_ptr<int> f() {
    auto p = std::make_unique<int>(10);
    return p; // moves, no std::move needed
}
```

---

## 17. What is the problem with `auto_ptr`?

**Answer:**  
Deprecated — copy actually moves (broken semantics).

---

## 18. Can `shared_ptr` manage an array?

**Answer:**  
Yes, with `shared_ptr<int[]>`. Better: `vector`.

```cpp
std::shared_ptr<int[]> sp(new int[10]);
```

---

## 19. What is a custom deleter for `unique_ptr`?

**Answer:**  
Function or lambda called on destruction.

```cpp
auto deleter = [](FILE* f) { fclose(f); };
std::unique_ptr<FILE, decltype(deleter)> fp(fopen("x.txt", "r"), deleter);
```

---

## 20. What is `std::enable_shared_from_this`?

**Answer:**  
Allows a class to safely return a `shared_ptr` to `this`.

```cpp
class A : public std::enable_shared_from_this<A> {
public:
    std::shared_ptr<A> getptr() { return shared_from_this(); }
};
```

---

## 21. Can `shared_from_this` be called from constructor?

**Answer:**  
No — object not yet owned by `shared_ptr`.

---

## 22. Output:

```cpp
struct A { int x; };
int main() {
    std::unique_ptr<A> p1(new A{10});
    A* p2 = p1.get();
    p1.reset();
    std::cout << p2->x;
}
```

**Answer:**  
Undefined behavior — dangling pointer.

---

## 23. What is `std::unique_ptr::release()`?

**Answer:**  
Releases ownership, returns raw pointer, does not delete.

```cpp
auto p = std::make_unique<int>(5);
int* raw = p.release();
delete raw;
```

---

## 24. Difference between `reset()` and `release()`?

**Answer:**  
`reset()` deletes object then optionally takes new one.  
`release()` gives up ownership without deletion.

---

## 25. What is `std::move_if_noexcept`?

**Answer:**  
Moves if move ctor is `noexcept`, else copies (used internally by containers).

---

## 26. Can a `shared_ptr` be created from `this` raw pointer?

**Answer:**  
Never — leads to double deletion. Use `enable_shared_from_this`.

---

## 27. Output:

```cpp
auto sp1 = std::make_shared<int>(100);
auto sp2(sp1);
std::cout << sp1.use_count();
```

**Answer:**  
`2`

---

## 28. What is atomic `shared_ptr`?

**Answer:**  
C++20: `std::atomic<std::shared_ptr<T>>` for thread-safe access.

---

## 29. Is `shared_ptr` thread-safe?

**Answer:**  
Control block (ref count) is thread-safe, but pointed object is not.

---

## 30. Can you `std::move` a `shared_ptr`?

**Answer:**  
Yes — moves ownership, ref count unchanged.

```cpp
shared_ptr<int> p1 = make_shared<int>(1);
shared_ptr<int> p2 = std::move(p1); // p1 now null
```

---

## 31. What is the overhead of `shared_ptr`?

**Answer:**  
Two pointers (object + control block) plus atomic ref count operations.

---

## 32. Why prefer `make_unique` over `unique_ptr{new T}`?

**Answer:**  
Exception safety: avoids leak if other arguments throw.

---

## 33. Output:

```cpp
struct Foo { ~Foo() { cout << "~"; } };
int main() {
    std::shared_ptr<Foo> sp(new Foo());
    sp.reset();
    cout << "end";
}
```

**Answer:**  
`~end`

---

## 34. Can we use `std::move` on `const` object?

**Answer:**  
Yes, but it copies (const prevents modification).

---

## 35. What is a forwarding (universal) reference?

**Answer:**  
`T&&` with template type deduction.

```cpp
template<typename T>
void f(T&& arg) { /* perfect forwarding */ }
```

---

## 36. What is `std::forward` used for?

**Answer:**  
Preserves value category (lvalue/rvalue) for perfect forwarding.

---

## 37. Is `std::move` the same as rvalue cast?

**Answer:**  
Yes — `std::move(x)` is essentially `static_cast<T&&>(x)`.

---

## 38. Can `unique_ptr` be stored in a `vector` and sorted?

**Answer:**  
Yes, but must move, not copy.

```cpp
std::vector<std::unique_ptr<int>> v;
std::sort(v.begin(), v.end(), 
    [](const auto& a, const auto& b) { return *a < *b; });
```

---

## 39. What is `std::unique_ptr<T[]>`?

**Answer:**  
Specialization for arrays: calls `delete[]`.

```cpp
std::unique_ptr<int[]> arr(new int[10]);
arr[0] = 5;
```

---

## 40. Why use `std::shared_ptr<void>`?

**Answer:**  
Type-erased ownership (rare, but possible).

---

## 41. What is `std::owner_less`?

**Answer:**  
Compares `shared_ptr`/`weak_ptr` by owner (not value) for associative containers.

---

## 42. Can you have `weak_ptr` to `unique_ptr`?

**Answer:**  
No — `weak_ptr` only works with `shared_ptr`.

---

## 43. Output:

```cpp
std::unique_ptr<int> p(nullptr);
if (!p) cout << "empty";
```

**Answer:**  
`empty` — `operator bool` returns `false` if null.

---

## 44. What is dangling `weak_ptr`?

**Answer:**  
`weak_ptr` whose `shared_ptr` expired — `lock()` returns null.

---

## 45. What is `std::make_unique` for arrays (C++20)?

**Answer:**  
`auto p = std::make_unique<int[]>(10);` — value-initializes.

---

## 46. Can a move constructor be `private`?

**Answer:**  
Yes — prevents moving outside class.

---

## 47. Why is moved-from object still destructible?

**Answer:**  
Destructor must still run; moved-from state must be valid.

---

## 48. What is the rule of five?

**Answer:**  
If you define one of: destructor, copy ctor, copy assign, move ctor, move assign — define all or `=default`.

---

## 49. What is `std::move` on `std::shared_ptr` cost?

**Answer:**  
Very cheap: just swaps pointers, no atomic ref count change.

---

## 50. Is this safe?

```cpp
void f(std::unique_ptr<int> p, int x);
int* q = new int(2);
f(std::unique_ptr<int>(q), *q);
```

**Answer:**  
Unsafe — order of evaluation may cause dereference after move.

**Fix:** `int val = *q; f(std::unique_ptr<int>(q), val);`

---

Let me know if you want a combined document covering both constructor/destructor and smart pointers, or a quiz format.