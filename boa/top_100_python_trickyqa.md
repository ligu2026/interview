Below are 100 tricky Python interview questions on `map`, `list`, and `set` with detailed answers and code examples. These questions test subtle behaviors, edge cases, and advanced usage patterns.

---

### 1. List multiplication with mutable objects  
**Q:** What is the output of `[[]] * 3` and why?  
**A:** Creates a list with three references to the **same** inner list. Modifying one affects all.  
```python
lst = [[]] * 3
lst[0].append(1)
print(lst)  # [[1], [1], [1]]
```

---

### 2. List copy confusion  
**Q:** How is `a = b = [1,2,3]` different from `a = [1,2,3]; b = a`?  
**A:** Both bind `a` and `b` to the same list object. There is no difference – both create a single shared list.  
```python
a = b = [1,2,3]
a.append(4)
print(b)  # [1,2,3,4]
```

---

### 3. Removing from list while iterating  
**Q:** Why does `for i in lst: if i % 2 == 0: lst.remove(i)` not remove all evens?  
**A:** Modifying a list while iterating shifts indices, causing skipped elements.  
```python
lst = [1,2,3,4,5]
for i in lst:
    if i % 2 == 0:
        lst.remove(i)
print(lst)  # [1,3,5]? Actually [1,3,5] works here, but try [2,4,6] → problem
# Better counterexample: lst = [2,4,6,8] → after removing 2, index moves to 4 but 4 becomes 4? Actually prints [4,8]
```

---

### 4. Set from list with unhashable elements  
**Q:** What happens when you try `set([[1,2], [3,4]])`?  
**A:** Raises `TypeError` because lists are unhashable. Use tuples instead.  
```python
# set([[1,2],[3,4]])  # TypeError: unhashable type: 'list'
```

---

### 5. map with None as function  
**Q:** What does `list(map(None, [1,2,3], [4,5,6]))` do in Python 3?  
**A:** In Python 3, `None` as first argument is not allowed; raises `TypeError`. In Python 2, it returned tuples of zipped elements.  
```python
# Python 3: TypeError: 'NoneType' object is not callable
# list(map(None, [1,2]))  # TypeError
```

---

### 6. List sorting with key that modifies elements  
**Q:** The list `lst = [[1,2],[2,1],[3,0]]`. If you sort with `lst.sort(key=lambda x: x.pop())`, what happens?  
**A:** The key function mutates the sublists while sorting, leading to unpredictable results. Sorting algorithm calls key multiple times.  
```python
lst = [[1,2],[2,1],[3,0]]
lst.sort(key=lambda x: x.pop())
print(lst)  # content depends on internal sorts; generally broken
```

---

### 7. set.add vs set.update  
**Q:** What is the difference between `s.add([1,2])` and `s.update([1,2])`?  
**A:** `add` expects a single hashable element; `update` takes an iterable and adds each element. `s.add([1,2])` fails because list unhashable.  
```python
s = {1,2}
s.update([3,4])   # {1,2,3,4}
# s.add([5,6])    # TypeError
```

---

### 8. map with multiple iterables of different length  
**Q:** `list(map(lambda x,y: x+y, [1,2,3], [10,20]))` result?  
**A:** Stops at the shortest iterable -> `[11,22]`.  
```python
print(list(map(lambda x,y: x+y, [1,2,3], [10,20])))  # [11,22]
```

---

### 9. Set symmetric difference gotcha  
**Q:** `{1,2,3} ^ {2,3,4}` is `{1,4}`. What about `{1,2,3}.symmetric_difference({2,3,4})`?  
**A:** Same result, but method can accept any iterable. The `^` operator requires both operands be sets.  
```python
print({1,2,3} ^ {2,3,4})                       # {1,4}
print({1,2,3}.symmetric_difference([2,3,4]))   # {1,4}
```

---

### 10. List assignment with slice  
**Q:** `a = [1,2,3]; a[1:1] = [4,5]`. What is `a`?  
**A:** Inserts `[4,5]` at index 1, resulting in `[1,4,5,2,3]`.  
```python
a = [1,2,3]
a[1:1] = [4,5]
print(a)  # [1,4,5,2,3]
```

---

