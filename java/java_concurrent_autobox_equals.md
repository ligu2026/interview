# Java Tricky Interview Topics
## ConcurrentModificationException + Autoboxing, Object Creation & == vs equals()

---

## 🔷 PART 1 — ConcurrentModificationException

### The Problem: Deleting a Key Inside a Map Loop

When you iterate over a `Map` and delete a key directly during the loop, Java throws a `ConcurrentModificationException` because the map's internal `modCount` changes while the iterator is active.

```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
map.put("c", 3);

// ❌ WRONG — throws ConcurrentModificationException
for (String key : map.keySet()) {
    if (key.equals("b")) {
        map.remove(key); // modifies map while iterating!
    }
}
```

---

### The Fixes

**✅ Option 1 — Use `Iterator.remove()` (classic, safe)**
```java
Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, Integer> entry = it.next();
    if (entry.getKey().equals("b")) {
        it.remove(); // safe — removes via the iterator itself
    }
}
```

---

**✅ Option 2 — Use `removeIf()` on `entrySet()` (Java 8+, cleanest)**
```java
map.entrySet().removeIf(entry -> entry.getKey().equals("b"));
```

---

**✅ Option 3 — Collect keys first, then remove**
```java
List<String> toRemove = new ArrayList<>();
for (String key : map.keySet()) {
    if (key.equals("b")) toRemove.add(key);
}
toRemove.forEach(map::remove);
```

---

**✅ Option 4 — Use `ConcurrentHashMap` (for multithreaded code)**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
// iterators are weakly consistent — no exception on remove during iteration
for (String key : map.keySet()) {
    if (key.equals("b")) map.remove(key); // safe
}
```

---

### Summary Table

| Approach | Safe? | Best for |
|----------|-------|----------|
| `map.remove()` in for-each | ❌ | — |
| `Iterator.remove()` | ✅ | Classic single-thread |
| `entrySet().removeIf()` | ✅ | Clean Java 8+ code |
| Collect keys, then remove | ✅ | Readable, simple |
| `ConcurrentHashMap` | ✅ | Multithreaded code |

---

## 🔷 PART 2 — Autoboxing & Wrapper Classes

**Q1. What is autoboxing and unboxing? What are the risks?**
- **Autoboxing** — automatic conversion of primitive → wrapper (`int` → `Integer`)
- **Unboxing** — automatic conversion of wrapper → primitive (`Integer` → `int`)

**Risk 1 — NullPointerException on unboxing:**
```java
Integer x = null;
int y = x; // ❌ NullPointerException — unboxing null
```

**Risk 2 — Hidden performance cost in loops:**
```java
Long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i; // ❌ unboxes sum, adds i, re-boxes result — 1M objects created!
}
// ✅ Fix: use primitive long sum = 0L;
```

---

**Q2. What does this print and why?**
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // ?

Integer c = 128;
Integer d = 128;
System.out.println(c == d); // ?
```
**Answer:**
```
true
false
```
**Why?** Java caches `Integer` objects for values **-128 to 127** (the Integer Cache). So `a` and `b` point to the **same cached object**. `128` is outside the cache range — two separate objects are created on the heap, so `==` compares references and returns `false`.

---

**Q3. What does this print?**
```java
Integer a = 100;
int b = 100;
System.out.println(a == b); // ?
```
**Answer:** `true`

When one operand is a primitive, Java **unboxes** the `Integer` to `int` and compares values — not references. So this becomes `100 == 100`.

---

**Q4. What does this print?**
```java
Integer x = null;
Integer y = null;
System.out.println(x == y);      // ?
System.out.println(x.equals(y)); // ?
```
**Answer:**
```
true
NullPointerException
```
`x == y` compares two null references — both are null so `true`. `x.equals(y)` calls a method on `null` — throws `NullPointerException`.

---

**Q5. What is the Integer Cache and can you change its range?**
Java caches `Integer` instances from **-128 to 127** by default via `IntegerCache` inside `Integer` class. You can extend the upper bound (but not lower) via JVM flag:
```bash
-XX:AutoBoxCacheMax=1000
```
This affects `Integer` only — `Long`, `Short`, `Byte` also cache -128 to 127 but are not configurable.

---

**Q6. Which wrapper types have a cache?**

