# C++ Smart Pointers — Interview Prep

> Complete reference for `unique_ptr`, `shared_ptr`, and `weak_ptr` — concepts, Q&A, when to use each, and common traps.

---

## The Three Smart Pointers

### `unique_ptr` — Exclusive ownership, zero overhead

Sole owner of a heap resource. When it goes out of scope the resource is destroyed automatically. Cannot be copied — only moved.

**Overhead:** Zero. Same size as a raw pointer. Destructor is inlined at compile time.

```cpp
// Basic usage
auto p = std::make_unique<Widget>(42);
p->doSomething();
// destroyed automatically here

// Transfer ownership (move only — no copy)
auto p2 = std::move(p);  // p is now null

// Custom deleter (e.g. for FILE*)
auto f = std::unique_ptr<FILE, decltype(&fclose)>(
    fopen("data.txt", "r"), fclose
);
```

> **Tip:** Always prefer `make_unique` over `new` — it's exception-safe and avoids raw `new` entirely.

---

### `shared_ptr` — Shared ownership, reference counted

Multiple owners share a resource via a control block that tracks the reference count. Resource is destroyed when the last `shared_ptr` to it is destroyed.

**Overhead:** Two pointers (object + control block), atomic ref-count increment/decrement. Costs real CPU cycles.

```cpp
auto a = std::make_shared<Widget>(42);
auto b = a;  // ref count = 2

std::cout << a.use_count();  // 2

b.reset();   // ref count = 1
// Widget destroyed when 'a' goes out of scope

// Passing without sharing ownership
void process(const std::shared_ptr<Widget>& w);  // doesn't bump count
void observe(std::weak_ptr<Widget> w);            // no ownership
```

> **Tip:** Passing `shared_ptr` by value copies it and bumps the refcount — expensive in hot paths. Pass by `const&` when you don't need to extend lifetime.

---

### `weak_ptr` — Non-owning observer, breaks cycles

Observes a `shared_ptr`-managed object without participating in ownership. Must be `lock()`ed to a temporary `shared_ptr` before use — this is how you safely check if the object still exists.

```cpp
auto shared = std::make_shared<Widget>(42);
std::weak_ptr<Widget> weak = shared;

// Safe access
if (auto locked = weak.lock()) {
    locked->doSomething();  // safe: object still alive
} else {
    // object was already destroyed
}

// Classic use: breaking parent-child cycle
struct Node {
    std::shared_ptr<Node> child;
    std::weak_ptr<Node>   parent;  // weak! avoids cycle
};
```

> **Tip:** `weak_ptr::lock()` is the ONLY safe way to access the object. Never dereference a raw pointer extracted from a `weak_ptr`.

---

## When to Use Which

| Pointer | Use when… | Avoid when… |
|---|---|---|
| `unique_ptr` | One clear owner. Factory return types. Class members that own resources. Polymorphic hierarchies. | You need to share across multiple owners or threads without copies. |
| `shared_ptr` | Genuinely shared ownership. Object lifetime tied to multiple users (caches, event systems, async callbacks). | You only have one owner — reach for `unique_ptr` first. Hot paths where atomic refcounting is too expensive. |
| `weak_ptr` | Breaking `shared_ptr` cycles. Caches that don't prevent eviction. Observers in pub-sub systems. | You need guaranteed access — `lock()` can return `nullptr`. |
| Raw pointer / ref | Non-owning function parameters. C API interop (wrap at boundary). Performance-critical internals. | Any heap allocation, ownership transfer, or uncertain lifetime. |

---

## Interview Q&A

### Q1. What is a smart pointer and why do we need them? `[Easy]`

A smart pointer is an RAII wrapper around a raw pointer that automatically manages the lifetime of the heap object it owns. When it goes out of scope — whether normally or via an exception — its destructor runs and frees the resource.

Raw pointers have no ownership semantics: it's unclear who is responsible for `delete`, double-deletes and leaks happen easily, and exceptions make cleanup paths hard to get right.

```cpp
// Raw pointer — who deletes this?
Widget* w = new Widget();
process(w);           // throws? leak.
if (early_exit) return; // leak.
delete w;

// Smart pointer — always safe
auto w = std::make_unique<Widget>();
process(*w);          // throws? destructor runs, no leak.
```

---

### Q2. What is the difference between `unique_ptr` and `shared_ptr`? `[Easy]`