### 11. map returns a generator-like object  
**Q:** Why does `m = map(str, [1,2,3]); print(list(m)); print(list(m))` output two lists?  
**A:** `map` returns an iterator; it can be exhausted. Second call gives empty list.  
```python
m = map(str, [1,2,3])
print(list(m))  # ['1','2','3']
print(list(m))  # []
```

---

### 12. List multiplication with strings  
**Q:** `['hello'] * 3` gives `['hello','hello','hello']`. But `['hello'] * 0`?  
**A:** Returns empty list `[]`. Multiplying by zero empties the list.  
```python
print(['hello'] * 0)  # []
```

---

### 13. Set intersection with non-set  
**Q:** `{1,2,3}.intersection([2,3,4])` works? And `{1,2,3} & [2,3,4]`?  
**A:** Method works with any iterable; operator requires both sets.  
```python
print({1,2,3}.intersection([2,3,4]))  # {2,3}
# print({1,2,3} & [2,3,4])            # TypeError
```

---

### 14. List extend vs append with string  
**Q:** `lst = [1,2]; lst.append('ab'); lst.extend('ab')`. Explain.  
**A:** `append` adds the string as one element; `extend` adds each character.  
```python
lst = [1,2]
lst.append('ab')   # [1,2,'ab']
lst.extend('ab')   # [1,2,'ab','a','b']
```

---

### 15. map with lambda capturing loop variable  
**Q:** `funcs = list(map(lambda x: lambda: x, range(3))); print([f() for f in funcs])` output?  
**A:** All functions return `2` because `x` is looked up at call time (late binding).  
```python
funcs = list(map(lambda x: lambda: x, range(3)))
print([f() for f in funcs])  # [2,2,2] (in Python 3; Python 2 may be different)
```

---

### 16. Sorting list of tuples by second element  
**Q:** `sorted([(1,2),(3,1),(2,3)], key=lambda x: x[1])` result?  
**A:** `[(3,1),(1,2),(2,3)]`.  
```python
print(sorted([(1,2),(3,1),(2,3)], key=lambda x: x[1]))
```

---

### 17. Set membership with frozenset  
**Q:** Can a set contain a frozenset?  
**A:** Yes, because frozenset is hashable.  
```python
s = {frozenset([1,2]), 3}
print(s)  # {frozenset({1,2}), 3}
```

---

### 18. List copying with slice vs copy()  
**Q:** `a = [1,2,[3,4]]; b = a[:]; b[2].append(5); print(a)`?  
**A:** Shallow copy: inner list is shared, so `a` becomes `[1,2,[3,4,5]]`.  
```python
a = [1,2,[3,4]]
b = a[:]
b[2].append(5)
print(a)  # [1,2,[3,4,5]]
```

---

### 19. map with function that has side effects  
**Q:** `def f(x): print(x); return x*2; list(map(f, [1,2,3]))`. How many times is `f` called?  
**A:** Three times, because `map` applies f lazily as the list is constructed.  
```python
# Output: 1 2 3 [2,4,6]
```

---

### 20. Set difference with multiple sets  
**Q:** `{1,2,3,4}.difference({1,2},{2,3})` equals?  
**A:** `{4}`. Difference removes all elements appearing in any of the other iterables.  
```python
print({1,2,3,4}.difference({1,2},{2,3}))  # {4}
```

---

### 21. List index with start and stop  
**Q:** `[1,2,3,2,1].index(2,2,4)` returns?  
**A:** Finds `2` between index 2 (inclusive) and 4 (exclusive) → index 3.  
```python
print([1,2,3,2,1].index(2,2,4))  # 3
```

---

### 22. map with built-in function int on list of strings  
**Q:** `list(map(int, ['1','2','3']))` works. What about `list(map(int, ['1','2','a']))`?  
**A:** Raises `ValueError` on 'a'.  
```python
# list(map(int, ['1','2','a']))  # ValueError: invalid literal for int()
```

---

### 23. Set operation on list of sets  
**Q:** `from functools import reduce; reduce(set.union, [{1,2},{2,3},{3,4}])` result?  
**A:** `{1,2,3,4}`. Equivalent to union of all sets.  
```python
from functools import reduce
print(reduce(set.union, [{1,2},{2,3},{3,4}]))  # {1,2,3,4}
```

