Here are **50 tricky C++ interview questions** focusing on **mutex, condition variable, producer-consumer, and concurrency patterns** with answers and code examples.

---

## 1. What is the difference between `std::mutex` and `std::recursive_mutex`?

**Answer:**  
`recursive_mutex` allows same thread to lock multiple times; `mutex` deadlocks.

```cpp
std::recursive_mutex rm;
rm.lock();
rm.lock(); // OK
```

---

## 2. Why shouldn't you use `std::mutex` recursively?

**Answer:**  
Undefined behavior (usually deadlock).

```cpp
std::mutex m;
m.lock();
m.lock(); // UB - likely deadlock
```

---

## 3. What is lock inversion?

**Answer:**  
Thread A holds lock L1, waits for L2; Thread B holds L2, waits for L1 → deadlock.

---

## 4. What is `std::try_lock`?

**Answer:**  
Tries to lock multiple mutexes without blocking.

```cpp
int result = std::try_lock(m1, m2, m3);
if (result == -1) {
    // all locked
} else {
    // mutex at index 'result' failed
}
```

---

## 5. Output of this producer-consumer bug:

```cpp
std::mutex m;
std::queue<int> q;
void producer() {
    for (int i = 0; i < 10; i++) {
        std::lock_guard lock(m);
        q.push(i);
    }
}
void consumer() {
    while (true) {
        std::lock_guard lock(m);
        if (!q.empty()) {
            q.pop();
        }
    }
}
```

**Answer:**  
Consumer never releases mutex while checking empty → producer blocks forever (deadlock or starvation). Fix: release mutex before checking again.

---

## 6. What is a condition variable used for?

**Answer:**  
Block threads until a condition becomes true, avoiding busy-waiting.

---

## 7. Implement correct producer-consumer with `std::condition_variable`:

**Answer:**
```cpp
std::mutex m;
std::condition_variable cv;
std::queue<int> q;
bool done = false;

void producer() {
    for (int i = 0; i < 10; i++) {
        {
            std::lock_guard lock(m);
            q.push(i);
        }
        cv.notify_one();
    }
    {
        std::lock_guard lock(m);
        done = true;
    }
    cv.notify_all();
}

void consumer() {
    while (true) {
        std::unique_lock lock(m);
        cv.wait(lock, []{ return !q.empty() || done; });
        while (!q.empty()) {
            int val = q.front();
            q.pop();
            lock.unlock();
            // process val
            lock.lock();
        }
        if (done) break;
    }
}
```

---

## 8. What happens if you forget to lock mutex before `cv.wait()`?

**Answer:**  
Undefined behavior (wait requires locked mutex).

---

## 9. Why must `cv.wait()` be called with a `unique_lock` not `lock_guard`?

**Answer:**  
`wait` unlocks mutex while sleeping; `lock_guard` can't be unlocked.

---

## 10. What is spurious wakeup? How to handle?

**Answer:**  
`wait` may return without notify. Always wait with predicate:

```cpp
cv.wait(lock, []{ return ready; }); // correct
// cv.wait(lock); // dangerous
```

---

## 11. What is `cv.wait_for()` vs `cv.wait_until()`?

**Answer:**  
`wait_for` — timeout duration; `wait_until` — absolute time point.

```cpp
cv.wait_for(lock, std::chrono::seconds(1), []{ return ready; });
```

---

## 12. What is `notify_one()` vs `notify_all()`?

**Answer:**  
`notify_one` wakes one waiting thread; `notify_all` wakes all.

---

## 13. Producer-consumer with multiple consumers: which notify better?

**Answer:**  
`notify_one` (wake one consumer) unless broadcast needed.

---

## 14. Lost wakeup problem? How to avoid?

**Answer:**  
Lost wakeup = notify before wait. Prevention: condition check must be inside mutex lock.

```cpp
// BAD: possible lost wakeup
if (!ready) cv.wait(lock); // ready may change after check

// GOOD
cv.wait(lock, []{ return ready; });
```

---

## 15. Can you use `std::condition_variable_any`?

**Answer:**  
Works with any lockable type (not just `unique_lock<mutex>`).

```cpp
std::condition_variable_any cv;
std::shared_mutex sm;
std::shared_lock lock(sm);
cv.wait(lock);
```

---

## 16. What is the output?

```cpp
std::mutex m;
std::condition_variable cv;
bool flag = false;

void waiter() {
    std::unique_lock lock(m);
    cv.wait(lock, []{ return flag; });
    std::cout << "done";
}
int main() {
    std::thread t(waiter);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard lock(m);
        flag = true;
    }
    cv.notify_one();
    t.join();
}
```

**Answer:**  
`done` after ~1 second.

