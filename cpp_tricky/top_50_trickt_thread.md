Here are **50 tricky C++ interview questions** focusing on **threads, concurrency, synchronization, and memory ordering** with answers and code examples.

---

## 1. What is the difference between `std::thread` and `std::jthread` (C++20)?

**Answer:**  
`jthread` automatically joins on destruction and supports cooperative interruption.

```cpp
std::jthread t([](std::stop_token st) {
    while (!st.stop_requested()) { /* work */ }
}); // automatically joins
```

---

## 2. What happens if you don't join or detach a `std::thread`?

**Answer:**  
`std::terminate()` is called in the destructor.

---

## 3. Can you copy a `std::thread`?

**Answer:**  
No — move-only.

```cpp
std::thread t1(f);
std::thread t2 = std::move(t1); // OK
// std::thread t3 = t1; // Error
```

---

## 4. What is a data race?

**Answer:**  
Two or more threads accessing same memory concurrently, at least one write, no synchronization → undefined behavior.

---

## 5. What is `std::mutex`? Why use `std::lock_guard`?

**Answer:**  
`mutex` provides exclusive access; `lock_guard` is RAII wrapper.

```cpp
std::mutex m;
void safe_increment() {
    std::lock_guard<std::mutex> lock(m);
    // critical section
}
```

---

## 6. What's wrong here?

```cpp
std::mutex m1, m2;
void f1() { std::lock_guard l1(m1); std::lock_guard l2(m2); }
void f2() { std::lock_guard l2(m2); std::lock_guard l1(m1); }
```

**Answer:**  
Potential deadlock (different lock order).

**Fix:** Use `std::lock(m1, m2)` or same order.

---

## 7. What is `std::recursive_mutex`?

**Answer:**  
Same thread can lock multiple times (counts locks).

```cpp
std::recursive_mutex m;
void f() { m.lock(); m.lock(); } // OK
```

---

## 8. When should you NOT use `recursive_mutex`?

**Answer:**  
Design smell — indicates bad locking strategy or broken invariants.

---

## 9. What is `std::unique_lock` vs `std::lock_guard`?

**Answer:**  
`unique_lock` more flexible (defer, try_lock, unlock, move).

```cpp
std::unique_lock<std::mutex> lock(m, std::defer_lock);
lock.lock(); // later
```

---

## 10. What is deadlock? How to avoid?

**Answer:**  
Two+ threads wait forever for each other's locks. Avoid by:  
- Lock in consistent order  
- Use `std::lock`  
- Use timed mutexes

---

## 11. What is `std::scoped_lock` (C++17)?

**Answer:**  
Variadic RAII lock — locks multiple mutexes without deadlock.

```cpp
std::scoped_lock lock(m1, m2, m3); // locks all
```

---

## 12. What is `std::atomic`? When to use?

**Answer:**  
Lock-free operations on fundamental types.

```cpp
std::atomic<int> counter{0};
counter++; // atomic
```

---

## 13. Is `volatile` useful for threading?

**Answer:**  
No — `volatile` doesn't provide atomicity or synchronization; use `std::atomic`.

---

## 14. What is a condition variable? When to use?

**Answer:**  
`std::condition_variable` blocks thread until condition met.

```cpp
std::mutex m;
std::condition_variable cv;
bool ready = false;

cv.wait(lock, []{ return ready; }); // waits
cv.notify_one(); // signals
```

---

## 15. What is spurious wakeup?

**Answer:**  
`cv.wait()` can return even without notify. Always check predicate in loop.

---

## 16. Output:

```cpp
std::atomic<int> x{0};
void f() { x++; }
int main() {
    std::thread t1(f), t2(f);
    t1.join(); t2.join();
    std::cout << x;
}
```

**Answer:**  
`2` (always, because atomic)

---

## 17. What is `std::atomic::load()` and `store()`?

**Answer:**  
Explicit atomic read/write with memory ordering.

```cpp
x.load(std::memory_order_acquire);
x.store(10, std::memory_order_release);
```

---

## 18. What are memory ordering semantics?

**Answer:**  
- `relaxed` — no ordering constraints  
- `acquire` — subsequent reads not reordered before  
- `release` — previous writes not reordered after  
- `seq_cst` — sequential consistency (default)

---

## 19. What is a happens-before relationship?

**Answer:**  
Guarantees that write to atomic with `release` is visible to `acquire` read.

---

## 20. What is false sharing?

**Answer:**  
Threads on different cores modify variables in same cache line → performance degradation.

**Fix:** Padding or alignas.

```cpp
struct alignas(64) Padded { int x; };
```

---

## 21. What is `std::call_once` and `std::once_flag`?

**Answer:**  
Execute function exactly once, even in multiple threads.

```cpp
std::once_flag flag;
std::call_once(flag, []{ init(); });
```

---

## 22. What is `thread_local` storage?

**Answer:**  
Each thread has its own instance.

```cpp
thread_local int x = 0; // each thread gets copy
```

---

## 23. Can you pass reference to thread function?

**Answer:**  
Yes, but use `std::ref`.

```cpp
void f(int& x) { x++; }
int a = 0;
std::thread t(f, std::ref(a));
```

---

## 24. What happens if thread function throws?

**Answer:**  
`std::terminate` called unless caught inside thread.

---

## 25. What is `std::async`? Difference from `std::thread`?

**Answer:**  
`async` returns `std::future`; may use thread pool.

```cpp
auto fut = std::async(f); // may run in separate thread or deferred
int result = fut.get(); // blocks
```