---

### 24. List removal by value vs index  
**Q:** `lst = [1,2,3,2]; lst.remove(2); print(lst)`?  
**A:** Removes the first occurrence → `[1,3,2]`.  
```python
lst = [1,2,3,2]
lst.remove(2)
print(lst)  # [1,3,2]
```

---

### 25. map on a function that returns None  
**Q:** `list(map(lambda x: print(x), [1,2,3]))` returns?  
**A:** `[None, None, None]` because `print` returns `None`.  
```python
print(list(map(lambda x: print(x), [1,2,3])))  # [None, None, None] and prints 1 2 3
```

---

### 26. List comprehension vs map performance  
**Q:** Which is faster for squaring numbers: `[x*x for x in range(1000)]` or `list(map(lambda x: x*x, range(1000)))`?  
**A:** Usually list comprehension is slightly faster because it avoids lambda overhead.  
```python
# Not shown but test with timeit.
```

---

### 27. Set cannot contain list but can contain tuple  
**Q:** `{ (1,2), [3,4] }` raises?  
**A:** `TypeError` because `[3,4]` is unhashable.  
```python
# { (1,2), [3,4] }  # TypeError
```

---

### 28. map with itertools.starmap  
**Q:** `list(itertools.starmap(pow, [(2,3), (3,2)]))` gives?  
**A:** `[8, 9]` because starmap unpacks each tuple as arguments.  
```python
import itertools
print(list(itertools.starmap(pow, [(2,3), (3,2)])))  # [8,9]
```

---

### 29. List multiplication with list comprehension  
**Q:** `[[0]*3 for _ in range(3)]` vs `[[0]*3]*3`. Difference?  
**A:** First creates independent rows; second creates references to same row.  
```python
a = [[0]*3 for _ in range(3)]
a[0][0] = 1
print(a)  # [[1,0,0],[0,0,0],[0,0,0]]

b = [[0]*3]*3
b[0][0] = 1
print(b)  # [[1,0,0],[1,0,0],[1,0,0]]
```

---

### 30. map with class method  
**Q:** `class A: def m(self,x): return x+1; objs = [A(),A()]; list(map(A.m, objs, [1,2]))`?  
**A:** `[2,3]`. `A.m` is the unbound method, takes instance as first arg.  
```python
class A:
    def m(self,x):
        return x+1
objs = [A(),A()]
print(list(map(A.m, objs, [1,2])))  # [2,3]
```

---

### 31. Set comprehensions with condition  
**Q:** `{x for x in range(5) if x%2 == 0}` results?  
**A:** `{0,2,4}`.  
```python
print({x for x in range(5) if x%2 == 0})  # {0,2,4}
```

---

### 32. List concatenation vs extend  
**Q:** `a = [1,2]; b = [3,4]; a + b` vs `a.extend(b)`. Difference?  
**A:** `+` creates a new list; `extend` modifies `a` in place.  
```python
a = [1,2]; b = [3,4]
c = a + b      # a unchanged, c = [1,2,3,4]
a.extend(b)    # a becomes [1,2,3,4]
```

---

### 33. map with None and zip alternative  
**Q:** How to achieve Python 2's `map(None, a, b)` in Python 3?  
**A:** Use `zip_longest` from itertools.  
```python
from itertools import zip_longest
print(list(zip_longest([1,2], [3,4,5])))  # [(1,3),(2,4),(None,5)]
```

---

### 34. Set discard vs remove  
**Q:** What happens if you `s.remove(10)` and `s.discard(10)` on an empty set?  
**A:** `remove` raises `KeyError`; `discard` does nothing.  
```python
s = {1,2}
# s.remove(10)   # KeyError
s.discard(10)   # no error
```

---

### 35. List assignment to slice beyond length  
**Q:** `a = [1,2,3]; a[5:5] = [10,11]; print(a)`?  
**A:** Extends the list: `[1,2,3,10,11]`. Slicing beyond length works.  
```python
a = [1,2,3]
a[5:5] = [10,11]
print(a)  # [1,2,3,10,11]
```

---