---

## 17. What is a monitor pattern?

**Answer:**  
Mutex + condition variable + invariant = monitor (e.g., `std::sync` in Rust).

---

## 18. What is a bounded producer-consumer (with max queue size)?

**Answer:**
```cpp
std::mutex m;
std::condition_variable not_full, not_empty;
std::queue<int> q;
const size_t MAX = 10;

void producer() {
    for (int i = 0; i < 100; i++) {
        std::unique_lock lock(m);
        not_full.wait(lock, []{ return q.size() < MAX; });
        q.push(i);
        not_empty.notify_one();
    }
}
```

---

## 19. Single producer, single consumer: do you need mutex?

**Answer:**  
If using lock-free queue, no. With `std::queue`, yes.

---

## 20. What is a `std::counting_semaphore` alternative to CV?

**Answer:**
```cpp
std::counting_semaphore<10> slots(10); // producer waits
std::counting_semaphore<0> items(0);   // consumer waits

void producer() {
    slots.acquire(); // wait for slot
    // produce
    items.release(); // signal item
}
```

---

## 21. Compare semaphore vs condition variable for producer-consumer.

**Answer:**  
Semaphores simpler for counting; CV more flexible (complex conditions).

---

## 22. What is the bad busy-wait anti-pattern?

**Answer:**
```cpp
while (!ready) {} // burns CPU
```
Fix: use mutex + CV.

---

## 23. Error in this code?

```cpp
void consumer() {
    std::unique_lock lock(m);
    if (q.empty()) {
        cv.wait(lock);
    }
    int val = q.front(); q.pop();
}
```

**Answer:**  
After spurious wakeup, queue still empty → `front()` on empty queue → UB.

**Fix:** `while(q.empty()) cv.wait(lock);`

---

## 24. What is a thread-safe queue? Implement basic.

**Answer:**
```cpp
template<typename T>
class SafeQueue {
    std::queue<T> q;
    mutable std::mutex m;
    std::condition_variable cv;
public:
    void push(T val) {
        std::lock_guard lock(m);
        q.push(std::move(val));
        cv.notify_one();
    }
    T pop() {
        std::unique_lock lock(m);
        cv.wait(lock, [this]{ return !q.empty(); });
        T val = std::move(q.front());
        q.pop();
        return val;
    }
};
```

---

## 25. What is `try_pop` for thread-safe queue?

**Answer:**
```cpp
bool try_pop(T& val) {
    std::lock_guard lock(m);
    if (q.empty()) return false;
    val = std::move(q.front());
    q.pop();
    return true;
}
```

---

## 26. Why pass condition by reference in `cv.wait` predicate?

**Answer:**  
Lambdas capture by reference to avoid copying condition state.

---

## 27. What is `std::notify_all_at_thread_exit` useful for?

**Answer:**  
Signal after thread-local destructors run in detached thread.

```cpp
std::condition_variable cv;
std::mutex m;
bool done = false;

void f() {
    std::unique_lock lock(m);
    std::notify_all_at_thread_exit(cv, std::move(lock));
    done = true;
} // mutex unlocked, cv notified after thread-local cleanup
```

---

## 28. What is the readers-writers problem? Solve with `shared_mutex`.

**Answer:**
```cpp
std::shared_mutex sm;
void reader() {
    std::shared_lock lock(sm); // shared
    // read
}
void writer() {
    std::unique_lock lock(sm); // exclusive
    // write
}
```

---

## 29. Producer-consumer with `shared_mutex` — possible?

**Answer:**  
Not directly — need exclusive access for queue modification. Use regular mutex.

---

## 30. How to implement wait with timeout in producer-consumer?

**Answer:**
```cpp
if (cv.wait_for(lock, std::chrono::milliseconds(100), 
                []{ return !q.empty(); })) {
    // got item
} else {
    // timeout
}
```

---

## 31. What is `std::latch`? Use for producer-consumer startup?

**Answer:**
```cpp
std::latch start(2); // 2 threads ready
void worker() {
    start.arrive_and_wait(); // wait for both
    // work
}
```

---

## 32. What is `std::barrier`? Use for phased producer-consumer?

**Answer:**
```cpp
std::barrier sync(3); // 3 threads (2 prod + 1 cons)
void phase() {
    sync.arrive_and_wait(); // synchronize all
}
```

---

## 33. Diagnose deadlock:

```cpp
std::mutex m;
std::condition_variable cv;
bool flag = false;

void prod() {
    std::lock_guard lock(m);
    flag = true;
    cv.notify_one();
}
void cons() {
    std::unique_lock lock(m);
    cv.wait(lock);
    std::cout << flag;
}
```

