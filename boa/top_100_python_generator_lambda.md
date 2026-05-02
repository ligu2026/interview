Below are **100 Python interview questions** covering **lambda functions**, **generators**, and other essential topics, each with an answer and code example. The questions range from basic to advanced, with a strong focus on `lambda`, `generator`, `yield`, and related concepts.

---

## Lambda Functions

### 1. What is a lambda function in Python? Write a lambda to add two numbers.
**Answer:** A lambda function is an anonymous, single-expression function. It can take any number of arguments but returns only one value.
```python
add = lambda x, y: x + y
print(add(3, 5))  # 8
```

### 2. How do you use a lambda function with `map()`? Give an example.
**Answer:** `map()` applies a lambda to each element of an iterable.
```python
nums = [1, 2, 3, 4]
squared = list(map(lambda x: x**2, nums))
print(squared)  # [1, 4, 9, 16]
```

### 3. Can a lambda function contain multiple statements? Why?
**Answer:** No, a lambda can only contain a single expression. For multiple statements, use a regular `def` function.

### 4. Write a lambda to filter even numbers from a list using `filter()`.
```python
nums = [1, 2, 3, 4, 5, 6]
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4, 6]
```

### 5. How does `reduce()` work with a lambda? Provide an example.
**Answer:** `reduce()` cumulatively applies a lambda to items of an iterable.
```python
from functools import reduce
nums = [1, 2, 3, 4]
product = reduce(lambda x, y: x * y, nums)
print(product)  # 24
```

### 6. What is the scope of variables inside a lambda?
**Answer:** Lambda uses the same scoping rules as normal functions – it can access global and enclosing scope variables.
```python
x = 10
f = lambda y: x + y
print(f(5))  # 15
```

### 7. Can a lambda function be recursive? Provide an example.
**Answer:** Yes, but it's tricky because the lambda needs a name to call itself. Use assignment.
```python
fact = lambda n: 1 if n <= 1 else n * fact(n-1)
print(fact(5))  # 120
```

### 8. Write a lambda that sorts a list of tuples by the second element.
```python
pairs = [(1, 2), (3, 1), (5, 0)]
pairs.sort(key=lambda x: x[1])
print(pairs)  # [(5, 0), (3, 1), (1, 2)]
```

### 9. How do you use a lambda function with `sorted()` to sort by multiple keys?
```python
students = [('Alice', 25), ('Bob', 20), ('Charlie', 23)]
sorted_students = sorted(students, key=lambda x: (x[1], x[0]))
print(sorted_students)  # [('Bob',20), ('Charlie',23), ('Alice',25)]
```

### 10. What is the difference between a lambda and a regular function?
**Answer:** Lambdas are anonymous, single-expression, and cannot contain statements or annotations. Regular functions have a name, can have multiple expressions and statements, and support docstrings.

### 11. Write a lambda that returns the maximum of three numbers.
```python
max3 = lambda a,b,c: a if (a>=b and a>=c) else (b if b>=c else c)
print(max3(10, 25, 15))  # 25
```

### 12. Can you use `if-else` inside a lambda? Example.
**Answer:** Yes, using conditional expression (ternary operator).
```python
even_odd = lambda x: "even" if x % 2 == 0 else "odd"
print(even_odd(7))  # odd
```

### 13. How do you create a list of lambda functions in a loop? Watch out for late binding.
**Answer:** Late binding captures the loop variable. Use default arguments to capture current value.
```python
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])  # [0,1,2]
```

### 14. Write a lambda that strips whitespace from a string and converts to uppercase.
```python
clean = lambda s: s.strip().upper()
print(clean("  hello  "))  # "HELLO"
```

### 15. Can a lambda function return another lambda? Example.
```python
multiply_by = lambda n: lambda x: x * n
double = multiply_by(2)
print(double(5))  # 10
```

### 16. Use `map()` and lambda to convert a list of Celsius to Fahrenheit.
```python
celsius = [0, 20, 37]
fahrenheit = list(map(lambda c: (c * 9/5) + 32, celsius))
print(fahrenheit)  # [32.0, 68.0, 98.6]
```

### 17. What is the type of a lambda function?
```python
f = lambda x: x
print(type(f))  # <class 'function'>
```

### 18. Can a lambda be used as a key function in `max()` or `min()`? Example.
```python
words = ["apple", "banana", "cherry"]
longest = max(words, key=lambda w: len(w))
print(longest)  # "banana"
```