### 36. map with lambda that modifies original list  
**Q:** `lst = [[1],[2],[3]]; list(map(lambda x: x.append(0), lst)); print(lst)`?  
**A:** `lst` becomes `[[1,0],[2,0],[3,0]]`. map applies the function but the result list is `[None, None, None]`.  
```python
lst = [[1],[2],[3]]
list(map(lambda x: x.append(0), lst))
print(lst)  # [[1,0],[2,0],[3,0]]
```

---

### 37. set.union with multiple arguments  
**Q:** `{1,2}.union({3,4}, {5,6})` returns?  
**A:** `{1,2,3,4,5,6}`. Union can take any number of iterables.  
```python
print({1,2}.union({3,4}, {5,6}))  # {1,2,3,4,5,6}
```

---

### 38. Sorting list of strings case-sensitively  
**Q:** `sorted(['banana','Apple','cherry'])` orders?  
**A:** Lexicographic based on ASCII: `['Apple','banana','cherry']`. Use `key=str.lower` for case-insensitive.  
```python
print(sorted(['banana','Apple','cherry']))  # ['Apple','banana','cherry']
```

---

### 39. map with multiple iterables and function with default  
**Q:** `list(map(lambda x,y=10: x+y, [1,2,3]))`?  
**A:** Raises `TypeError` because `map` passes only one argument, but lambda expects two (y has default but still needs at least x). Actually this works? Wait: `map(lambda x,y=10: x+y, [1,2,3])` passes single argument to lambda, which is fine because y has default. But map passes only one value from the single iterable, so lambda gets x=1,y=10 etc. It works: `[11,12,13]`.  
```python
print(list(map(lambda x,y=10: x+y, [1,2,3])))  # [11,12,13]
```

---

### 40. Set from string with duplicates  
**Q:** `set('hello')` gives?  
**A:** `{'h','e','l','o'}`. Order not preserved.  
```python
print(set('hello'))  # {'h','e','l','o'}
```

---

### 41. List reverse method vs slicing  
**Q:** `a = [1,2,3]; a.reverse(); print(a); b = [1,2,3]; print(b[::-1])`. Difference?  
**A:** `reverse()` reverses in place and returns None; slicing creates a new reversed list.  
```python
a = [1,2,3]
a.reverse()
print(a)        # [3,2,1]
b = [1,2,3]
print(b[::-1])  # [3,2,1]; b unchanged
```

---

### 42. map with str.upper on mixed case  
**Q:** `list(map(str.upper, ['hello','world']))` result?  
**A:** `['HELLO','WORLD']`. Works because str.upper is a method.  
```python
print(list(map(str.upper, ['hello','world'])))  # ['HELLO','WORLD']
```

---

### 43. Set isdisjoint method  
**Q:** `{1,2,3}.isdisjoint({4,5,6})` returns?  
**A:** `True` because no common elements.  
```python
print({1,2,3}.isdisjoint({4,5,6}))  # True
```

---

### 44. List copy with `*` operator  
**Q:** `x = [[1]] * 2; x[0][0] = 2; print(x)`?  
**A:** `[[2],[2]]` – both inner lists are same object.  
```python
x = [[1]] * 2
x[0][0] = 2
print(x)  # [[2],[2]]
```

---

### 45. map with generator expression  
**Q:** `m = map(lambda x: x**2, (i for i in range(3))); print(list(m))`?  
**A:** `[0,1,4]`. map accepts any iterable.  
```python
print(list(map(lambda x: x**2, (i for i in range(3)))))  # [0,1,4]
```

---

### 46. Set pop method order  
**Q:** `s = {1,2,3}; print(s.pop()); print(s)` – which element removed?  
**A:** Arbitrary; sets are unordered. Typically the smallest or first inserted but not guaranteed.  
```python
s = {1,2,3}
print(s.pop())  # could be 1,2, or 3
```

---

### 47. List sort with reverse=True and key  
**Q:** `sorted([1,2,3,4], key=lambda x: -x, reverse=True)` result?  
**A:** Sorting by `-x` normally gives descending; `reverse=True` reverses that → ascending. Actually careful: key=-x yields [4,3,2,1]; then reverse=True -> [1,2,3,4].  
```python
print(sorted([1,2,3,4], key=lambda x: -x, reverse=True))  # [1,2,3,4]
```

