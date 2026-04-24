# Top 50 Java Collections Interview Questions & Answers

---

## 1. What is the Java Collections Framework?

The Java Collections Framework (JCF) is a unified architecture for storing and manipulating groups of objects. It includes:
- **Interfaces**: `Collection`, `List`, `Set`, `Queue`, `Map`
- **Implementations**: `ArrayList`, `HashMap`, `HashSet`, etc.
- **Algorithms**: `Collections.sort()`, `Collections.shuffle()`, etc.

```
Collection
├── List      → ArrayList, LinkedList, Vector, Stack
├── Set       → HashSet, LinkedHashSet, TreeSet
└── Queue     → PriorityQueue, ArrayDeque

Map           → HashMap, LinkedHashMap, TreeMap, Hashtable
```

---

## 2. What is the difference between `Collection` and `Collections`?

| | `Collection` | `Collections` |
|---|---|---|
| Type | Interface | Utility class |
| Purpose | Root interface for lists, sets, queues | Static helper methods |
| Example | `List<String> list = new ArrayList<>()` | `Collections.sort(list)` |

```java
List<Integer> nums = new ArrayList<>(List.of(3, 1, 2));
Collections.sort(nums);           // utility method
System.out.println(nums);         // [1, 2, 3]
```

---

## 3. What is the difference between `ArrayList` and `LinkedList`?

| Feature | `ArrayList` | `LinkedList` |
|---|---|---|
| Data structure | Dynamic array | Doubly linked list |
| Random access | O(1) | O(n) |
| Insert/Delete (middle) | O(n) | O(1) after traversal |
| Memory | Less (no node pointers) | More (prev + next pointers) |
| Best for | Read-heavy | Insert/Delete-heavy |

```java
List<String> al = new ArrayList<>();
al.add("A"); al.add("B"); al.add("C");
System.out.println(al.get(1));    // B — O(1)

List<String> ll = new LinkedList<>();
ll.add("X"); ll.add("Y");
((LinkedList<String>) ll).addFirst("Z"); // efficient prepend
System.out.println(ll);           // [Z, X, Y]
```

---

## 4. What is the difference between `ArrayList` and `Vector`?

| Feature | `ArrayList` | `Vector` |
|---|---|---|
| Thread-safe | No | Yes (synchronized) |
| Performance | Faster | Slower |
| Growth | 50% | 100% (doubles) |
| Legacy | No | Yes (Java 1.0) |

> Prefer `ArrayList` + explicit synchronization or `CopyOnWriteArrayList` over `Vector`.

---

## 5. What is the difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?

| Feature | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| Order | No order | Insertion order | Sorted order |
| Null | 1 allowed | 1 allowed | Not allowed |
| Performance | O(1) | O(1) | O(log n) |
| Underlying | `HashMap` | `LinkedHashMap` | Red-Black tree |

```java
Set<String> hs = new HashSet<>(List.of("Banana", "Apple", "Cherry"));
Set<String> ls = new LinkedHashSet<>(List.of("Banana", "Apple", "Cherry"));
Set<String> ts = new TreeSet<>(List.of("Banana", "Apple", "Cherry"));

System.out.println(hs); // [Cherry, Apple, Banana] — unordered
System.out.println(ls); // [Banana, Apple, Cherry] — insertion order
System.out.println(ts); // [Apple, Banana, Cherry] — sorted
```

---

## 6. What is the difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?

| Feature | `HashMap` | `LinkedHashMap` | `TreeMap` |
|---|---|---|---|
| Order | No order | Insertion order | Sorted by key |
| Null keys | 1 allowed | 1 allowed | Not allowed |
| Performance | O(1) avg | O(1) avg | O(log n) |

```java
Map<String, Integer> hm = new HashMap<>();
hm.put("B", 2); hm.put("A", 1); hm.put("C", 3);
System.out.println(hm);   // {A=1, B=2, C=3} — unordered

Map<String, Integer> tm = new TreeMap<>(hm);
System.out.println(tm);   // {A=1, B=2, C=3} — always sorted by key
```

---

## 7. How does `HashMap` work internally?

`HashMap` uses an **array of buckets** (Node[] table). Each key is hashed:

1. `hash = key.hashCode() ^ (hash >>> 16)`
2. `index = hash & (capacity - 1)`
3. Entry stored in bucket; collisions handled by a **linked list** (Java 8+: converted to **red-black tree** when bucket size ≥ 8)