### 19. Write a lambda that checks if a string is a palindrome.
```python
is_palindrome = lambda s: s == s[::-1]
print(is_palindrome("radar"))  # True
```

### 20. How do you pickle a lambda function? Potential issue.
**Answer:** Lambdas are not picklable because they are anonymous and named by `__name__` as `<lambda>`. Use a regular function instead.

### 21. Use a lambda with `itertools.accumulate()` to compute running product.
```python
from itertools import accumulate
nums = [1,2,3,4]
running_prod = list(accumulate(nums, lambda x,y: x*y))
print(running_prod)  # [1,2,6,24]
```

### 22. How to use a lambda in `tkinter` button command?
```python
import tkinter as tk
root = tk.Tk()
btn = tk.Button(root, command=lambda: print("Clicked"))
btn.pack()
```

### 23. Write a lambda that zips two lists and sums corresponding elements.
```python
list1 = [1,2,3]; list2 = [4,5,6]
sum_pairs = list(map(lambda x,y: x+y, list1, list2))
print(sum_pairs)  # [5,7,9]
```

### 24. Can a lambda raise an exception? No direct raise, but can call a function that raises.
```python
safe_div = lambda x,y: x/y if y!=0 else None
# or
def raise_exc(): raise ValueError("zero")
lambda x,y: raise_exc() if y==0 else x/y
```

### 25. Use lambda with `functools.partial` to fix arguments.
```python
from functools import partial
pow2 = partial(lambda base, exp: base**exp, exp=2)
print(pow2(5))  # 25
```

---

## Generators and `yield`

### 26. What is a generator in Python?
**Answer:** A generator is a function that returns an iterator using `yield`. It produces values lazily, one at a time.

### 27. Write a generator that yields numbers from 0 to n.
```python
def count_up_to(n):
    i = 0
    while i <= n:
        yield i
        i += 1

for num in count_up_to(3):
    print(num)  # 0 1 2 3
```

### 28. How does `yield` differ from `return`?
**Answer:** `yield` suspends the function, remembering its state, and can resume later. `return` terminates the function and returns a value.

### 29. What is a generator expression? Provide an example.
**Answer:** A generator expression is like a list comprehension but returns a generator object, using parentheses.
```python
squares = (x*x for x in range(5))
print(next(squares))  # 0
print(list(squares))  # [1,4,9,16]
```

### 30. How do you iterate through a generator?
**Answer:** Using a `for` loop, `next()`, or converting to list/tuple.
```python
gen = (x for x in range(2))
for val in gen:
    print(val)  # 0 1
```

### 31. Write a generator that reads a large file line by line efficiently.
```python
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line

for line in read_large_file("data.txt"):
    print(line.strip())
```

### 32. What is the advantage of generators over lists?
**Answer:** Memory efficiency – they produce items on the fly and don't store the whole sequence. Also can represent infinite sequences.

### 33. Write an infinite generator for Fibonacci numbers.
```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
for _ in range(10):
    print(next(fib), end=" ")  # 0 1 1 2 3 5 8 13 21 34
```

### 34. How to send values into a generator using `send()`? Example.
```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value

acc = accumulator()
next(acc)          # prime it
print(acc.send(5)) # 5
print(acc.send(3)) # 8
```

### 35. What does `throw()` do on a generator?
**Answer:** It raises an exception at the point where the generator is paused.
```python
def gen():
    try:
        yield 1
        yield 2
    except ValueError:
        yield "caught"

g = gen()
print(next(g))      # 1
print(g.throw(ValueError))  # "caught"
```

### 36. Explain `close()` on a generator.
**Answer:** `close()` raises `GeneratorExit` inside the generator to clean up.
```python
def gen():
    try:
        yield 1
    except GeneratorExit:
        print("cleanup")
g = gen()
next(g)
g.close()   # prints "cleanup"
```

### 37. Write a generator that yields even numbers from a list.
```python
def evens(lst):
    for x in lst:
        if x % 2 == 0:
            yield x

print(list(evens([1,2,3,4])))  # [2,4]
```

### 38. Can a generator be used with `itertools.chain`? Example.
```python
import itertools
gen1 = (x for x in range(3))
gen2 = (x for x in "ab")
chained = itertools.chain(gen1, gen2)
print(list(chained))  # [0,1,2,'a','b']
```