`unique_ptr` — exclusive ownership, move-only, zero overhead.  
`shared_ptr` — shared ownership, copyable, reference-counted with a heap-allocated control block.

```cpp
// unique_ptr: move-only
auto u = std::make_unique<int>(1);
auto u2 = u;            // ERROR: copy deleted
auto u2 = std::move(u); // OK: transfer ownership

// shared_ptr: copyable
auto s = std::make_shared<int>(1);
auto s2 = s;            // OK: ref count = 2
std::cout << s.use_count(); // 2
```

> **Interview tip:** "I default to `unique_ptr`. I only reach for `shared_ptr` when I genuinely need multiple owners."

---

### Q3. Why use `make_unique` / `make_shared` instead of `new`? `[Easy]`

Three reasons:

1. **Exception safety.** The compiler may interleave allocations in function call arguments. If one throws, a raw `new` can leak before the smart pointer takes ownership.
2. **Single allocation for `shared_ptr`.** `make_shared` allocates the object and control block together — one allocation instead of two, better cache locality.
3. **No raw `new`.** Avoids the type name appearing twice and removes any possibility of accidentally using `delete`.

```cpp
// Dangerous: compiler may interleave these evaluations
foo(shared_ptr<Widget>(new Widget()), new Gadget()); // Gadget throws? Widget leaks.

// Safe: each make_* is a single atomic expression
foo(make_shared<Widget>(), make_unique<Gadget>());
```

---

### Q4. What is `weak_ptr` and when do you use it? `[Easy]`

`weak_ptr` is a non-owning observer of a `shared_ptr`-managed object. It does not increment the reference count, so it does not keep the object alive.

Use it when you need to refer to a shared object but must not extend its lifetime — and to break circular references between `shared_ptr`s which would otherwise cause a memory leak.

```cpp
// Cache that doesn't keep objects alive
class Cache {
    std::unordered_map<int, std::weak_ptr<Texture>> cache;
public:
    std::shared_ptr<Texture> get(int id) {
        if (auto sp = cache[id].lock())  // still alive?
            return sp;
        auto t = std::make_shared<Texture>(id);
        cache[id] = t;
        return t;
    }
};
```

---

### Q5. Can you explain the `shared_ptr` control block? `[Medium]`

A control block is a separate heap allocation created alongside the managed object. It contains:

- **Strong ref count** — number of `shared_ptr`s owning the object. Object is destroyed when it hits 0.
- **Weak ref count** — number of `weak_ptr`s + 1 (the +1 represents shared ownership). Control block is freed when this hits 0.
- **Deleter** — the function used to destroy the object (default is `delete`).
- **Allocator** — used to free the control block itself.

`make_shared` allocates object + control block in one contiguous chunk. `shared_ptr(new T)` requires two separate allocations.

```cpp
auto p1 = std::shared_ptr<Widget>(new Widget()); // two allocations
auto p2 = std::make_shared<Widget>();            // one allocation (preferred)
```

> **Gotcha:** With `make_shared`, the memory for the object is not freed until ALL `weak_ptr`s are gone too, because the control block and object share the same allocation.

---

### Q6. How do you pass smart pointers to functions? `[Medium]`

Pass the pointer type only when ownership transfer is the point of the call. Otherwise pass the underlying object.

```cpp
// WRONG — forces shared ownership on callers unnecessarily
void process(std::shared_ptr<Widget> w);   // copies, bumps refcount

// RIGHT — just using the object
void process(Widget& w);           // non-owning, can't be null
void process(const Widget& w);     // read-only
void process(Widget* w);           // non-owning, nullable

// RIGHT — transferring unique ownership into the function
void takeOwnership(std::unique_ptr<Widget> w);

// RIGHT — extending shared lifetime (only if function stores it)
void shareWith(std::shared_ptr<Widget> w);

// RIGHT — observing without affecting lifetime
void observe(std::weak_ptr<Widget> w);
```

> Herb Sutter's guideline: "Don't pass smart pointers unless you mean to transfer or share ownership."

---

### Q7. What is a circular reference and how do you fix it? `[Medium]`

A circular reference occurs when two or more `shared_ptr`s form a cycle. The reference counts never reach zero so the objects are never destroyed — a memory leak.