```java
Map<String, Integer> map = new HashMap<>();
map.put("apple", 1);   // bucket = hash("apple") % capacity
map.put("banana", 2);
map.get("apple");      // same hash → same bucket → found
```

---

## 8. What is the load factor and initial capacity in `HashMap`?

- **Initial capacity**: number of buckets (default **16**)
- **Load factor**: threshold ratio before resize (default **0.75**)
- **Resize trigger**: when `size > capacity × loadFactor` → capacity doubles

```java
// Custom capacity and load factor
Map<String, Integer> map = new HashMap<>(32, 0.5f);
```

---

## 9. What happens when two keys have the same `hashCode()`?

A **collision** occurs. The keys land in the same bucket and are stored as a linked list (or tree if ≥ 8 entries). `equals()` is used to find the exact key during lookup.

```java
// "FB" and "Ea" have the same hashCode in Java
System.out.println("FB".hashCode());   // 2236
System.out.println("Ea".hashCode());   // 2236

Map<String, Integer> map = new HashMap<>();
map.put("FB", 1);
map.put("Ea", 2);
System.out.println(map.size());  // 2 — both stored, different equals()
```

---

## 10. Why must you override both `equals()` and `hashCode()` for HashMap keys?

The contract: if `a.equals(b)` then `a.hashCode() == b.hashCode()`. Breaking this causes keys to be "lost" in a HashMap.

```java
class BadKey {
    int id;
    BadKey(int id) { this.id = id; }
    // equals() overridden but hashCode() NOT overridden ← BROKEN
    @Override public boolean equals(Object o) { return ((BadKey) o).id == id; }
}

Map<BadKey, String> map = new HashMap<>();
BadKey k1 = new BadKey(1);
map.put(k1, "hello");
System.out.println(map.get(new BadKey(1)));  // null! — different hashCode
```

**Fix**: always override both:
```java
@Override public int hashCode() { return id; }
```

---

## 11. What is `ConcurrentHashMap` and how is it different from `HashMap`?

| Feature | `HashMap` | `ConcurrentHashMap` |
|---|---|---|
| Thread-safe | No | Yes |
| Null keys/values | 1 null key allowed | Not allowed |
| Locking | None | Segment/bucket-level lock |
| Performance | Fastest single-thread | Best concurrent |

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
// Safe to use from multiple threads without external synchronization
```

---

## 12. What is the difference between `HashMap` and `Hashtable`?

| Feature | `HashMap` | `Hashtable` |
|---|---|---|
| Thread-safe | No | Yes (fully synchronized) |
| Null key/value | Allowed | Not allowed |
| Performance | Faster | Slower |
| Legacy | No | Yes (Java 1.0) |

> Prefer `ConcurrentHashMap` over `Hashtable` in modern code.

---

## 13. What is `List.of()` vs `new ArrayList<>()`?

| Feature | `List.of()` | `new ArrayList<>()` |
|---|---|---|
| Mutable | ❌ Immutable | ✅ Mutable |
| Null elements | ❌ Not allowed | ✅ Allowed |
| Introduced | Java 9 | Java 2 |

```java
List<String> immutable = List.of("A", "B", "C");
// immutable.add("D");  ❌ UnsupportedOperationException

List<String> mutable = new ArrayList<>(immutable);
mutable.add("D");        // ✅ works
```

---

## 14. What is the difference between `Iterator` and `ListIterator`?

| Feature | `Iterator` | `ListIterator` |
|---|---|---|
| Direction | Forward only | Forward + backward |
| Applicable to | Any `Collection` | `List` only |
| Add elements | No | Yes |
| Index access | No | Yes (`nextIndex()`) |

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
ListIterator<String> it = list.listIterator(list.size());

while (it.hasPrevious()) {
    System.out.print(it.previous() + " "); // C B A
}
```

---

## 15. What is a `fail-fast` iterator?

A fail-fast iterator throws `ConcurrentModificationException` if the collection is structurally modified after the iterator is created (except through the iterator's own `remove()`).

```java
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
Iterator<String> it = list.iterator();

while (it.hasNext()) {
    it.next();
    list.add("D");  // ❌ ConcurrentModificationException
}

// Safe removal:
while (it.hasNext()) {
    if (it.next().equals("B")) it.remove();  // ✅ OK
}
```

---

## 16. What is a `fail-safe` iterator?

Iterates over a **copy** of the collection — no `ConcurrentModificationException`. Used in `java.util.concurrent` classes.

```java
List<String> list = new CopyOnWriteArrayList<>(List.of("A", "B", "C"));
for (String s : list) {
    list.add("D");   // ✅ No exception — iterates over snapshot
    System.out.print(s + " "); // A B C
}
```

---

## 17. How do you remove elements while iterating a List?

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5));