---

### 48. map with lambda returning lambda  
**Q:** `list(map(lambda x: lambda: x, [1,2,3]))[0]()` output?  
**A:** `3` due to late binding closure. Same as question 15.  
```python
funcs = list(map(lambda x: lambda: x, [1,2,3]))
print(funcs[0]())  # 3
```

---

### 49. list.count vs set for frequency  
**Q:** Given a large list, which is faster to check if an element appears more than once?  
**A:** Convert to set and compare lengths; set is O(1) average per element, list.count is O(n) each call.  
```python
def has_duplicates(lst):
    return len(lst) != len(set(lst))
```

---

### 50. map with empty iterable  
**Q:** `list(map(lambda x: x*2, []))` returns?  
**A:** `[]`. Map yields no items.  
```python
print(list(map(lambda x: x*2, [])))  # []
```

---

### 51. Set update with string  
**Q:** `s = {1,2}; s.update('345'); print(s)`?  
**A:** Adds characters '3','4','5' -> `{1,2,'3','4','5'}`. Because string is iterable.  
```python
s = {1,2}
s.update('345')
print(s)  # {1,2,'3','4','5'}
```

---

### 52. List multiplication with negative integer  
**Q:** `[1,2] * -1`?  
**A:** Returns empty list `[]`.  
```python
print([1,2] * -1)  # []
```

---

### 53. map with custom class instance method  
**Q:** `class C: def m(self): return 1; c = C(); list(map(c.m, range(3)))`?  
**A:** `TypeError` because `c.m` expects no arguments but map passes each element.  
```python
class C:
    def m(self, x): return x+1   # fix signature
c = C()
print(list(map(c.m, [1,2,3])))  # [2,3,4] if method takes one arg
```

---

### 54. Set union with list of sets  
**Q:** `set().union(*[{1,2},{2,3},{3,4}])` uses `*` – what does it do?  
**A:** Unpacks the list as arguments to union, returning `{1,2,3,4}`.  
```python
print(set().union(*[{1,2},{2,3},{3,4}]))  # {1,2,3,4}
```

---

### 55. List delete by slice  
**Q:** `a = [1,2,3,4,5]; del a[1:4]; print(a)`?  
**A:** Removes indices 1-3 -> `[1,5]`.  
```python
a = [1,2,3,4,5]
del a[1:4]
print(a)  # [1,5]
```

---

### 56. map with function that takes no arguments  
**Q:** `list(map(lambda: 1, [1,2,3]))`?  
**A:** `TypeError: <lambda>() takes 0 positional arguments but 1 was given`.  
```python
# list(map(lambda: 1, [1,2,3]))  # TypeError
```

---

### 57. Set equivalence equality  
**Q:** `{1,2,3} == {3,2,1}`?  
**A:** `True`. Sets are order-independent.  
```python
print({1,2,3} == {3,2,1})  # True
```

---

### 58. List index out of range slicing  
**Q:** `[1,2,3][100:200]` returns?  
**A:** `[]` – slicing returns empty list without error.  
```python
print([1,2,3][100:200])  # []
```

---

### 59. map with multiple iterables and short-circuit  
**Q:** `list(map(lambda x,y: x/y, [1,2,3], [1,0,3]))`?  
**A:** Raises `ZeroDivisionError` when evaluating second pair (2/0). Map evaluates all pairs before building list? Actually map is lazy, but when list() consumes it, it will compute each step and stop at error.  
```python
# list(map(lambda x,y: x/y, [1,2,3], [1,0,3]))  # ZeroDivisionError
```

---

### 60. set.add with tuple vs list  
**Q:** `s = set(); s.add((1,2)); s.add([3,4])`. Which fails?  
**A:** Second fails because list unhashable.  
```python
s = set()
s.add((1,2))   # OK
# s.add([3,4]) # TypeError
```

---

### 61. List comprehension vs map for side effects  
**Q:** Which is better for printing each element: `[print(x) for x in range(3)]` or `list(map(print, range(3)))`?  
**A:** Both are discouraged; use a simple for loop. List comprehension creates a useless list of Nones.  
```python
# avoid both; for x in range(3): print(x)
```