| Wrapper | Cache Range | Configurable? |
|---------|------------|---------------|
| `Byte` | -128 to 127 | No |
| `Short` | -128 to 127 | No |
| `Integer` | -128 to 127 | Upper bound only |
| `Long` | -128 to 127 | No |
| `Character` | 0 to 127 | No |
| `Boolean` | `TRUE` / `FALSE` | No |
| `Float` / `Double` | ❌ No cache | — |

---

**Q7. What does this print?**
```java
Double a = 1.0;
Double b = 1.0;
System.out.println(a == b); // ?
```
**Answer:** `false`

`Double` has **no cache** — every autoboxed `Double` creates a new heap object. `==` compares references → `false`. Always use `.equals()` for wrappers.

---

## 🔷 PART 3 — Object Creation

**Q8. How many objects are created here?**
```java
String s = new String("hello");
```
**Answer: Up to 2 objects**
1. `"hello"` literal — added to the **String Pool** if not already there
2. `new String("hello")` — a **new heap object** always created, separate from the pool

---

**Q9. How many objects are created here?**
```java
String a = "hello";
String b = "hello";
```
**Answer: 1 object**

Both `a` and `b` point to the **same interned string** in the pool. No new object is created for `b`.

---

**Q10. What does this print?**
```java
String a = "hello";
String b = "hello";
String c = new String("hello");

System.out.println(a == b);          // ?
System.out.println(a == c);          // ?
System.out.println(a.equals(c));     // ?
System.out.println(a == c.intern()); // ?
```
**Answer:**
```
true   — same pool reference
false  — c is a new heap object
true   — same content
true   — c.intern() returns the pool reference
```

---

**Q11. What does this print?**
```java
String s1 = "Hello" + "World";
String s2 = "HelloWorld";
System.out.println(s1 == s2); // ?
```
**Answer:** `true`

Both string literals are **compile-time constants** — the compiler folds `"Hello" + "World"` into `"HelloWorld"` at compile time. Both point to the same pool entry.

---

**Q12. What does this print?**
```java
String a = "Hello";
String b = "World";
String s1 = a + b;
String s2 = "HelloWorld";
System.out.println(s1 == s2); // ?
```
**Answer:** `false`

`a` and `b` are **variables** — concatenation happens at **runtime** using `StringBuilder`, creating a new heap object. `s2` is in the pool. Different references.

---

**Q13. How many objects are created in this loop?**
```java
String result = "";
for (int i = 0; i < 5; i++) {
    result += i;
}
```
**Answer:** Each `+=` creates a new `StringBuilder` + a new `String` — roughly **10+ objects**. The compiler translates each `+=` to `new StringBuilder(result).append(i).toString()`.

✅ Fix:
```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 5; i++) sb.append(i);
String result = sb.toString();
```

---

## 🔷 PART 4 — `==` vs `equals()` Deep Dive

**Q14. What does `==` compare for objects vs primitives?**
- **Primitives** — compares **values** directly
- **Objects/References** — compares **memory addresses** (whether both point to the same object)

```java
int a = 5, b = 5;
a == b; // true — value comparison

Object x = new Object();
Object y = new Object();
x == y; // false — different heap addresses
x == x; // true — same reference
```

---

**Q15. What is the contract of `equals()`?**
The `equals()` method must be:
- **Reflexive**: `x.equals(x)` → `true`
- **Symmetric**: `x.equals(y)` ↔ `y.equals(x)`
- **Transitive**: if `x.equals(y)` and `y.equals(z)` → `x.equals(z)`
- **Consistent**: same result on repeated calls if no state changes
- **Null-safe**: `x.equals(null)` → always `false`

---

**Q16. What happens if you override `equals()` but not `hashCode()`?**
Objects that are logically equal will have **different hash codes** — breaking the contract. This causes them to land in different `HashMap` buckets, so `map.get(key)` will **fail to find** a logically equal key.

```java
class Point {
    int x, y;
    // equals() overridden, hashCode() NOT overridden
}

Map<Point, String> map = new HashMap<>();
map.put(new Point(1, 2), "A");
map.get(new Point(1, 2)); // ❌ returns null — different hashCode!
```

---