// Option 1: Iterator.remove()
Iterator<Integer> it = nums.iterator();
while (it.hasNext()) {
    if (it.next() % 2 == 0) it.remove();
}

// Option 2: removeIf() — Java 8+
nums.removeIf(n -> n % 2 == 0);

System.out.println(nums);  // [1, 3, 5]
```

---

## 18. What is a `PriorityQueue`?

A queue where elements are dequeued in **natural order** (or via `Comparator`) — not FIFO. Backed by a **min-heap** by default.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5); pq.offer(1); pq.offer(3);

while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " "); // 1 3 5 — min first
}

// Max-heap:
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Comparator.reverseOrder());
```

---

## 19. What is `ArrayDeque` and when to use it over `Stack`?

`ArrayDeque` is a resizable array-based double-ended queue. It is the **preferred** stack and queue implementation — faster than `Stack` (synchronized) and `LinkedList`.

```java
Deque<String> stack = new ArrayDeque<>();
stack.push("A");
stack.push("B");
System.out.println(stack.pop());   // B — LIFO

Deque<String> queue = new ArrayDeque<>();
queue.offer("X");
queue.offer("Y");
System.out.println(queue.poll());  // X — FIFO
```

---

## 20. What is the difference between `poll()`, `peek()`, and `remove()` in Queue?

| Method | Behavior on empty queue |
|---|---|
| `poll()` | Returns `null` |
| `peek()` | Returns `null` |
| `remove()` | Throws `NoSuchElementException` |
| `element()` | Throws `NoSuchElementException` |

```java
Queue<String> q = new LinkedList<>();
System.out.println(q.poll());    // null
System.out.println(q.peek());    // null
// System.out.println(q.remove()); // ❌ NoSuchElementException
```

---

## 21. How does `TreeMap` maintain sorted order?

`TreeMap` is backed by a **Red-Black Tree** (self-balancing BST). Keys must implement `Comparable` or a `Comparator` must be provided.

```java
Map<String, Integer> tm = new TreeMap<>(Comparator.reverseOrder());
tm.put("Banana", 2); tm.put("Apple", 1); tm.put("Cherry", 3);
System.out.println(tm); // {Cherry=3, Banana=2, Apple=1} — reverse sorted
```

---

## 22. What is `NavigableMap` and useful `TreeMap` methods?

`NavigableMap` (implemented by `TreeMap`) adds navigation methods.

```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(1, "one"); tm.put(3, "three"); tm.put(5, "five"); tm.put(7, "seven");

System.out.println(tm.floorKey(4));      // 3 — largest key ≤ 4
System.out.println(tm.ceilingKey(4));    // 5 — smallest key ≥ 4
System.out.println(tm.headMap(5));       // {1=one, 3=three}
System.out.println(tm.tailMap(5));       // {5=five, 7=seven}
System.out.println(tm.subMap(3, 7));     // {3=three, 5=five}
```

---

## 23. What is `EnumMap`?

A highly efficient `Map` implementation specifically for `Enum` keys. Internally uses an array indexed by enum ordinal.

```java
enum Day { MON, TUE, WED, THU, FRI }

Map<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MON, "Team standup");
schedule.put(Day.FRI, "Sprint review");
System.out.println(schedule.get(Day.MON)); // Team standup
```

---

## 24. What is `EnumSet`?

A `Set` implementation for enum types — extremely compact (uses a bit vector internally).

```java
EnumSet<Day> weekend = EnumSet.of(Day.MON, Day.TUE);
EnumSet<Day> weekdays = EnumSet.complementOf(weekend);
System.out.println(weekdays); // [WED, THU, FRI]
```

---

## 25. What is `Collections.unmodifiableList()`?

Returns a view that throws `UnsupportedOperationException` on write operations.

```java
List<String> mutable = new ArrayList<>(List.of("A", "B", "C"));
List<String> readonly = Collections.unmodifiableList(mutable);

// readonly.add("D");  // ❌ UnsupportedOperationException
mutable.add("D");
System.out.println(readonly); // [A, B, C, D] — reflects underlying list changes
```