```cpp
// LEAK: circular reference
struct Child;
struct Parent {
    std::shared_ptr<Child> child;
};
struct Child {
    std::shared_ptr<Parent> parent;  // keeps Parent alive forever
};

auto p = std::make_shared<Parent>();
auto c = std::make_shared<Child>();
p->child  = c;  // ref count = 2
c->parent = p;  // ref count = 2
// p, c go out of scope: rc drops to 1 each — never freed!

// FIX: back-pointer is weak
struct Child {
    std::weak_ptr<Parent> parent;   // no ownership, no cycle
};
```

> **Rule:** In a parent-child or observer relationship, the child/observer uses `weak_ptr` to refer back to the owner.

---

### Q8. Is `shared_ptr` thread-safe? `[Medium]`

**Partially.** The reference count is atomically updated — it is safe to copy/destroy `shared_ptr` instances pointing to the same object from multiple threads.

However, the `shared_ptr` object itself is not thread-safe to read/write concurrently from different threads. And the managed object has no thread safety at all — that's your responsibility.

```cpp
// SAFE: independent copies, atomic refcount
std::shared_ptr<Widget> g = make_shared<Widget>();
thread t1([copy = g]{ copy->read(); });  // safe
thread t2([copy = g]{ copy->read(); });  // safe

// UNSAFE: two threads writing the same shared_ptr variable
std::shared_ptr<Widget> shared_var;
thread t1([&]{ shared_var = make_shared<Widget>(); }); // data race!
thread t2([&]{ shared_var.reset(); });                  // data race!

// FIX: use std::atomic<std::shared_ptr<Widget>> (C++20)
std::atomic<std::shared_ptr<Widget>> safe_var;
```

---

### Q9. What is `enable_shared_from_this`? `[Hard]`

A base class mixin that allows an object managed by `shared_ptr` to safely produce additional `shared_ptr`s to itself — without creating a new, disconnected control block that would double-delete the object.

```cpp
// WRONG — creates a second independent control block → double free
struct Widget {
    std::shared_ptr<Widget> getThis() {
        return std::shared_ptr<Widget>(this); // DISASTER
    }
};

// CORRECT — shares the existing control block
struct Widget : std::enable_shared_from_this<Widget> {
    std::shared_ptr<Widget> getThis() {
        return shared_from_this();
    }

    void asyncOp(Executor& ex) {
        ex.post([self = shared_from_this()] {
            self->doWork();  // object guaranteed alive during async work
        });
    }
};

// MUST be under shared_ptr management before calling shared_from_this()
auto w = std::make_shared<Widget>();
auto w2 = w->getThis();  // safe
```

> Calling `shared_from_this()` on a stack object or raw-new object that isn't yet owned by a `shared_ptr` throws `std::bad_weak_ptr`.

---

### Q10. Why prefer `unique_ptr` as a function return type over raw pointer? `[Medium]`

Returning `unique_ptr` makes ownership transfer explicit and impossible to ignore. A raw pointer return is ambiguous — is the caller supposed to delete it? Is it borrowed? `unique_ptr` is also implicitly convertible to `shared_ptr`, so callers who need shared ownership can still get it.

```cpp
// Ambiguous — caller is guessing
Widget* createWidget();

// Crystal clear — caller owns it
std::unique_ptr<Widget> createWidget();

// Caller can escalate to shared if needed
std::shared_ptr<Widget> sp = createWidget();  // implicit conversion

// Factory pattern
std::unique_ptr<Shape> makeShape(ShapeType t) {
    switch(t) {
        case CIRCLE:    return std::make_unique<Circle>();
        case RECTANGLE: return std::make_unique<Rectangle>();
    }
}
```

---

### Q11. What are the performance costs of `shared_ptr`? `[Hard]`

1. **Extra allocation.** `shared_ptr(new T)` does two heap allocations. `make_shared` does one but keeps both alive until all `weak_ptr`s die.
2. **Atomic reference counting.** Copy/destroy of `shared_ptr` does an atomic increment/decrement — not free, especially under contention on multi-core systems.
3. **Cache pressure.** Two pointer dereferences (object + control block) instead of one.
4. **Size.** `shared_ptr` is 16 bytes (two pointers). `unique_ptr` is 8 bytes (same as raw pointer).