---

## 26. What is `std::future` and `std::promise`?

**Answer:**  
`promise` sets value, `future` gets it (one-way communication).

```cpp
std::promise<int> p;
std::future<int> f = p.get_future();
p.set_value(42);
int v = f.get();
```

---

## 27. What is `std::shared_future`?

**Answer:**  
Multiple threads can wait on same result (copyable).

```cpp
std::shared_future<int> sf = f.share();
```

---

## 28. What is a thread pool?

**Answer:**  
Fixed number of threads reusing work queue. Not in standard library until C++26 executors proposal.

---

## 29. What is `std::hardware_concurrency()`?

**Answer:**  
Returns hint of number of concurrent threads supported.

```cpp
unsigned n = std::thread::hardware_concurrency();
```

---

## 30. What is `std::this_thread::sleep_for`?

**Answer:**  
Sleeps current thread.

```cpp
std::this_thread::sleep_for(std::chrono::milliseconds(10));
```

---

## 31. Can mutex be locked in one thread, unlocked in another?

**Answer:**  
Yes for `std::mutex` — undefined behavior (must unlock in same thread).

---

## 32. What is `std::timed_mutex`?

**Answer:**  
Try to lock with timeout.

```cpp
std::timed_mutex m;
if (m.try_lock_for(std::chrono::milliseconds(100))) { ... }
```

---

## 33. What is a semaphore? How in C++?

**Answer:**  
Counting synchronization primitive. C++20: `std::counting_semaphore`.

```cpp
std::counting_semaphore sem(3); // max 3
sem.acquire(); // decrement
sem.release(); // increment
```

---

## 34. What is `std::latch` (C++20)?

**Answer:**  
Single-use countdown latch.

```cpp
std::latch done(5); // wait for 5
done.count_down(); // decrement
done.wait(); // wait until 0
```

---

## 35. What is `std::barrier` (C++20)?

**Answer:**  
Reusable synchronization point for thread phases.

```cpp
std::barrier sync(3, []{ cout << "phase done\n"; });
sync.arrive_and_wait();
```

---

## 36. What is ABA problem in lock-free programming?

**Answer:**  
Value changes A→B→A, CAS succeeds incorrectly. Fix: tagged pointers.

---

## 37. Can you use `std::atomic` on custom types?

**Answer:**  
Yes, if `std::is_trivially_copyable<T>`.

```cpp
struct Point { int x,y; };
std::atomic<Point> p;
```

---

## 38. What is `std::atomic_flag`?

**Answer:**  
Lowest-level atomic boolean, lock-free guaranteed.

```cpp
std::atomic_flag flag = ATOMIC_FLAG_INIT;
if (!flag.test_and_set()) { /* was false */ }
flag.clear();
```

---

## 39. Output:

```cpp
int x = 0;
void f() { x++; }
int main() {
    std::thread t1(f), t2(f);
    t1.join(); t2.join();
    std::cout << x; // ????
}
```

**Answer:**  
Data race → undefined behavior. Output could be 1 or 2 or crash.

---

## 40. How to fix the above?

**Answer:**  
`std::atomic<int> x{0};` or `std::mutex`.

---

## 41. What is `std::notify_all_at_thread_exit`?

**Answer:**  
Notifies condition variable after thread-local destructors run.

---

## 42. Can you detach a thread that owns mutex?

**Answer:**  
Yes, but dangerous — mutex may be destroyed while locked.

---

## 43. What is `std::stop_token` and `std::stop_source` (C++20)?

**Answer:**  
Cooperative thread interruption for `jthread`.

```cpp
std::jthread t([](std::stop_token st) {
    while (!st.stop_requested()) { }
});
t.request_stop();
```

---

## 44. What is a spinlock? Implement with `atomic_flag`.

**Answer:**  
Busy-wait lock.

```cpp
class Spinlock {
    std::atomic_flag flag = ATOMIC_FLAG_INIT;
public:
    void lock() { while (flag.test_and_set()) ; }
    void unlock() { flag.clear(); }
};
```

---

## 45. Why is `std::async` with `std::launch::async` dangerous?

**Answer:**  
Might spawn many threads if called in loop → resource exhaustion.

---

## 46. What is double-checked locking? Is it safe?

**Answer:**  
In C++11+, safe with `std::atomic`.

```cpp
std::atomic<Singleton*> instance;
if (!instance.load(std::memory_order_acquire)) {
    lock; if (!instance) instance = new Singleton();
}
```

---

## 47. What is `std::exception_ptr`?

**Answer:**  
Capture exception from thread.

```cpp
std::exception_ptr eptr;
try { throw 1; } catch(...) { eptr = std::current_exception(); }
```

---

## 48. Can you use `std::future` without `std::async`?

**Answer:**  
Yes — `std::packaged_task`.

```cpp
std::packaged_task<int()> task([]{ return 42; });
std::future<int> f = task.get_future();
std::thread t(std::move(task));
```

---

## 49. What is a deadlock avoidance with `std::lock`?

**Answer:**  
`std::lock(m1, m2)` locks without deadlock (try-and-backoff algorithm).

---

## 50. Is this thread-safe?

```cpp
std::map<int, int> m;
void add(int k, int v) { m[k] = v; }
```

**Answer:**  
No — concurrent modification of non-atomic container is data race. Use mutex or `std::concurrent_map` (Intel TBB).

---

Let me know if you want the complete set (all 150 questions across 3 topics) compiled into one document, or a quiz format for self-testing.