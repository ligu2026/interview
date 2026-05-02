# Top 100 Java Tricky Interview Questions on Map, List, Set with Answers and Examples

## 1. Can a `HashSet` contain duplicate elements?  
**Answer:** No. `HashSet` does not allow duplicates. It uses `equals()` and `hashCode()` to check for uniqueness.  
**Example:**  
```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("A"); // ignored
System.out.println(set.size()); // 1
```

## 2. What happens if you add an element to a `TreeSet` that doesn’t implement `Comparable` and no `Comparator` is provided?  
**Answer:** `ClassCastException` at runtime because `TreeSet` needs to order elements.  
**Example:**  
```java
class Person {}
Set<Person> set = new TreeSet<>();
set.add(new Person()); // ClassCastException
```

## 3. Can you add `null` to a `TreeSet`?  
**Answer:** Yes, but only once, and only if the `TreeSet` is empty or the comparator allows null. Since Java 7, adding null to a non-empty `TreeSet` throws `NullPointerException` because comparison fails.  
**Example:**  
```java
TreeSet<String> set = new TreeSet<>();
set.add(null); // OK if empty
set.add("A");
set.add(null); // NullPointerException
```

## 4. How does `HashMap` handle hash collisions?  
**Answer:** Java 8+ uses a balanced tree (TreeNode) for bins with many collisions (≥8 entries) to improve worst-case performance from O(n) to O(log n). Before that or for fewer collisions, it uses linked list chaining.

## 5. What is the initial capacity of `HashMap`?  
**Answer:** Default initial capacity is **16** (must be power of two). Load factor is 0.75.

## 6. If two objects are equal (by `equals()`), what must be true about their `hashCode()`?  
**Answer:** They must produce the same `hashCode()` value. Otherwise, `HashMap`/`HashSet` will not recognize them as duplicates.

## 7. Can a `HashMap` have multiple keys mapping to the same value?  
**Answer:** Yes. Values can be duplicated; only keys must be unique.

## 8. What is the difference between `fail-fast` and `fail-safe` iterators?  
**Answer:** Fail-fast iterators (e.g., `ArrayList`, `HashSet`) throw `ConcurrentModificationException` if the collection is modified structurally after iterator creation. Fail-safe iterators (e.g., `ConcurrentHashMap`, `CopyOnWriteArrayList`) work on a clone and don’t throw exceptions.

## 9. Does `HashSet` maintain insertion order?  
**Answer:** No. Use `LinkedHashSet` for insertion order.

## 10. What is the purpose of `Collections.unmodifiableList()`?  
**Answer:** Returns a read-only view. Any modification attempt throws `UnsupportedOperationException`.

