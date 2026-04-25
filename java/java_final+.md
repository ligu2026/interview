# Difference Between `final`, `finally`, and `finalize` in Java
This is **one of the most frequently asked Java interview questions** — these three keywords sound similar but have **completely different purposes**.

Let’s break them down clearly with simple definitions, use cases, and code examples.

---

## 1. `final` → **Keyword (Modifier)**
`final` is used to restrict **classes, methods, and variables**.

### Use Cases:
- **Final variable**: Value cannot be changed (constant)
- **Final method**: Cannot be **overridden** by subclasses
- **Final class**: Cannot be **inherited** (extended)

### Example:
```java
// Final variable
final int MAX = 100;
// MAX = 200; → ERROR: Cannot assign value to final variable

// Final method
class Parent {
    final void show() {}
}
class Child extends Parent {
    // void show() {} → ERROR: Cannot override final method
}

// Final class
final class Demo {}
// class Test extends Demo {} → ERROR: Cannot inherit final class
```

---

## 2. `finally` → **Block (Exception Handling)**
`finally` is a block used **with try/catch** for **cleanup code** that **MUST run** — whether an exception occurs or not.

### Use Case:
Close files, database connections, sockets, release resources.

### Rule:
- Executes **after try/catch**
- Only skipped if JVM crashes (`System.exit()`)

### Example:
```java
try {
    // risky code
} catch (Exception e) {
    // handle error
} finally {
    // This code ALWAYS runs
    closeFile();
    closeDBConnection();
}
```

---

## 3. `finalize()` → **Method (Garbage Collection)**
`finalize()` is a **method of Object class** called by the **Garbage Collector (GC)** **before deleting an unused object**.

### Important Facts:
- Used for **resource cleanup** (before object destruction)
- **Unpredictable** (we don’t know when GC runs)
- **Deprecated in Java 9+** (bad practice, unsafe)

### Example:
```java
class Test {
    // Called by GC before object is deleted
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Object destroyed");
    }
}
```

---

# Quick Comparison Table
| Feature       | `final`                      | `finally`                    | `finalize()`                  |
|---------------|------------------------------|------------------------------|-------------------------------|
| Type          | Keyword (modifier)           | Block                        | Method                        |
| Purpose       | Restrict classes/methods/vars| Cleanup code (always runs)   | Cleanup before GC destroys obj|
| Used With     | Variables, methods, classes  | try/catch                    | Object class                  |
| Execution     | Compile-time control         | Runs after try/catch         | Called by GC (unpredictable)  |
| Status        | Commonly used                | Commonly used                | Deprecated (Java 9+)          |

---

### Summary
- **`final`**: Makes things **fixed/unchangeable**
- **`finally`**: Ensures code **always runs** (cleanup)
- **`finalize()`**: GC method to clean up objects (**avoid using it**)

This is the complete, interview-ready answer you need!

**No, you cannot override a final method in Java.**

### Explanation
- The `final` modifier on a method **explicitly prevents overriding** in any subclass.
- If you attempt to declare a method with the **same signature** in a subclass, the compiler will throw an error:
  > `Cannot override the final method from SuperClass`

### Example (Compile Error)
```java
class Parent {
    public final void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    // ❌ COMPILE ERROR: overridden method is final
    public void show() {
        System.out.println("Child");
    }
}
```

### Key Note
- A final method is still **inherited** (you can call it from the subclass), but you cannot change its implementation.
- This is often used for security or to preserve critical logic.

The **`finalize()`** method in Java is a special method defined in the `Object` class, intended to perform **resource cleanup** right before an object is **garbage collected (GC)**.

### Core Purpose
- To release **non-JVM resources** that the garbage collector does not manage automatically, such as:
  - Open file handles
  - Network sockets
  - Database connections
  - Native memory (allocated via JNI)
- It acts as a **safety net** for resources that were not properly closed by the developer.

### Important Facts & Pitfalls
1. **Called by the Garbage Collector**
   You never call `finalize()` directly; the JVM invokes it *once*, just before reclaiming the object’s memory.

2. **Unpredictable & Unreliable**
   - You have **no control over when GC runs**, so `finalize()` may never execute at all.
   - It can cause performance issues and delays in resource release.

3. **Exception Behavior**
   Any exception thrown inside `finalize()` is **ignored** and does not stop the GC process.

4. **Deprecated Since Java 9**
   `finalize()` is **deprecated and marked for removal** because it is unsafe, slow, and leads to resource leaks.

### Modern Replacement
Instead of `finalize()`, use:
- **`try-with-resources`** (for `AutoCloseable` objects like streams, connections)
- **`Cleaner` / `PhantomReference`** (Java 9+) for safe post-mortem cleanup

### Simple Example
```java
@Override
protected void finalize() throws Throwable {
    try {
        // close socket, file, etc.
    } finally {
        super.finalize();
    }
}
```

### Short Answer (Interview Version)
The `finalize()` method is used to perform cleanup operations on unmanaged resources before an object is garbage collected, but it is unpredictable, deprecated, and should not be used in modern Java.


### Short Answer:
**Yes, you CAN override the `finalize()` method in Java** — but you **SHOULD NOT** (it’s deprecated, unsafe, and bad practice).

---

### Full Explanation
1. **`finalize()` is a protected method in the `Object` class**
   Since all Java classes inherit from `Object`, you **can override it** in your own class.

2. **Override Rules**
   - You must keep the method signature:
     ```java
     @Override
     protected void finalize() throws Throwable {
         // your code
     }
     ```
   - You can make the access modifier **less restrictive** (e.g., `public`), but not more restrictive.

3. **Critical Warning**
   - `finalize()` was **deprecated in Java 9** and is planned for removal in future Java versions.
   - It’s **unpredictable**: the JVM garbage collector decides when (or if) to call it.
   - It causes **performance issues, memory leaks, and bugs**.