> For truly immutable lists, use `List.copyOf()` (Java 10+).

---

## 26. What is `Collections.synchronizedList()`?

Wraps a list to make all operations synchronized (thread-safe). Iteration still requires external synchronization.

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

synchronized (syncList) {           // must synchronize iteration
    for (String s : syncList) { }
}
```

---

## 27. What is `CopyOnWriteArrayList`?

A thread-safe `List` where all mutating operations create a **fresh copy** of the underlying array. Ideal for read-heavy, rarely-modified lists.

```java
List<String> list = new CopyOnWriteArrayList<>(List.of("A", "B"));
for (String s : list) {
    list.add("C");   // ✅ No ConcurrentModificationException
    System.out.print(s + " "); // A B — iterates original snapshot
}
```

---

## 28. How do you sort a `List` of custom objects?

```java
class Employee {
    String name;
    int salary;
    Employee(String n, int s) { name = n; salary = s; }
    public String toString() { return name + "(" + salary + ")"; }
}

List<Employee> emps = new ArrayList<>(List.of(
    new Employee("Alice", 80000),
    new Employee("Bob",   60000),
    new Employee("Carol", 70000)
));

// By salary ascending
emps.sort(Comparator.comparingInt(e -> e.salary));
System.out.println(emps); // [Bob(60000), Carol(70000), Alice(80000)]

// By name, then salary descending
emps.sort(Comparator.comparing((Employee e) -> e.name)
                    .thenComparingInt((Employee e) -> e.salary).reversed());
```

---

## 29. What is the difference between `Comparable` and `Comparator`?

| | `Comparable` | `Comparator` |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T o1, T o2)` |
| Location | Inside the class | External / lambda |
| Use case | Natural ordering | Custom / multiple orderings |

```java
class Product implements Comparable<Product> {
    String name; int price;
    Product(String n, int p) { name = n; price = p; }

    @Override
    public int compareTo(Product o) { return this.price - o.price; } // natural: by price
}

List<Product> products = new ArrayList<>(List.of(
    new Product("TV", 500), new Product("Phone", 300)));
Collections.sort(products);                          // uses Comparable
products.sort(Comparator.comparing(p -> p.name));    // uses Comparator
```

---

## 30. What is the difference between `Collections.sort()` and `List.sort()`?

Both use **TimSort** (O(n log n)), but:
- `Collections.sort(list)` — static utility method, modifies list
- `list.sort(comparator)` — instance method on `List` (Java 8+), more fluent

```java
List<Integer> nums = new ArrayList<>(List.of(3, 1, 4, 1, 5));
Collections.sort(nums);                          // [1, 1, 3, 4, 5]
nums.sort(Comparator.reverseOrder());            // [5, 4, 3, 1, 1]
```

---

## 31. What is `Map.getOrDefault()` and `Map.computeIfAbsent()`?

```java
Map<String, Integer> wordCount = new HashMap<>();
String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};

// getOrDefault
for (String w : words) {
    wordCount.put(w, wordCount.getOrDefault(w, 0) + 1);
}

// computeIfAbsent — creates value only if key absent
Map<String, List<String>> groups = new HashMap<>();
groups.computeIfAbsent("fruits", k -> new ArrayList<>()).add("apple");
groups.computeIfAbsent("fruits", k -> new ArrayList<>()).add("banana");
System.out.println(groups); // {fruits=[apple, banana]}
```

---

## 32. What is `Map.merge()`?

Merges a value into a map — inserts if absent, applies a function if present.

```java
Map<String, Integer> scores = new HashMap<>();
String[] players = {"Alice", "Bob", "Alice", "Alice", "Bob"};

for (String p : players) {
    scores.merge(p, 1, Integer::sum);  // add 1 on each occurrence
}

System.out.println(scores); // {Alice=3, Bob=2}
```

---

## 33. What is `Map.forEach()` and `Map.entrySet()`?

```java
Map<String, Integer> map = Map.of("A", 1, "B", 2, "C", 3);

// entrySet — most efficient for key+value access
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " -> " + e.getValue());
}

// forEach — Java 8+
map.forEach((k, v) -> System.out.println(k + " = " + v));
```

---

## 34. How do you find duplicate elements in a List?