### 39. What is `yield from` used for?
**Answer:** It delegates part of the generator’s output to another iterable or sub-generator.
```python
def sub_gen():
    yield 1
    yield 2

def main_gen():
    yield 0
    yield from sub_gen()
    yield 3

print(list(main_gen()))  # [0,1,2,3]
```

### 40. Write a generator that returns a sliding window over a sequence.
```python
def sliding_window(seq, size):
    for i in range(len(seq)-size+1):
        yield seq[i:i+size]

list(sliding_window([1,2,3,4,5], 3))  # [[1,2,3],[2,3,4],[3,4,5]]
```

### 41. How to convert a generator to a list?
```python
gen = (x for x in range(5))
lst = list(gen)
print(lst)  # [0,1,2,3,4]
```

### 42. What happens when a generator raises `StopIteration`?
**Answer:** It signals that the generator is exhausted. A `for` loop catches it automatically.

### 43. Write a generator that yields all permutations of a list (naive).
```python
def permutations(lst):
    if len(lst) <= 1:
        yield lst
    else:
        for i in range(len(lst)):
            rest = lst[:i] + lst[i+1:]
            for p in permutations(rest):
                yield [lst[i]] + p

print(list(permutations([1,2])))  # [[1,2],[2,1]]
```

### 44. What is the memory consumption difference between a generator and a list of a million numbers?
**Answer:** The list occupies ~8 MB (for ints) + overhead; the generator occupies a fixed small amount (function frame state).

### 45. Can you use `return` with a value in a generator? What does it do?
**Answer:** In Python 3, `return value` raises `StopIteration(value)`. The value can be caught.
```python
def gen():
    yield 1
    return "done"

g = gen()
print(next(g))  # 1
try:
    next(g)
except StopIteration as e:
    print(e.value)  # "done"
```

### 46. Write a generator that yields prime numbers up to n (Sieve of Eratosthenes).
```python
def primes(n):
    sieve = [True] * (n+1)
    for p in range(2, n+1):
        if sieve[p]:
            yield p
            for multiple in range(p*p, n+1, p):
                sieve[multiple] = False

print(list(primes(20)))  # [2,3,5,7,11,13,17,19]
```

### 47. How to detect if a generator is exhausted without causing an exception?
**Answer:** Use `next(generator, default)` and compare to a sentinel.
```python
gen = (x for x in range(2))
print(next(gen, None))  # 0
print(next(gen, None))  # 1
print(next(gen, None))  # None -> exhausted
```

### 48. Write a generator that yields lines from multiple files sequentially.
```python
def cat(*filepaths):
    for path in filepaths:
        with open(path) as f:
            yield from f

# for line in cat('a.txt','b.txt'): print(line)
```

### 49. What is the difference between a generator function and a normal function?
**Answer:** A generator function contains `yield`; when called it returns a generator object without executing the body. A normal function runs immediately and returns a value.

### 50. Can a generator be restarted? How to reuse it?
**Answer:** No, a generator cannot be restarted. To reuse, call the generator function again to create a new generator.

---

## Combined Lambda + Generators & Advanced Topics

### 51. Use a lambda inside a generator expression to filter odd numbers.
```python
nums = range(10)
odd_gen = (x for x in nums if (lambda n: n%2)(x))
print(list(odd_gen))  # [1,3,5,7,9]
```

### 52. Write a generator that uses a lambda to map values lazily.
```python
def lazy_map(func, iterable):
    for item in iterable:
        yield func(item)

squares = lazy_map(lambda x: x*x, [1,2,3])
print(list(squares))  # [1,4,9]
```

### 53. Combine `filter` (with lambda) and a generator – memory efficient filtering.
```python
data = range(1000000)
filtered = filter(lambda x: x % 1000 == 0, data)  # returns iterator
print(next(filtered))  # 0
print(next(filtered))  # 1000
```

### 54. Write a lambda that accepts a generator and returns the sum.
```python
sum_gen = lambda gen: sum(gen)
gen = (x for x in range(5))
print(sum_gen(gen))  # 10
```

### 55. Explain how `yield` works with lambda – can a lambda contain yield?
**Answer:** No, a lambda cannot contain `yield` because it would then be a generator function, but lambda bodies are expressions, not statements.

### 56. Use a generator to produce a sliding window and apply a lambda to each window.
```python
from itertools import islice
def window(seq, n):
    it = iter(seq)
    win = tuple(islice(it, n))
    if len(win) == n:
        yield win
    for elem in it:
        win = win[1:] + (elem,)
        yield win

data = [1,2,3,4]
means = [sum(w)/len(w) for w in window(data, 2)]
print(means)  # [1.5,2.5,3.5]
```