## 11. Can you modify a list while iterating over it using a for-each loop?  
**Answer:** No. It will throw `ConcurrentModificationException`. Use `Iterator.remove()` or `ListIterator`.  
**Example:**  
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
for (Integer i : list) {
    if (i == 2) list.remove(i); // ConcurrentModificationException
}
```

## 12. What is the difference between `ArrayList` and `LinkedList`?  
**Answer:** `ArrayList` uses dynamic array → fast random access, slow insert/delete in middle. `LinkedList` uses doubly-linked list → slow random access, fast insert/delete at ends/middle if you have reference.

## 13. When would you use `LinkedHashMap` over `HashMap`?  
**Answer:** When you need predictable iteration order (insertion-order or access-order for LRU caches).

## 14. What is the time complexity of `contains()` on a `HashSet`?  
**Answer:** O(1) average, but O(n) in worst-case due to collisions.

## 15. How does `HashMap` compute the bucket index?  
**Answer:** `hash = key.hashCode()` then `(hash ^ (hash >>> 16)) & (capacity - 1)`.

## 16. Can you store `null` keys in `HashMap`? How many?  
**Answer:** Yes, one `null` key (stored in bucket 0). Multiple `null` values allowed.

## 17. What happens if you use a mutable object as a key in `HashMap` and then modify it?  
**Answer:** The key’s `hashCode` changes, so the entry becomes unfindable. Memory leak possible.

## 18. How to sort a `List` of custom objects?  
**Answer:** Implement `Comparable` or provide `Comparator` to `Collections.sort()` or `List.sort()`.

## 19. What is the difference between `Set` and `List`?  
**Answer:** `Set` does not allow duplicates, no order guarantee (except `LinkedHashSet`, `TreeSet`). `List` allows duplicates and maintains insertion order (indexed).

## 20. Can a `TreeSet` contain `null` if a custom `Comparator` handles it?  
**Answer:** Yes, but the comparator must be able to compare `null` with other elements. Example:  
```java
Comparator<String> nullFriendly = Comparator.nullsFirst(Comparator.naturalOrder());
TreeSet<String> set = new TreeSet<>(nullFriendly);
set.add(null);
set.add("B");
System.out.println(set); // [null, B]
```

## 21. What is the difference between `pollFirst()` and `removeFirst()` in `TreeSet`?  
**Answer:** `pollFirst()` returns `null` if set empty; `removeFirst()` throws `NoSuchElementException`.

## 22. Can we get a `ConcurrentModificationException` from `ConcurrentHashMap`?  
**Answer:** No. Its iterators are weakly consistent, not fail-fast.

## 23. How does `ConcurrentHashMap` achieve thread safety without locking the whole map?  
**Answer:** Uses bucket-level locks (striped locking) and CAS operations. Since Java 8, uses `synchronized` on specific nodes.

## 24. What is `IdentityHashMap` used for?  
**Answer:** Uses reference equality (`==`) instead of `equals()` for keys. Useful for serialization or object graph traversal.

## 25. Does `LinkedList` implement `Deque`?  
**Answer:** Yes. It can be used as stack, queue, or double-ended queue.

## 26. What is the performance of `ArrayList.remove(int)` vs `LinkedList.remove(int)`?  
**Answer:** `ArrayList` shifts elements → O(n). `LinkedList` must traverse to index → O(n) but no shifting.

## 27. How to make a `List` unmodifiable?  
**Answer:** `Collections.unmodifiableList(list)` or `List.of(...)` (Java 9+).

## 28. Difference between `EnumSet` and `HashSet`?  
**Answer:** `EnumSet` is extremely fast and memory efficient, but only stores enum constants. It uses bit vectors.

## 29. When does `HashMap` resize?  
**Answer:** When number of entries exceeds `capacity * loadFactor`. It doubles the capacity and rehashes.

## 30. Can we use `LinkedHashSet` to maintain insertion order?  
**Answer:** Yes. It extends `HashSet` and uses `LinkedHashMap` internally to preserve order.

## 31. Why is `HashMap` not thread-safe?  
**Answer:** No internal synchronization. Concurrent modifications can lead to infinite loops (old Java versions) or lost updates. Use `ConcurrentHashMap`.

## 32. How to synchronize a `HashMap`?  
**Answer:** `Collections.synchronizedMap(new HashMap<>())`. But better to use `ConcurrentHashMap`.

## 33. What is a `WeakHashMap`?  
**Answer:** Keys are weak references. If a key is only referenced inside the map, GC can remove it. Useful for caches.

## 34. What is the initial size of `ArrayList` when created using default constructor?  
**Answer:** Since Java 8, an empty array (lazy initialization). First `add()` creates capacity 10.

## 35. What is `Arrays.asList()` return type?  
**Answer:** Returns a fixed-size `List` backed by the array. Not `ArrayList`; it's an internal `Arrays.ArrayList`. Cannot add/remove, but can set elements.

## 36. How to convert an array to a mutable `ArrayList`?  
**Answer:** `new ArrayList<>(Arrays.asList(array))`.

## 37. What is `subList()` behavior?  
**Answer:** Returns a view backed by the original list. Changes in view reflect in original, and vice versa. Structural modifications in original make view invalid.

## 38. What is the difference between `size()` and `capacity` of `ArrayList`?  
**Answer:** `size()` = number of elements. Capacity = current internal array length; not directly accessible.

## 39. Can `HashSet` be sorted?  
**Answer:** Not directly. Convert to `TreeSet` or `List` and sort.

## 40. How `HashMap` get() works internally?  
**Answer:** Computes hash, finds bucket, then traverses linked list/tree checking `equals()`.

## 41. What is `NavigableMap`?  
**Answer:** Interface extended by `TreeMap` providing methods like `lowerKey()`, `ceilingKey()`, `subMap()`.

## 42. What is `Load factor` in `HashMap`?  
**Answer:** Measures how full map can be before resizing. Default 0.75 balances time and space.

## 43. Why are buckets in `HashMap` sized as power-of-two?  
**Answer:** To use bitwise AND (`&`) instead of modulo for faster bucket index calculation.

## 44. What is the difference between `Map.compute()` and `Map.merge()`?  
**Answer:** `compute(k, (k, v) -> newV)` always computes new value. `merge(k, val, remappingFunction)` uses default if key absent.

## 45. How to iterate over a `Map` in insertion order?  
**Answer:** Use `LinkedHashMap` and iterate over `entrySet()`.

## 46. What is the difference between `peek()` and `poll()` in `Queue`?  
**Answer:** `peek()` returns null if empty; `poll()` returns null if empty; `element()` and `remove()` throw exception.

## 47. Can a `List` be retrieved from a `Set`?  
**Answer:** Not directly, but you can create `new ArrayList<>(set)`.

## 48. What is the purpose of `TreeMap` when we have `HashMap`?  
**Answer:** `TreeMap` provides sorted order, navigation methods, and can be extended with custom comparators.

## 49. What is `CopyOnWriteArrayList`?  
**Answer:** Thread-safe variant where all mutative operations create a new copy of the array. Iterators reflect state at creation, no `ConcurrentModificationException`.

## 50. What is `ArrayDeque` and why prefer it over `Stack`?  
**Answer:** Resizable-array implementation of `Deque`. Faster than `Stack` (which is vector-based) and supports both LIFO and FIFO.

## 51. Can you add an element at a specific index in `LinkedList`?  
**Answer:** Yes, `add(index, element)`. But must traverse to that index → O(n).

## 52. What is the difference between `remove()` of `Collection` and `Iterator`?  
**Answer:** `Iterator.remove()` is the only safe way to remove during iteration. `Collection.remove()` may cause `ConcurrentModificationException`.

## 53. Does `ArrayList` implement `RandomAccess` interface?  
**Answer:** Yes. So indexed access is O(1). `LinkedList` does not.

## 54. What is the memory overhead of `HashSet` compared to `ArrayList`?  
**Answer:** `HashSet` has higher memory because each entry stores key, hash, next pointer, plus the `HashMap` internal structure.

## 55. Can we make a `List` of primitive types?  
**Answer:** No. Generics work with objects. Use wrapper classes (`Integer`, `Double`). Boxing/unboxing is automatic.

## 56. What is a `BlockingQueue`?  
**Answer:** Queue that waits for queue to become non-empty when retrieving, and waits for space when adding. Implementations: `ArrayBlockingQueue`, `LinkedBlockingQueue`.

## 57. What is the difference between `ConcurrentSkipListSet` and `TreeSet`?  
**Answer:** `ConcurrentSkipListSet` is thread-safe, sorted, based on skip list. `TreeSet` is unsynchronized.

## 58. How does `LinkedHashMap` implement LRU cache?  
**Answer:** Construct with `accessOrder=true`. Then `get` operations reorder entries. Override `removeEldestEntry` to automatically remove oldest.

## 59. What is the output of `set.remove(set.iterator().next())`?  
**Answer:** Removes first element, provided iterator's `next()` returns element. Safe because using iterator's `remove` is allowed.

## 60. Why does `HashSet` not have a `get()` method?  
**Answer:** Because sets don't map keys to values. To check membership use `contains()`.

## 61. What is the difference between `Collections.emptyList()` and `List.of()`?  
**Answer:** Both return immutable empty list. `emptyList()` is type-inferred, older. `List.of()` from Java 9 also returns immutable but with varargs.

## 62. How to find duplicate elements in a `List` using `Set`?  
**Answer:** Add to set, if `add` returns false, it's duplicate.

## 63. What is the most efficient way to remove duplicates from a `List` while preserving order?  
**Answer:** Use `LinkedHashSet`.  
**Example:**  
```java
List<Integer> list = Arrays.asList(1,2,2,3);
Set<Integer> set = new LinkedHashSet<>(list);
list.clear();
list.addAll(set);
```

## 64. What is `Spliterator`?  
**Answer:** Interface for parallel traversal; used by Stream API internally to split input into chunks.

## 65. Does `HashMap` allow keys that are equal but with different `hashCode()`?  
**Answer:** No. If `equals()` returns true, `hashCode()` must be equal. Violation leads to duplicates.

## 66. What happens if a `HashSet`’s element modifies its `hashCode` after insertion?  
**Answer:** The element may be lost (not found, duplicates possible). Not recommended.

## 67. What is the difference between `EnumMap` and `HashMap` with enum keys?  
**Answer:** `EnumMap` uses array and is extremely fast, no hashing, keys are enum constants.

## 68. Can two different keys with same `hashCode()` be stored in `HashMap`?  
**Answer:** Yes, they are stored in the same bucket chained or treed.

## 69. Is `TreeMap` slower than `HashMap`?  
**Answer:** Generally yes for basic operations (log n vs O(1) avg). But `TreeMap` offers sorted order.

## 70. What is a `BitSet`?  
**Answer:** Not part of `Collection` framework but used as a set of bits. Useful for flag sets.

## 71. What does `Map.putIfAbsent` return?  
**Answer:** Returns previous value associated with key, or null if none.

## 72. How to get a synchronized `Set` from `HashSet`?  
**Answer:** `Collections.synchronizedSet(new HashSet<>())`.

## 73. What is the `Deque` method to add at the front?  
**Answer:** `addFirst(e)`, `offerFirst(e)`. For a stack: `push(e)`.

## 74. Can `LinkedList` be used as a `Queue`?  
**Answer:** Yes. Implements `Queue` interface. `offer()`, `poll()`, `peek()`.

## 75. What is the output of `list.remove(1)` vs `list.remove((Integer)1)`?  
**Answer:** `list.remove(1)` removes element at index 1. `list.remove((Integer)1)` removes object 1 (first occurrence) because `remove(Object)` is used.

## 76. What is the difference between `Set.of()` (Java 9) and `Collections.unmodifiableSet()`?  
**Answer:** `Set.of()` returns an immutable set, cannot hold null. `unmodifiableSet` wraps a modifiable set; the original can still be changed unless made unmodifiable.

## 77. Why does `TreeSet` not allow adding an element that is not comparable with existing ones?  
**Answer:** Because it needs to maintain sorted order; type mismatch breaks comparison.

## 78. Can a `Map` be iterated using `forEach`?  
**Answer:** Yes, `map.forEach((k, v) -> {})`.

## 79. What is the purpose of `isEmpty()` vs `size() == 0`?  
**Answer:** Semantically same, but `isEmpty()` may be more efficient for some collections (e.g., `ConcurrentLinkedQueue` uses a different check).

## 80. What is a `Multiset`?  
**Answer:** Not in Java standard library. Guava's `Multiset` allows duplicates with counts.

## 81. How to convert a `List` of strings to a comma-separated string?  
**Answer:** `String.join(",", list)` or using stream: `list.stream().collect(Collectors.joining(","))`.

## 82. Can we have a `HashMap` that maps to a `List`?  
**Answer:** Yes e.g., `Map<String, List<Integer>>`. Often used for multi-value mapping.

## 83. What is the difference between `replace` and `put` in `Map`?  
**Answer:** `replace(k, v)` only replaces if key already mapped to some value.

## 84. What is `computeIfAbsent` useful for?  
**Answer:** To lazily initialize a value, often used for caching or map of lists.  
**Example:**  
```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