---

### Example (Valid but NOT Recommended)
```java
public class MyClass {
    // Overriding finalize() - works but deprecated/unsafe
    @Override
    protected void finalize() throws Throwable {
        try {
            // Cleanup code (DO NOT USE THIS IN REAL CODE)
            System.out.println("finalize() called");
        } finally {
            super.finalize(); // Always call parent finalize
        }
    }
}
```

---

### What to Use Instead?
Use **`try-with-resources`** (for auto-closing files, streams, connections) — this is the modern, safe replacement.

### Summary
- ✅ **Can override?** Yes
- ❌ **Should override?** No (deprecated, unsafe)
- 🚀 **Modern alternative:** `try-with-resources`

This is the exact answer you need for interviews!



### How can I perform custom cleanup tasks in Java if finalize() is not reliable?
### Short Answer
**Technically you can, but you absolutely should NOT rely on it in real code.**

### Full Explanation
1. **You *can* write cleanup logic inside `finalize()`**
   You can override `finalize()` to close files, sockets, release native resources, etc. But this is **unreliable**.

2. **Why it’s a terrible idea**
   - **Unpredictable execution**: You have no control over *when* the GC runs — `finalize()` may be delayed for minutes/hours, or **never called at all**.
   - **Performance cost**: Finalization adds significant GC overhead.
   - **Exceptions are swallowed**: Any exception thrown in `finalize()` is ignored, so cleanup can fail silently.
   - **Objects can be “resurrected”**: If you make the object reachable again inside `finalize()`, it escapes GC but won’t be finalized a second time.
   - **Deprecated & being removed**: `finalize()` is deprecated since Java 9 and marked for future deletion.

3. **What to use instead (proper cleanup)**
   - For resources like streams, connections, sockets: use **`try-with-resources`** (they implement `AutoCloseable`).
   - For advanced post-mortem cleanup: use `java.lang.ref.Cleaner` or `PhantomReference` (Java 9+).

### Interview Summary
> Yes, you *can* put custom cleanup in `finalize()`, but it is unsafe, unpredictable, deprecated, and not guaranteed to run. It should never be used for critical resource cleanup in production code.


# Best Ways to Perform Custom Cleanup in Java (Without `finalize()`)
Since `finalize()` is **unreliable, deprecated, and unsafe**, Java provides **modern, guaranteed, and safe alternatives** for custom cleanup tasks (closing files, DB connections, releasing native resources, etc.).

Below are the **4 industry-standard approaches** — ordered by priority (use #1 for 99% of cases).

---

## 1. **try-with-resources (Best & Most Used)**
This is the **default choice** for all resources that implement `AutoCloseable` (Java 7+).  
It **guarantees cleanup** — even if an exception occurs.

### How it works:
- Declare resources inside `try(...)`
- Java automatically calls `close()`
- No need for `finally` blocks

### Example:
```java
// Auto-closeable resources (files, streams, DB connections)
try (FileInputStream fis = new FileInputStream("test.txt");
     Socket socket = new Socket("localhost", 8080)) {
    
    // Use resources
} catch (IOException e) {
    // Handle error
}
// Resources are CLOSED AUTOMATICALLY here
```

### For custom cleanup:
Make your class implement `AutoCloseable`:
```java
class CustomResource implements AutoCloseable {
    @Override
    public void close() {
        // YOUR CUSTOM CLEANUP (guaranteed to run)
        System.out.println("Cleanup done!");
    }
}

// Usage
try (CustomResource res = new CustomResource()) {
    // Use resource
}
```

---

## 2. **Explicit close() / cleanup() Method**
For non-auto-closeable resources, define a **public cleanup method** and call it manually (ideally in `finally`).

### Example:
```java
class MyService {
    public void disconnect() {
        // Custom cleanup logic
    }
}

// Usage
MyService service = new MyService();
try {
    // Use service
} finally {
    service.disconnect(); // GUARANTEED cleanup
}
```

---

## 3. **java.lang.ref.Cleaner (Java 9+)**
For **advanced, post-GC cleanup** (native memory, off-heap resources) — **safe replacement for finalize()**.

- Runs *after* the object is garbage collected
- No risk of object resurrection
- Predictable & safe

### Example:
```java
public class NativeResource implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    private final Cleaner.Cleanable cleanable;

    public NativeResource() {
        // Register cleanup task
        this.cleanable = cleaner.register(this, new CleanupTask());
    }

    // Cleanup logic (runs when object is GC'd)
    private static class CleanupTask implements Runnable {
        @Override
        public void run() {
            System.out.println("Safe GC cleanup");
        }
    }

    @Override
    public void close() {
        cleanable.clean(); // Explicit cleanup
    }
}
```

---

## 4. **PhantomReference + ReferenceQueue**
Low-level API for full control over post-mortem cleanup (used by frameworks).  
More complex than `Cleaner` — use only if needed.

---

# Quick Comparison (Interview Answer)
| Method               | Reliability  | Use Case                                  |
|----------------------|--------------|-------------------------------------------|
| try-with-resources   | ✅ 100%      | Most resources (files, streams, DB)       |
| Explicit close()     | ✅ 100%      | Custom services                           |
| Cleaner              | ✅ Safe      | Native/off-heap resources (post-GC)       |
| finalize()           | ❌ Unreliable | DEPRECATED — never use                    |

---

### Final Summary
1. **First choice**: `try-with-resources` (implements `AutoCloseable`)
2. **Second choice**: Explicit `close()` + `finally`
3. **Advanced**: `java.lang.ref.Cleaner` (Java 9+)

**Never use `finalize()` for real-world cleanup.**