---

### 62. Set operation `<=` vs `<`  
**Q:** `{1,2} <= {1,2,3}` is `True`. What is `{1,2,3} < {1,2,3}`?  
**A:** `False` because proper subset requires the superset to be strictly larger.  
```python
print({1,2,3} < {1,2,3})  # False
```

---

### 63. Map with itertools.cycle  
**Q:** `list(map(lambda x,y: x+y, [1,2,3], itertools.cycle([10,20])))`?  
**A:** Takes from cycle repeatedly: `[11,22,13]` because cycle repeats 10,20.  
```python
import itertools
print(list(map(lambda x,y: x+y, [1,2,3], itertools.cycle([10,20]))))  # [11,22,13]
```

---

### 64. List assignment to slice with different length  
**Q:** `a = [1,2,3]; a[1:2] = [4,5,6]; print(a)`?  
**A:** Replaces element at index 1 (2) with `[4,5,6]` -> `[1,4,5,6,3]`.  
```python
a = [1,2,3]
a[1:2] = [4,5,6]
print(a)  # [1,4,5,6,3]
```

---

### 65. set symmetric difference update  
**Q:** `s = {1,2,3}; s.symmetric_difference_update({2,3,4}); print(s)`?  
**A:** `{1,4}` – elements in either but not both.  
```python
s = {1,2,3}
s.symmetric_difference_update({2,3,4})
print(s)  # {1,4}
```

---

### 66. map with built-in `pow` and two iterables  
**Q:** `list(map(pow, [2,3,4], [5,2,1]))`?  
**A:** `[32,9,4]`.  
```python
print(list(map(pow, [2,3,4], [5,2,1])))  # [32,9,4]
```

---

### 67. List reverse iteration  
**Q:** How to iterate a list in reverse without modifying it?  
**A:** Use `reversed(lst)` or slicing `lst[::-1]`.  
```python
lst = [1,2,3]
for x in reversed(lst):
    print(x)  # 3,2,1
```

---

### 68. Set intersection update  
**Q:** `s = {1,2,3,4}; s.intersection_update({2,3,5}); print(s)`?  
**A:** `{2,3}`.  
```python
s = {1,2,3,4}
s.intersection_update({2,3,5})
print(s)  # {2,3}
```

---

### 69. List as function default argument  
**Q:** `def f(lst=[]): lst.append(1); return lst; print(f()); print(f())` output?  
**A:** `[1]` then `[1,1]` because default list is mutable and persists across calls.  
```python
def f(lst=[]):
    lst.append(1)
    return lst
print(f())  # [1]
print(f())  # [1,1]
```

---

### 70. map with lambda and multiple statements  
**Q:** `list(map(lambda x: x*2 if x>2 else x, [1,2,3,4]))`?  
**A:** `[1,2,6,8]`.  
```python
print(list(map(lambda x: x*2 if x>2 else x, [1,2,3,4])))  # [1,2,6,8]
```

---

### 71. Set from range with step  
**Q:** `set(range(1,10,2))` gives?  
**A:** `{1,3,5,7,9}`.  
```python
print(set(range(1,10,2)))  # {1,3,5,7,9}
```

---

### 72. List insert beyond length  
**Q:** `a = [1,2]; a.insert(100, 3); print(a)`?  
**A:** Inserts at end: `[1,2,3]`.  
```python
a = [1,2]
a.insert(100, 3)
print(a)  # [1,2,3]
```

---

### 73. map with str.split  
**Q:** `list(map(str.split, ['a b','c d']))` result?  
**A:** `[['a','b'], ['c','d']]`.  
```python
print(list(map(str.split, ['a b','c d'])))  # [['a','b'],['c','d']]
```

---

### 74. Set difference vs symmetric difference  
**Q:** `{1,2,3} - {2,3,4}` vs `{1,2,3} ^ {2,3,4}`.  
**A:** Difference `{1}`; symmetric difference `{1,4}`.  
```python
print({1,2,3} - {2,3,4})  # {1}
print({1,2,3} ^ {2,3,4})  # {1,4}
```

---