```cpp
// Hot path — avoid unnecessary copies
void hotPath(const std::shared_ptr<Widget>& w) {  // by ref: NO atomic op
    w->compute();
}

void coldPath(std::shared_ptr<Widget> w) {  // by value: atomic inc + dec
    w->compute();
}
```

> In high-frequency code (game loops, network packet processing), consider intrusive reference counting or `unique_ptr` instead.

---

### Q12. When would you still use a raw pointer in modern C++? `[Hard]`

Raw pointers are still appropriate in three narrow cases:

1. **Non-owning observer.** When you just need to look at an object whose lifetime is guaranteed by the caller. A raw pointer or reference communicates "I don't own this."
2. **C API interop.** Wrap at the boundary with a smart pointer + custom deleter.
3. **Performance-critical internal implementation.** Inside a data structure where you control all lifetimes manually. The public API still uses smart pointers.

```cpp
// Non-owning observer — fine as raw pointer
void render(const Texture* tex);   // "I look but don't own"

// C API boundary — wrap immediately
auto handle = std::unique_ptr<SSL, decltype(&SSL_free)>(
    SSL_new(ctx), SSL_free
);

// Inside a low-level container
struct ListNode {
    ListNode* next;  // owner manages lifetimes externally
    int value;
};
```

> **Key insight:** raw pointer = "non-owning borrow". Smart pointer = "ownership". If you're not transferring or sharing ownership, a raw pointer or reference is the right signal.

---

## Common Traps

### Trap 1 — Calling `shared_from_this` before `shared_ptr` takes ownership

```cpp
struct Widget : enable_shared_from_this<Widget> {
    Widget() {
        auto s = shared_from_this(); // THROWS bad_weak_ptr!
        // No shared_ptr owns this yet during construction
    }
};
```

**Fix:** Never call `shared_from_this()` in the constructor. Use a static factory instead.

---

### Trap 2 — Storing raw `this` from a `shared_ptr`-managed object

```cpp
auto w = make_shared<Widget>();
Widget* raw = w.get();
w.reset();           // Widget destroyed
raw->doSomething();  // undefined behaviour — dangling pointer
```

**Fix:** Raw pointers extracted from smart pointers are only valid for the scope in which the smart pointer lives.

---

### Trap 3 — Wrapping the same raw pointer in two `shared_ptr`s

```cpp
Widget* raw = new Widget();
shared_ptr<Widget> p1(raw);
shared_ptr<Widget> p2(raw);  // two independent control blocks!
// p1 destroyed → Widget deleted
// p2 destroyed → Widget deleted AGAIN → double free / crash
```

**Fix:** Always use `make_shared`. If you must wrap a raw pointer, do it ONCE and only copy the `shared_ptr`.

---

### Trap 4 — Thinking `shared_ptr` makes your data thread-safe

```cpp
auto data = make_shared<std::vector<int>>();

thread t1([data]{ data->push_back(1); });  // modifies vector
thread t2([data]{ data->push_back(2); });  // modifies vector
// DATA RACE on the vector — shared_ptr only protects its refcount
// You still need a mutex to protect the managed object
```

**Fix:** `shared_ptr` gives you safe refcounting. It gives you nothing about the thread safety of the object it points to.

---

### Trap 5 — Calling `shared_from_this` on a stack or raw-new object

```cpp
Widget w_stack;
auto bad = w_stack.shared_from_this();  // throws bad_weak_ptr

Widget* raw = new Widget();
auto bad2 = raw->shared_from_this();    // throws bad_weak_ptr

// CORRECT — must be owned by shared_ptr first
auto good = std::make_shared<Widget>();
auto ok = good->shared_from_this();    // fine
```

---

## Quick Reference Cheat Sheet

```
unique_ptr   →  one owner, move-only, zero cost, default choice
shared_ptr   →  many owners, ref-counted, has overhead
weak_ptr     →  observer, no ownership, must lock() before use
raw ptr/ref  →  non-owning borrow, C interop, hot-path internals
```

### The golden rules

- Default to `unique_ptr`. Upgrade to `shared_ptr` only when you genuinely need multiple owners.
- Always use `make_unique` / `make_shared` — never raw `new`.
- Pass smart pointers only when ownership transfer is the intent; otherwise pass by reference or raw pointer.
- `weak_ptr` breaks cycles; `lock()` is the only safe access path.
- `shared_ptr`'s refcount is thread-safe; the object it points to is not.