### 57. Write a lambda that returns a generator expression.
```python
make_squares = lambda n: (x*x for x in range(n))
gen = make_squares(5)
print(list(gen))  # [0,1,4,9,16]
```

### 58. How to implement a generator that yields values until a condition (like `takewhile`)?
```python
def takewhile(predicate, iterable):
    for x in iterable:
        if predicate(x):
            yield x
        else:
            break

evens_until_odd = takewhile(lambda x: x%2==0, [2,4,6,7,8])
print(list(evens_until_odd))  # [2,4,6]
```

### 59. Write a generator that yields pairs of elements from two generators using `zip`.
```python
def zip_generators(gen1, gen2):
    for a,b in zip(gen1, gen2):
        yield (a,b)

g1 = (x for x in range(3))
g2 = (x*10 for x in range(3))
print(list(zip_generators(g1,g2)))  # [(0,0),(1,10),(2,20)]
```

### 60. Use a lambda to define a key for `groupby` from `itertools` on a generator.
```python
from itertools import groupby
data = [1,2,3,4,5,6]
groups = groupby(data, key=lambda x: x%2)
for k, g in groups:
    print(k, list(g))  # 0 [2,4,6]; 1 [1,3,5]
```

### 61. Write a lambda that consumes a generator and returns the first n items as list.
```python
first_n = lambda gen, n: [next(gen) for _ in range(n)]
gen = iter(range(100))
print(first_n(gen, 5))  # [0,1,2,3,4]
```

### 62. Efficiently read a CSV file using a generator and parse each row with a lambda.
```python
import csv
def read_csv_rows(filepath):
    with open(filepath) as f:
        reader = csv.reader(f)
        for row in reader:
            yield row

# usage
rows = read_csv_rows("data.csv")
process = lambda row: [x.strip().upper() for x in row]
processed_rows = (process(row) for row in rows)
```

### 63. Can a generator be used with `any()` or `all()`? Example.
```python
gen = (x for x in [0,1,2])
print(any(gen))   # True (stops early, consumes 0 and 1)
print(list(gen))  # [2] (remaining)
```

### 64. Write a generator that yields the running average of a stream of numbers.
```python
def running_avg():
    total = 0
    count = 0
    while True:
        val = yield
        if val is not None:
            total += val
            count += 1
            yield total / count

avg = running_avg()
next(avg)  # prime
print(avg.send(10))  # 10.0
print(avg.send(20))  # 15.0
```

### 65. What is `itertools.tee` used for with generators?
**Answer:** It copies a generator into multiple independent iterators that can be consumed separately.
```python
from itertools import tee
gen = (x for x in range(3))
gen1, gen2 = tee(gen, 2)
print(list(gen1))  # [0,1,2]
print(list(gen2))  # [0,1,2]
```

### 66. Write a lambda that checks if a generator is empty without consuming it (using `peek` technique).
```python
def peek(gen):
    try:
        first = next(gen)
        return False, first
    except StopIteration:
        return True, None
# Then wrap gen in a new generator that yields first then rest.
```

### 67. Use a generator to implement `range`-like functionality.
```python
def my_range(start, stop=None, step=1):
    if stop is None:
        start, stop = 0, start
    while start < stop:
        yield start
        start += step

print(list(my_range(2,6,2)))  # [2,4]
```

### 68. Write a generator that yields random numbers without repetition (using set).
```python
import random
def unique_randoms(limit, count):
    generated = set()
    while len(generated) < count:
        n = random.randint(0, limit)
        if n not in generated:
            generated.add(n)
            yield n

print(list(unique_randoms(10, 5)))
```

### 69. How to handle exceptions inside a generator and still yield?
```python
def safe_divider(nums):
    for a,b in nums:
        try:
            yield a / b
        except ZeroDivisionError:
            yield None

pairs = [(10,2), (5,0), (8,4)]
results = list(safe_divider(pairs))
print(results)  # [5.0, None, 2.0]
```

### 70. Write a generator that mimics `enumerate`.
```python
def my_enumerate(iterable, start=0):
    idx = start
    for item in iterable:
        yield idx, item
        idx += 1

print(list(my_enumerate(['a','b','c'], 1)))  # [(1,'a'),(2,'b'),(3,'c')]
```