### 75. List unpacking with star  
**Q:** `a = [1,2,3]; b = [0, *a, 4]; print(b)`?  
**A:** `[0,1,2,3,4]`.  
```python
a = [1,2,3]
b = [0, *a, 4]
print(b)  # [0,1,2,3,4]
```

---

### 76. Set membership speed vs list  
**Q:** Why is `if x in set` faster than `if x in list` for large collections?  
**A:** Set uses hash table O(1) average; list scans O(n).  
```python
# Not shown, but best practice.
```

---

### 77. map with None and multiple iterables in Python 2 vs 3  
**Q:** Historical: `map(None, [1,2], [3,4,5])` in Python 2 gave `[(1,3),(2,4),(None,5)]`. How in Python 3?  
**A:** Use `itertools.zip_longest`.  
```python
from itertools import zip_longest
print(list(zip_longest([1,2], [3,4,5])))  # [(1,3),(2,4),(None,5)]
```

---

### 78. List sorting stability  
**Q:** If you sort a list of tuples by first element, are items with equal first element kept in original order?  
**A:** Yes, Python's sort is stable.  
```python
lst = [(1,'a'), (2,'b'), (1,'c')]
lst.sort(key=lambda x: x[0])
print(lst)  # [(1,'a'), (1,'c'), (2,'b')]  # 'a' before 'c'
```

---

### 79. map with filter  
**Q:** How to combine `map` and `filter` to square only odd numbers?  
**A:** `list(map(lambda x: x**2, filter(lambda x: x%2, range(5))))` -> `[1,9]`.  
```python
print(list(map(lambda x: x**2, filter(lambda x: x%2, range(5)))))  # [1,9]
```

---

### 80. Set of tuples with mutable inside  
**Q:** `s = {(1, [2,3])}` – valid?  
**A:** No, because tuple contains list which is unhashable.  
```python
# s = {(1, [2,3])}  # TypeError: unhashable type: 'list'
```

---

### 81. List comprehension scope  
**Q:** `x = 10; [x for x in range(5)]; print(x)` in Python 3?  
**A:** `10` – list comprehension creates a local scope; `x` outside unchanged.  
```python
x = 10
[x for x in range(5)]
print(x)  # 10 (Python 3)
```

---

### 82. map with operator module  
**Q:** `import operator; list(map(operator.add, [1,2], [3,4]))`?  
**A:** `[4,6]`.  
```python
import operator
print(list(map(operator.add, [1,2], [3,4])))  # [4,6]
```

---

### 83. Set delete during iteration  
**Q:** `s = {1,2,3}; for i in s: if i==2: s.remove(i); print(s)` – safe?  
**A:** Raises `RuntimeError: Set changed size during iteration`.  
```python
s = {1,2,3}
# for i in s:
#     if i==2: s.remove(i)  # RuntimeError
```

---

### 84. List and tuple unpacking  
**Q:** `a, *b, c = [1,2,3,4,5]; print(b)`?  
**A:** `[2,3,4]`.  
```python
a, *b, c = [1,2,3,4,5]
print(b)  # [2,3,4]
```

---

### 85. map with function that modifies global  
**Q:** `cnt=0; list(map(lambda x: globals().update(cnt=cnt+1) or x, [1,2,3])); print(cnt)`?  
**A:** `0` because `cnt` inside lambda refers to outer scope but assignment creates local unless `global` declared. Better to avoid.  
```python
cnt = 0
list(map(lambda x: globals().update({'cnt': cnt+1}) or x, [1,2,3]))
print(cnt)  # 0 (wrong) - demonstrates issue
```

---

### 86. Set union and intersection precedence  
**Q:** `{1,2} | {2,3} & {3,4}`? Operator precedence: `&` before `|` → `{1,2} | {3}` = `{1,2,3}`.  
```python
print({1,2} | {2,3} & {3,4})  # {1,2,3}
```

---

### 87. List of lists identity  
**Q:** `a = [[1]]; b = a[0]; b.append(2); print(a)`?  
**A:** `[[1,2]]` because `b` references the inner list.  
```python
a = [[1]]
b = a[0]
b.append(2)
print(a)  # [[1,2]]
```

---