## 85. How does `PriorityQueue` order elements?  
**Answer:** Uses natural order or comparator. Not FIFO; always returns smallest (or highest) element.

## 86. Can `PriorityQueue` contain `null`?  
**Answer:** No, because `null` cannot be compared.

## 87. What is the time complexity of `contains()` in `TreeSet`?  
**Answer:** O(log n).

## 88. How to get a reverse order view of a `TreeSet`?  
**Answer:** `treeSet.descendingSet()`.

## 89. What is the difference between `ListIterator` and `Iterator`?  
**Answer:** `ListIterator` allows bidirectional traversal, setting/adding elements, and getting index.

## 90. Can you use `Arrays.sort()` on a `List`?  
**Answer:** No, use `Collections.sort(list)` or `list.sort(null)`.

## 91. What is `Collections.checkedMap()` used for?  
**Answer:** To get a dynamically type-safe view that enforces type checks at runtime (helpful with legacy code mixing raw types).

## 92. What output? `Set<Short> set = new HashSet<>(); for (short i = 0; i < 100; i++) set.add(i); set.remove(1); System.out.println(set.size());`  
**Answer:** 100. `1` is int, autoboxed to Integer, not Short, so no removal. Cast to `(short)1` works.

## 93. In `TreeMap.ceilingKey()` what is returned if no such key?  
**Answer:** Null.

## 94. What is `ConcurrentSkipListMap`?  
**Answer:** Thread-safe sorted map based on skip list. Provides log(n) operations.

## 95. Why does `HashMap` require keys to be immutable or effectively immutable?  
**Answer:** To prevent hash code changes causing loss of entries.

## 96. What is the difference between `Vector` and `ArrayList`?  
**Answer:** `Vector` is legacy, synchronized, slower. `ArrayList` not synchronized. Both resizable arrays.

## 97. What is `Stack` class issue?  
**Answer:** It extends `Vector`, not a contract-based stack. `Deque` recommended.

## 98. What does `Collections.singletonList(e)` return?  
**Answer:** Immutable list with exactly one element. Efficient.

## 99. How to create a `List` with initial elements in one line?  
**Answer:** `List.of(1,2,3)` (Java 9+) or `Arrays.asList(1,2,3)`.

## 100. What is the difference between `Map.entrySet()` vs `Map.keySet()`?  
**Answer:** `entrySet()` gives `Map.Entry` objects (key+value) for iteration; `keySet()` gives only keys.

---

*These questions cover common tricky areas including concurrency, hashing, equality, ordering, performance, and API subtleties. Understanding these will help in advanced Java interviews.*