### 71. Use `functools.reduce` on a generator.
```python
from functools import reduce
gen = (x for x in range(1,5))
product = reduce(lambda x,y: x*y, gen)
print(product)  # 24
```

### 72. What is `yield` in a context manager? Provide generator-based context manager.
```python
from contextlib import contextmanager
@contextmanager
def managed_resource():
    print("acquire")
    yield "resource"
    print("release")

with managed_resource() as res:
    print(res)  # acquire resource release
```

### 73. Write a generator that yields the reverse of a list without extra memory.
```python
def reverse_gen(lst):
    for i in range(len(lst)-1, -1, -1):
        yield lst[i]

print(list(reverse_gen([1,2,3])))  # [3,2,1]
```

### 74. Can you use `await` inside a generator? (Async generators)
**Answer:** Yes, in Python 3.6+ use `async def` and `yield` to create async generator.
```python
async def async_gen():
    for i in range(3):
        await asyncio.sleep(0.1)
        yield i
```

### 75. Write a generator that yields chunks of a list (batch processing).
```python
def chunker(seq, size):
    for i in range(0, len(seq), size):
        yield seq[i:i+size]

print(list(chunker([1,2,3,4,5], 2)))  # [[1,2],[3,4],[5]]
```

### 76. What is the `__next__()` method of a generator?
**Answer:** It advances the generator and returns the next yielded value. It is called implicitly by `next()`.

### 77. Write a generator that yields characters from a string one by one.
```python
def str_gen(s):
    for ch in s:
        yield ch

g = str_gen("Hi")
print(next(g))  # H
print(next(g))  # i
```

### 78. Explain `GeneratorExit` and when it is raised.
**Answer:** It is raised when `close()` is called on a generator, allowing cleanup.

### 79. Write a lambda that takes a generator and returns a new generator that squares each value.
```python
square_gen = lambda gen: (x*x for x in gen)
orig = (1,2,3)
sq = square_gen(orig)
print(list(sq))  # [1,4,9]
```

### 80. Use `itertools.islice` on a generator to get a slice.
```python
from itertools import islice
gen = (x for x in range(100))
first_10 = list(islice(gen, 10))
print(first_10)  # [0..9]
```

### 81. Write a generator that yields all subsets of a set (power set).
```python
def power_set(s):
    items = list(s)
    for i in range(1 << len(items)):
        yield {items[j] for j in range(len(items)) if (i & (1 << j))}

print(list(power_set({1,2})))  # [set(), {1}, {2}, {1,2}]
```

### 82. What is the difference between generator and coroutine?
**Answer:** Coroutines (using `yield` as an expression) can receive values, usually through `send()` and `yield` on the right side. Generators only produce values.

### 83. Write a generator that interleaves two iterables.
```python
def interleave(a, b):
    it1, it2 = iter(a), iter(b)
    while True:
        try:
            yield next(it1)
        except StopIteration:
            yield from it2
            break
        try:
            yield next(it2)
        except StopIteration:
            yield from it1
            break

print(list(interleave([1,2], ['a','b','c'])))  # [1,'a',2,'b','c']
```

### 84. Use a generator to implement a simple round-robin scheduler.
```python
def round_robin(*tasks):
    while tasks:
        new_tasks = []
        for task in tasks:
            try:
                yield next(task)
                new_tasks.append(task)
            except StopIteration:
                pass
        tasks = new_tasks

t1 = (x for x in 'ab')
t2 = (x for x in [1,2,3])
for val in round_robin(t1, t2):
    print(val, end=' ')  # a 1 b 2 3
```

### 85. Write a generator that yields the longest consecutive subsequence from a list.
```python
def longest_consecutive(nums):
    if not nums:
        return
    nums = sorted(set(nums))
    start = nums[0]
    length = 1
    max_len = 1
    best_start = start
    for i in range(1, len(nums)):
        if nums[i] == nums[i-1] + 1:
            length += 1
            if length > max_len:
                max_len = length
                best_start = start
        else:
            start = nums[i]
            length = 1
    yield [best_start + i for i in range(max_len)]

print(list(longest_consecutive([100,4,200,1,3,2])))  # [[1,2,3,4]]
```

### 86. Can a generator be used as a context manager? Write one.
```python
from contextlib import closing
def file_gen(filename):
    f = open(filename)
    try:
        yield f
    finally:
        f.close()

with closing(file_gen("test.txt")) as g:
    for line in next(g):
        print(line)
```