**Answer:**  
No deadlock, but `wait` may never wake if `prod` runs first. Flag check missing in predicate.

**Fix:** `cv.wait(lock, []{ return flag; });`

---

## 34. Can condition variable be used without mutex?

**Answer:**  
No — `wait` requires mutex. Use `std::atomic` + spin if avoid mutex.

---

## 35. What is memory ordering effect on condition variable?

**Answer:**  
Mutex and CV enforce acquire/release semantics automatically.

---

## 36. Implement single-producer single-consumer lock-free with `atomic`.

**Answer:**
```cpp
template<typename T>
class LockFreeQueue {
    std::atomic<T*> data;
    std::atomic<bool> has_item{false};
public:
    void produce(T val) {
        while (has_item.load()) {} // spin
        data.store(new T(val));
        has_item.store(true);
    }
    T consume() {
        while (!has_item.load()) {}
        T* p = data.load();
        T val = *p;
        delete p;
        has_item.store(false);
        return val;
    }
};
```
(Not fully correct — use memory orderings.)

---

## 37. What is a monitor object pattern?

**Answer:**  
Class with mutex and condition variable managing its own state.

```cpp
class Queue {
    std::mutex m;
    std::condition_variable cv;
    std::queue<int> q;
public:
    void push(int x) {
        std::lock_guard lock(m);
        q.push(x);
        cv.notify_one();
    }
};
```

---

## 38. Why is `std::condition_variable` not movable?

**Answer:**  
OS-specific native handle prevents moving.

---

## 39. How to wait on multiple condition variables?

**Answer:**  
Can't — use `wait` with predicate checking multiple flags, or `std::experimental::when_any`.

---

## 40. Error: using `std::unique_lock<Mutex>` with `std::condition_variable_any`?

**Answer:**  
Works fine — `condition_variable_any` accepts any lockable.

---

## 41. What is the lost notification problem with multiple condition variables?

**Answer:**  
Notify happens before `wait` on different CV → thread never wakes.

---

## 42. Implement producer-consumer with two `counting_semaphore`s.

**Answer:**
```cpp
std::counting_semaphore<10> empty_slots(10);
std::counting_semaphore<0> full_slots(0);
std::queue<int> q;

void producer() {
    empty_slots.acquire();
    q.push(rand());
    full_slots.release();
}
void consumer() {
    full_slots.acquire();
    int val = q.front(); q.pop();
    empty_slots.release();
}
```
(Need mutex for `std::queue` — semaphores alone insufficient.)

---

## 43. Why semaphores alone insufficient for shared queue?

**Answer:**  
Queue push/pop needs mutual exclusion. Semaphores count, but don't protect data.

---

## 44. What is the "dining philosophers" problem variant for producer-consumer?

**Answer:**  
Multiple producers, consumers sharing fixed-size buffer — needs both counting and mutex.

---

## 45. How to debug deadlock in producer-consumer?

**Answer:**  
- Log lock acquires/releases  
- Use `std::timed_mutex` with timeout  
- Use `gdb`, `helgrind`, `tsan` (ThreadSanitizer)

---

## 46. What is RAII for mutex? Why important?

**Answer:**  
Guarantees unlock on exception. `lock_guard`/`unique_lock`.

---

## 47. Can you `try_lock` on mutex and still use condition variable?

**Answer:**  
No — condition variable needs locked mutex for `wait`.

---

## 48. Output after adding variable:

```cpp
std::mutex m;
bool ready = false;
void wait() {
    std::unique_lock lock(m);
    while (!ready) {
        cv.wait(lock);
    }
}
// Thread 1: wait()
// Thread 2: { lock; ready = true; cv.notify_one(); }
```

**Answer:**  
No output — `while` loop with missing `ready` update check. Works after adding predicate.

---

## 49. What is a bounded buffer problem?

**Answer:**  
Fixed-size buffer with producers waiting when full, consumers when empty.

---

## 50. Final correct producer-consumer with bounded buffer:

**Answer:**
```cpp
class BoundedBuffer {
    std::mutex m;
    std::condition_variable not_full, not_empty;
    std::queue<int> q;
    size_t cap;

public:
    BoundedBuffer(size_t c) : cap(c) {}

    void push(int val) {
        std::unique_lock lock(m);
        not_full.wait(lock, [this]{ return q.size() < cap; });
        q.push(val);
        not_empty.notify_one();
    }

    int pop() {
        std::unique_lock lock(m);
        not_empty.wait(lock, [this]{ return !q.empty(); });
        int val = q.front(); q.pop();
        not_full.notify_one();
        return val;
    }
};
```

---

Let me know if you'd like all 150+ questions from the three sets merged into a single master document, or turned into a quiz format with answers hidden.