### 88. map with zip  
**Q:** `list(map(lambda x: x[0]+x[1], zip([1,2],[3,4])))`?  
**A:** `[4,6]`.  
```python
print(list(map(lambda x: x[0]+x[1], zip([1,2],[3,4]))))  # [4,6]
```

---

### 89. Set with custom object hash  
**Q:** If a class defines `__eq__` but not `__hash__`, can its instances be in a set?  
**A:** No; it becomes unhashable. Must define `__hash__` or set `__hash__ = None`.  
```python
class C:
    def __eq__(self, other): return True
# set([C(), C()])  # TypeError: unhashable
```

---

### 90. List copy with slicing  
**Q:** `a = [1,2,3]; b = a[:]; a is b`?  
**A:** `False` – slice creates a shallow copy.  
```python
a = [1,2,3]
b = a[:]
print(a is b)  # False
```

---

### 91. map with lambda that captures external variable in loop  
**Q:** `funcs = []; for x in range(3): funcs.append(lambda: x); print([f() for f in funcs])` – output?  
**A:** `[2,2,2]` because `x` is looked up at call time. Use default argument to capture: `lambda x=x: x`.  
```python
funcs = []
for x in range(3):
    funcs.append(lambda: x)
print([f() for f in funcs])  # [2,2,2]
```

---

### 92. Set max on empty set  
**Q:** `max(set())`?  
**A:** Raises `ValueError: max() arg is an empty sequence`.  
```python
# max(set())  # ValueError
```

---

### 93. List remove inside loop using index  
**Q:** How to safely remove elements while iterating?  
**A:** Iterate over a copy: `for i in lst[:]: if condition: lst.remove(i)`  
```python
lst = [1,2,3,4]
for i in lst[:]:
    if i%2==0:
        lst.remove(i)
print(lst)  # [1,3]
```

---

### 94. map with built-in `abs` on complex numbers  
**Q:** `list(map(abs, [3+4j, 1+1j]))` gives?  
**A:** `[5.0, 1.4142135623730951]` – magnitude of complex numbers.  
```python
print(list(map(abs, [3+4j, 1+1j])))  # [5.0, 1.4142135623730951]
```

---

### 95. Set `issuperset` vs `>=`  
**Q:** `{1,2,3}.issuperset({1,2})` is `True`. What is `{1,2,3} >= {1,2}`?  
**A:** Also `True`. The operator works.  
```python
print({1,2,3} >= {1,2})  # True
```

---

### 96. List multiplication with reference  
**Q:** `x = [1,2]; y = [x] * 2; y[0].append(3); print(y)`?  
**A:** `[[1,2,3], [1,2,3]]`.  
```python
x = [1,2]
y = [x] * 2
y[0].append(3)
print(y)  # [[1,2,3],[1,2,3]]
```

---

### 97. map with partial function  
**Q:** Use `functools.partial` to fix first argument in map.  
**A:** `from functools import partial; add2 = partial(lambda x,y: x+y, 2); list(map(add2, [1,2,3]))` -> `[3,4,5]`.  
```python
from functools import partial
add2 = partial(lambda x,y: x+y, 2)
print(list(map(add2, [1,2,3])))  # [3,4,5]
```

---

### 98. List equality vs is  
**Q:** `[1,2] == [1,2]` vs `[1,2] is [1,2]`?  
**A:** First `True`, second `False` because two distinct list objects.  
```python
print([1,2] == [1,2])  # True
print([1,2] is [1,2])  # False
```

---

### 99. Set comprehension with nested loop  
**Q:** `{x+y for x in range(2) for y in range(2)}` result?  
**A:** `{0,1,2}`.  
```python
print({x+y for x in range(2) for y in range(2)})  # {0,1,2}
```

---

### 100. map with recursive lambda?  
**Q:** Can you create a recursive lambda inside map?  
**A:** Not directly – lambda cannot refer to itself by name. Use a function definition.  
```python
# map(lambda n: 1 if n<2 else n*fact(n-1), [1,2,3])  # NameError
def fact(n): return 1 if n<2 else n*fact(n-1)
print(list(map(fact, [1,2,3,4])))  # [1,2,6,24]
```

---

These 100 questions cover a wide range of tricky behaviors in Python’s `list`, `set`, and `map`. Use them to test deep understanding during interviews.