### 87. Write a lambda that returns `True` if a generator produces any value matching a condition.
```python
any_match = lambda gen, pred: any(pred(x) for x in gen)
gen = (x for x in range(5))
print(any_match(gen, lambda x: x>3))  # True (consumes 0..4)
```

### 88. Create a generator of tuples from two lists with a lambda as join condition.
```python
def join_generators(left, right, key_lambda):
    for l in left:
        for r in right:
            if key_lambda(l, r):
                yield (l, r)

left = [1,2]; right = [2,3]
paired = join_generators(left, right, lambda x,y: x == y)
print(list(paired))  # [(2,2)]
```

### 89. Write a generator that yields the first n Fibonacci numbers using `yield from`.
```python
def fib(n, a=0, b=1):
    if n == 0:
        return
    yield a
    yield from fib(n-1, b, a+b)

print(list(fib(10)))  # first 10 Fib numbers
```

### 90. How to reuse a generator after it is exhausted? Provide a workaround.
**Answer:** Define a class that stores the generator function and recreate on demand.
```python
class ReusableGenerator:
    def __init__(self, generator_func, *args, **kwargs):
        self.gen_func = generator_func
        self.args = args
        self.kwargs = kwargs
    def __iter__(self):
        return self.gen_func(*self.args, **self.kwargs)

def count_to(n):
    for i in range(n):
        yield i

r = ReusableGenerator(count_to, 3)
print(list(r))  # [0,1,2]
print(list(r))  # [0,1,2] fresh each time
```

### 91. Write a lambda that accepts a generator and returns a new generator with `map` applied.
```python
map_gen = lambda func, gen: (func(x) for x in gen)
gen = (x for x in range(3))
double_gen = map_gen(lambda x: x*2, gen)
print(list(double_gen))  # [0,2,4]
```

### 92. Use `functools.partial` to fix the first argument of a generator.
```python
from functools import partial
def generate(start, step):
    while True:
        yield start
        start += step

natural = partial(generate, 1, 1)
gen = natural()
print(next(gen))  # 1
```

### 93. Write a generator that yields only the last n elements of an input generator (like `collections.deque`).
```python
from collections import deque
def tail(gen, n):
    dq = deque(maxlen=n)
    for item in gen:
        dq.append(item)
    yield from dq

gen = (x for x in range(5))
print(list(tail(gen, 3)))  # [2,3,4]
```

### 94. What is the `gi_running` attribute of a generator?
**Answer:** It indicates whether the generator is currently executing (e.g., after `send` but before next yield).

### 95. Write a generator that yields permutations of a string using recursion and `yield from`.
```python
def permute(s):
    if len(s) <= 1:
        yield s
    else:
        for i, ch in enumerate(s):
            rest = s[:i] + s[i+1:]
            for p in permute(rest):
                yield ch + p

print(list(permute("abc")))  # ['abc','acb','bac','bca','cab','cba']
```

### 96. Use `itertools.cycle` with a generator to repeat a pattern.
```python
from itertools import cycle
pattern = cycle((x for x in [1,2,3]))
first_7 = [next(pattern) for _ in range(7)]
print(first_7)  # [1,2,3,1,2,3,1]
```

### 97. Write a lambda that returns a generator of powers of 2.
```python
powers_of_2 = lambda: (1 << n for n in range(10**6))  # lazily
gen = powers_of_2()
print(next(gen))  # 1
print(next(gen))  # 2
```

### 98. Explain the purpose of `yield` in list comprehensions? (Not allowed, but in genexpr it is implicit)
**Answer:** You cannot use `yield` inside a list comprehension; use a generator expression instead.

### 99. Write a generator that flattens a nested list (depth first).
```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

print(list(flatten([1, [2, [3, 4], 5], 6])))  # [1,2,3,4,5,6]
```

### 100. Final advanced: Combine lambda, generator, and `yield from` to create a lazy query processor.
```python
def query(iterable, **filters):
    # filters: lambda expressions as strings
    for item in iterable:
        match = True
        for key, expr in filters.items():
            # expr is like "lambda x: x > 5"
            if not eval(expr)(item.get(key)):
                match = False
                break
        if match:
            yield item

data = [{'age':10},{'age':20},{'age':30}]
result = query(data, age='lambda x: x > 15')
print(list(result))  # [{'age':20},{'age':30}]
```

---

These 100 questions cover lambda functions, generators, and many advanced Python features you'll encounter in interviews. Practice each example to gain confidence!