```java
List<Integer> nums = List.of(1, 2, 3, 2, 4, 3, 5);

Set<Integer> seen = new HashSet<>();
Set<Integer> duplicates = new LinkedHashSet<>();

for (int n : nums) {
    if (!seen.add(n)) duplicates.add(n);
}

System.out.println(duplicates); // [2, 3]
```

---

## 35. How do you count frequencies of elements?

```java
List<String> fruits = List.of("apple", "banana", "apple", "cherry", "banana", "apple");

Map<String, Long> freq = fruits.stream()
    .collect(Collectors.groupingBy(s -> s, Collectors.counting()));

System.out.println(freq); // {apple=3, banana=2, cherry=1}
```

---

## 36. How do you get the top N entries from a Map by value?

```java
Map<String, Integer> scores = Map.of("Alice", 85, "Bob", 92, "Carol", 78, "Dave", 92);

scores.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .limit(2)
    .forEach(e -> System.out.println(e.getKey() + ": " + e.getValue()));
// Bob: 92
// Dave: 92
```

---

## 37. What is `WeakHashMap`?

A `Map` where keys are held with **weak references** — entries are automatically removed when the key is garbage collected. Useful for caches.

```java
Map<Object, String> cache = new WeakHashMap<>();
Object key = new Object();
cache.put(key, "value");
System.out.println(cache.size()); // 1

key = null;                        // key eligible for GC
System.gc();
System.out.println(cache.size()); // 0 — entry removed
```

---

## 38. What is `IdentityHashMap`?

Uses `==` (reference equality) instead of `equals()` for key comparison.

```java
Map<String, String> map = new IdentityHashMap<>();
String a = new String("key");
String b = new String("key");

map.put(a, "value-a");
map.put(b, "value-b");

System.out.println(map.size());   // 2 — a != b by reference
System.out.println(map.get(a));   // value-a
```

---

## 39. What is `BlockingQueue`?

A thread-safe queue that blocks on `take()` when empty and on `put()` when full. Used in producer-consumer patterns.

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(3);