**Q17. What does this print?**
```java
Integer a = 1000;
Integer b = 1000;
System.out.println(a.equals(b)); // ?
System.out.println(a == b);      // ?
```
**Answer:**
```
true   — equals() compares int value
false  — 1000 is outside cache; two different heap objects
```

---

**Q18. What does this print?**
```java
Object a = new Object();
Object b = a;
System.out.println(a == b);      // ?
System.out.println(a.equals(b)); // ?
```
**Answer:**
```
true
true
```
`b = a` copies the **reference** — both point to the same object. `Object.equals()` defaults to `==` unless overridden.

---

**Q19. What is the problem with this `equals()` implementation?**
```java
public boolean equals(Point other) { // ❌
    return this.x == other.x && this.y == other.y;
}
```
**Problem:** The parameter is `Point`, not `Object`. This **overloads** `equals()` instead of **overriding** it. `Collections`, `HashMap`, and `List.contains()` call `equals(Object)` — so this custom method is **never used** by the JDK.

✅ Fix:
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Point p)) return false;
    return this.x == p.x && this.y == p.y;
}
```

---

**Q20. What does `Objects.equals(a, b)` do differently?**
It is **null-safe** — avoids `NullPointerException` when either operand is `null`.

```java
Objects.equals(null, null);   // true
Objects.equals("hi", null);   // false
Objects.equals(null, "hi");   // false
Objects.equals("hi", "hi");   // true — calls a.equals(b)
```
Always prefer `Objects.equals()` when either value could be `null`.

---

## 🔷 PART 5 — Tricky Bonus Questions

**Q21. What does this print?**
```java
Integer i = 0;
while (i < 1000) {
    i = i + 1; // what's really happening?
}
```
Each iteration: unboxes `i` → adds 1 → **autoboxes result** back to `Integer`. So **1000 `Integer` objects** are created (though most will be cached for 0–127). A significant hidden cost!

---

**Q22. What does this print?**
```java
Integer a = new Integer(5);   // deprecated
Integer b = Integer.valueOf(5);
Integer c = 5;

System.out.println(a == b); // ?
System.out.println(b == c); // ?
System.out.println(a == c); // ?
```
**Answer:**
```
false  — new Integer() always creates a new heap object
true   — valueOf() uses cache; autoboxing also uses valueOf()
false  — new Integer() bypasses cache
```

---

**Q23. What does this print?**
```java
String s1 = new String("java");
String s2 = s1.intern();
String s3 = "java";

System.out.println(s1 == s2); // ?
System.out.println(s2 == s3); // ?
```
**Answer:**
```
false  — s1 is heap object; s2 is pool reference
true   — both point to pool
```

---

## 🔷 Master Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│              == vs equals() Quick Reference                 │
├─────────────────────────┬───────────────────────────────────┤
│ Primitives              │ == always compares VALUES          │
│ Wrapper objects         │ == compares REFERENCES             │
│                         │ .equals() compares VALUES          │
│ String literals         │ == works (pool), use .equals()     │
│ new String()            │ == always false vs other strings   │
│ Integer cache           │ == works only for -128 to 127      │
│ null check              │ use Objects.equals() for safety    │
├─────────────────────────┴───────────────────────────────────┤
│           ConcurrentModificationException Quick Fix         │
├─────────────────────────────────────────────────────────────┤
│ ❌ map.remove() in for-each     → ConcurrentModification    │
│ ✅ Iterator.remove()            → safe, single-thread        │
│ ✅ entrySet().removeIf()        → cleanest Java 8+           │
│ ✅ collect keys, then remove    → simple and readable        │
│ ✅ ConcurrentHashMap            → multi-threaded             │
├─────────────────────────────────────────────────────────────┤
│                  Autoboxing Gotchas                         │
├─────────────────────────────────────────────────────────────┤
│ Integer x = null; int y = x;   → NullPointerException      │
│ Long sum = 0L; sum += i;       → 1M objects in loop         │
│ Double a = 1.0; a == 1.0;      → false (no Double cache)    │
│ new Integer(5) == 5            → false (bypasses cache)     │
│ Integer.valueOf(5) == 5        → true  (uses cache)         │
└─────────────────────────────────────────────────────────────┘
```

---

*End of Combined Guide — ConcurrentModificationException + Autoboxing + Object Creation + == vs equals()*