// Producer thread
new Thread(() -> {
    try {
        queue.put(1); queue.put(2); queue.put(3);
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
}).start();

// Consumer thread
new Thread(() -> {
    try {
        System.out.println(queue.take()); // blocks until item available
    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
}).start();
```

---

## 40. What are the `Collections` utility methods?

```java
List<Integer> list = new ArrayList<>(List.of(3, 1, 4, 1, 5, 9, 2));

Collections.sort(list);                        // [1, 1, 2, 3, 4, 5, 9]
Collections.reverse(list);                    // [9, 5, 4, 3, 2, 1, 1]
Collections.shuffle(list);                    // random order
System.out.println(Collections.min(list));    // smallest element
System.out.println(Collections.max(list));    // largest element
System.out.println(Collections.frequency(list, 1));  // count of 1s

Collections.fill(list, 0);                   // set all to 0
Collections.nCopies(5, "x");                 // [x, x, x, x, x]
Collections.swap(list, 0, 1);                // swap index 0 and 1
Collections.disjoint(List.of(1,2), List.of(3,4)); // true if no common elements
```

---

## 41. How do you convert between arrays and Lists?

```java
// Array → List (fixed-size)
String[] arr = {"A", "B", "C"};
List<String> fixed = Arrays.asList(arr);         // backed by array, no add/remove

// Array → List (mutable)
List<String> mutable = new ArrayList<>(Arrays.asList(arr));
mutable.add("D");

// Java 9+
List<String> immutable = List.of(arr);

// List → Array
String[] back = mutable.toArray(new String[0]);
System.out.println(Arrays.toString(back));       // [A, B, C, D]
```

---

## 42. How do you reverse a List?

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4, 5));

// Option 1
Collections.reverse(list);
System.out.println(list); // [5, 4, 3, 2, 1]

// Option 2 — Stream
List<Integer> reversed = list.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
```

---

## 43. How do you find the most frequent element in a List?

```java
List<String> items = List.of("cat", "dog", "cat", "bird", "cat", "dog");

String mostFrequent = items.stream()
    .collect(Collectors.groupingBy(s -> s, Collectors.counting()))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .map(Map.Entry::getKey)
    .orElseThrow();

System.out.println(mostFrequent); // cat
```

---

## 44. How do you group a List of objects by a property?

```java
record Person(String name, String city) {}

List<Person> people = List.of(
    new Person("Alice", "NYC"),
    new Person("Bob",   "LA"),
    new Person("Carol", "NYC"),
    new Person("Dave",  "LA")
);

Map<String, List<Person>> byCity =
    people.stream().collect(Collectors.groupingBy(Person::city));

byCity.forEach((city, persons) ->
    System.out.println(city + ": " + persons.stream().map(Person::name).toList()));
// NYC: [Alice, Carol]
// LA:  [Bob, Dave]
```

---

## 45. What is the difference between `Stack` and `Deque` for stack operations?

| | `Stack` | `ArrayDeque` as Stack |
|---|---|---|
| Thread-safe | Yes (synchronized) | No |
| Performance | Slower | Faster |
| Extends | `Vector` | `AbstractCollection` |
| Recommended | ❌ Legacy | ✅ Preferred |

```java
Deque<String> stack = new ArrayDeque<>();
stack.push("first");
stack.push("second");
stack.push("third");

System.out.println(stack.peek()); // third — top without removing
System.out.println(stack.pop());  // third — LIFO
System.out.println(stack.pop());  // second
```

---

## 46. What is `LinkedHashMap` and how to use it as an LRU Cache?

`LinkedHashMap` maintains insertion order (or access order when `accessOrder = true`), enabling LRU cache behavior.

```java
int CAPACITY = 3;
Map<String, Integer> lruCache = new LinkedHashMap<>(CAPACITY, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {
        return size() > CAPACITY;
    }
};

lruCache.put("A", 1); lruCache.put("B", 2); lruCache.put("C", 3);
lruCache.get("A");     // access A — moves to end (most recent)
lruCache.put("D", 4);  // evicts B (least recently used)

System.out.println(lruCache.keySet()); // [C, A, D]
```

---

## 47. How do you make a collection immutable in Java?

```java
// Java 9+ — truly unmodifiable
List<String>  list = List.of("A", "B", "C");
Set<String>   set  = Set.of("X", "Y", "Z");
Map<String, Integer> map = Map.of("one", 1, "two", 2);

// Java 10+ — unmodifiable copy
List<String> original = new ArrayList<>(List.of("A", "B"));
List<String> copy = List.copyOf(original);

// Guava-style (Collections utility)
List<String> unmod = Collections.unmodifiableList(new ArrayList<>(List.of("A")));
```

---

## 48. How do you implement a frequency map (word count)?

```java
String text = "the cat sat on the mat the cat";
String[] words = text.split(" ");

// Classic approach
Map<String, Integer> freq = new HashMap<>();
for (String w : words) freq.merge(w, 1, Integer::sum);

// Stream approach
Map<String, Long> freq2 = Arrays.stream(words)
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));

System.out.println(freq);  // {the=3, cat=2, sat=1, on=1, mat=1}
```

---

## 49. What is the performance complexity summary of common collections?

| Operation | `ArrayList` | `LinkedList` | `HashSet` | `TreeSet` | `HashMap` | `TreeMap` |
|---|---|---|---|---|---|---|
| Add | O(1) amortized | O(1) | O(1) avg | O(log n) | O(1) avg | O(log n) |
| Remove | O(n) | O(1)* | O(1) avg | O(log n) | O(1) avg | O(log n) |
| Get/Contains | O(1) / O(n) | O(n) | O(1) avg | O(log n) | O(1) avg | O(log n) |
| Ordered | Yes | Yes | No | Yes | No | Yes |

*O(1) if node reference is known

---

## 50. What are best practices when using Java Collections?

```java
// 1. Program to interfaces, not implementations
List<String> list = new ArrayList<>();       // ✅
// ArrayList<String> list = new ArrayList<>(); // ❌

// 2. Use diamond operator and var
var map = new HashMap<String, List<Integer>>();

// 3. Prefer isEmpty() over size() == 0
list.isEmpty();  // ✅ — O(1) and more readable

// 4. Use computeIfAbsent for complex map values
map.computeIfAbsent("key", k -> new ArrayList<>()).add(42);

// 5. Specify initial capacity for large collections
Map<String, Integer> large = new HashMap<>(1000);

// 6. Prefer for-each over index loop for collections
for (String s : list) { }  // ✅ cleaner and works for all Iterables

// 7. Use streams for functional pipelines
list.stream()
    .filter(s -> s.startsWith("A"))
    .sorted()
    .collect(Collectors.toList());

// 8. Use ConcurrentHashMap in multi-threaded code
Map<String, Integer> concurrent = new ConcurrentHashMap<>();
```

---

*Good luck in your interview! 🚀*
