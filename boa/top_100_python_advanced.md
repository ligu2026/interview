# Top 100 Advanced Python Interview Questions with Answers and Examples

## 1. Decorators

### Q1: Write a decorator that caches return values of a function (LRU cache) without using `functools.lru_cache`.

**A:** Use a dictionary with a queue to track usage. Here's a simple LRU cache decorator:

```python
from collections import OrderedDict

def lru_cache(maxsize=128):
    def decorator(func):
        cache = OrderedDict()
        def wrapper(*args, **kwargs):
            key = (args, tuple(kwargs.items()))
            if key in cache:
                cache.move_to_end(key)
                return cache[key]
            result = func(*args, **kwargs)
            cache[key] = result
            if len(cache) > maxsize:
                cache.popitem(last=False)
            return result
        return wrapper
    return decorator

@lru_cache(maxsize=2)
def slow_square(x):
    return x * x

print(slow_square(3))  # computed
print(slow_square(3))  # cached
```

### Q2: How can you preserve metadata of the original function when using decorators?

**A:** Use `functools.wraps` to copy `__name__`, `__doc__`, etc.

```python
from functools import wraps

def debug(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@debug
def add(a, b):
    """Add two numbers."""
    return a + b

print(add.__name__)   # 'add' (not 'wrapper')
print(add.__doc__)    # 'Add two numbers.'
```

### Q3: Implement a decorator that retries a function on exception with exponential backoff.

**A:**

```python
import time
from functools import wraps

def retry(max_retries=3, delay=1, backoff=2):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            retries, current_delay = 0, delay
            while retries < max_retries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    retries += 1
                    if retries == max_retries:
                        raise
                    time.sleep(current_delay)
                    current_delay *= backoff
            return None
        return wrapper
    return decorator

@retry(max_retries=3, delay=0.5)
def unstable_network_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Failed")
    return "Success"
```

### Q4: Create a decorator that logs method execution time and arguments.

**A:**

```python
import time
from functools import wraps

def log_execution(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__}({args}, {kwargs}) took {end-start:.6f}s")
        return result
    return wrapper

@log_execution
def compute_fib(n):
    if n < 2:
        return n
    return compute_fib(n-1) + compute_fib(n-2)

compute_fib(10)
```

### Q5: Explain decorators with arguments and nested decorators.

**A:** Decorators with arguments require an extra outer layer. Example with prefix:

```python
def prefix_logger(prefix):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"{prefix} - calling {func.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@prefix_logger("DEBUG")
def test():
    pass

# Multiple decorators execute bottom-up: @A @B means A(B(func))
```

### Q6: Write a class-based decorator that maintains state.

**A:**

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call {self.count} of {self.func.__name__}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello")

say_hello()
say_hello()  # count increments
```

### Q7: How to decorate a class (i.e., class decorator)?

**A:** Class decorators modify or enhance class creation.

```python
def add_repr(cls):
    def __repr__(self):
        return f"{cls.__name__}({self.__dict__})"
    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
print(p)  # Point({'x': 1, 'y': 2})
```

### Q8: Implement a decorator that validates argument types using type hints.

**A:**

```python
from functools import wraps
from inspect import signature

def type_check(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        sig = signature(func)
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            expected = param.annotation
            if expected is not param.empty and not isinstance(value, expected):
                raise TypeError(f"{name} must be {expected}, got {type(value)}")
        return func(*args, **kwargs)
    return wrapper

@type_check
def multiply(a: int, b: int) -> int:
    return a * b

multiply(2, 3)   # OK
# multiply(2, "3")  # TypeError
```

### Q9: Create a decorator that caches property values (cached property).

**A:**

```python
class cached_property:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, instance, owner):
        if instance is None:
            return self
        value = self.func(instance)
        instance.__dict__[self.name] = value
        return value

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @cached_property
    def area(self):
        print("Computing area...")
        return 3.14159 * self.radius ** 2

c = Circle(5)
print(c.area)  # computes
print(c.area)  # cached
```

### Q10: How to write a decorator that optionally accepts arguments (flexible)?

**A:** Detect if the decorator is called with or without parentheses.

```python
from functools import wraps

def optional_decorator(func=None, *, prefix=">>> "):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            print(prefix + f.__name__)
            return f(*args, **kwargs)
        return wrapper
    if func is None:
        return decorator
    return decorator(func)

@optional_decorator
def hello():
    print("World")

@optional_decorator(prefix="!! ")
def goodbye():
    print("Bye")

hello()   # >>> hello \n World
goodbye() # !! goodbye \n Bye
```

## 2. Generators and Iterators

### Q11: Explain the difference between `yield` and `yield from`. Provide an example.

**A:** `yield` returns a value and pauses; `yield from` delegates to a subgenerator (PEP 380).

```python
def gen_with_yield():
    for i in range(3):
        yield i

def gen_with_yield_from():
    yield from range(3)   # cleaner

# Nested delegation
def outer():
    yield "start"
    yield from inner()
    yield "end"

def inner():
    yield 1
    yield 2

print(list(outer()))  # ['start', 1, 2, 'end']
```

### Q12: Implement a generator that reads a large file line by line lazily.

**A:**

```python
def read_large_file(file_path):
    with open(file_path, 'r') as f:
        for line in f:
            yield line.rstrip('\n')

# Memory efficient: only one line in memory at a time
for line in read_large_file("big.txt"):
    process(line)
```

### Q13: Write a generator that produces an infinite Fibonacci sequence.

**A:**

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

fib = fibonacci()
first_10 = [next(fib) for _ in range(10)]
print(first_10)  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### Q14: How to send values to a generator using `send()`?

**A:** Generators can receive data; `send()` resumes and passes a value into the generator.

```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value

acc = accumulator()
next(acc)          # initialize, yields 0
print(acc.send(5)) # total=5, yields 5
print(acc.send(3)) # total=8, yields 8
acc.close()
```

### Q15: Implement a coroutine using `yield` for cooperative multitasking (producer-consumer).

**A:**

```python
def consumer():
    while True:
        item = yield
        print(f"Consumed {item}")

def producer(consumer, items):
    next(consumer)  # prime the coroutine
    for item in items:
        print(f"Produced {item}")
        consumer.send(item)

c = consumer()
producer(c, [1,2,3])
```

### Q16: What is the purpose of `itertools.tee()`? Provide example.

**A:** `tee` creates multiple independent iterators from a single iterable.

```python
from itertools import tee

original = [1,2,3,4]
it1, it2 = tee(original, 2)
print(list(it1))  # [1,2,3,4]
print(list(it2))  # [1,2,3,4] - independent copies
```

### Q17: Explain the concept of generator delegation using `yield from` with `send()` and `throw()`.

**A:** `yield from` forwards `send()`, `throw()`, and `close()` to the subgenerator.

```python
def subgen():
    while True:
        x = yield
        print(f"subgen got {x}")

def delegator():
    yield from subgen()

d = delegator()
next(d)          # prime
d.send(10)       # subgen got 10
d.throw(ValueError)  # propagates
```

### Q18: Write a generator that yields running average of a stream of numbers.

**A:**

```python
def running_average():
    total = 0
    count = 0
    while True:
        value = yield
        if value is None:
            break
        total += value
        count += 1
        yield total / count

ra = running_average()
next(ra)
print(ra.send(10))  # 10.0
print(ra.send(20))  # 15.0
print(ra.send(30))  # 20.0
```

### Q19: How to convert a generator into a list without consuming it?

**A:** Not directly possible; you can materialize it and then recreate using `tee`.

```python
from itertools import tee

def gen():
    yield from range(5)

g = gen()
g, g_copy = tee(g)
list_copy = list(g_copy)   # consume copy
print(list_copy)           # [0,1,2,3,4]
# original g is still intact (but advanced to same point)
```

### Q20: Implement a generator that yields permutations of a list without using `itertools`.

**A:**

```python
def permutations(items):
    if len(items) <= 1:
        yield items
    else:
        for i, item in enumerate(items):
            rest = items[:i] + items[i+1:]
            for perm in permutations(rest):
                yield [item] + perm

for p in permutations([1,2,3]):
    print(p)
```

## 3. Context Managers

### Q21: Write a context manager using a class to manage a file that logs errors.

**A:**

```python
class ManagedFile:
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        if exc_type is not None:
            print(f"Error: {exc_val}")
        return False  # propagate exception

with ManagedFile("test.txt", "w") as f:
    f.write("Hello")
```

### Q22: Implement a context manager using `contextlib.contextmanager` decorator.

**A:**

```python
from contextlib import contextmanager

@contextmanager
def managed_file(filename, mode='r'):
    f = open(filename, mode)
    try:
        yield f
    finally:
        f.close()

with managed_file("test.txt", "w") as f:
    f.write("Using decorator")
```

### Q23: Create a context manager that measures execution time.

**A:**

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(name="Block"):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{name} took {elapsed:.4f}s")

with timer("Sleep"):
    time.sleep(0.5)
```

### Q24: How to write an asynchronous context manager (for `async with`)?

**A:** Define `__aenter__` and `__aexit__`.

```python
import asyncio

class AsyncResource:
    async def __aenter__(self):
        print("Acquiring resource")
        await asyncio.sleep(0.1)
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        await asyncio.sleep(0.1)

async def main():
    async with AsyncResource() as res:
        print("Inside block")

asyncio.run(main())
```

### Q25: Explain `contextlib.ExitStack` and provide a use case.

**A:** `ExitStack` dynamically manages multiple context managers, useful when number unknown.

```python
from contextlib import ExitStack

files = ['a.txt', 'b.txt', 'c.txt']
with ExitStack() as stack:
    file_handles = [stack.enter_context(open(f, 'w')) for f in files]
    for fh in file_handles:
        fh.write("data")
# all files closed automatically
```

## 4. Metaclasses

### Q26: What is a metaclass? Write one that automatically adds a `created_at` attribute to all instances.

**A:** Metaclasses define how classes behave; they are classes of classes.

```python
import datetime

class AutoTimestampMeta(type):
    def __call__(cls, *args, **kwargs):
        instance = super().__call__(*args, **kwargs)
        instance.created_at = datetime.datetime.now()
        return instance

class MyClass(metaclass=AutoTimestampMeta):
    pass

obj = MyClass()
print(obj.created_at)  # current timestamp
```

### Q27: Implement a metaclass that enforces that class methods have docstrings.

**A:**

```python
class DocstringMeta(type):
    def __new__(cls, name, bases, dct):
        for attr_name, attr_value in dct.items():
            if callable(attr_value) and not attr_name.startswith('__'):
                if not attr_value.__doc__:
                    raise TypeError(f"Method {attr_name} must have a docstring")
        return super().__new__(cls, name, bases, dct)

class Good(metaclass=DocstringMeta):
    def foo(self):
        """Has docstring"""
        pass

# class Bad(metaclass=DocstringMeta):
#     def bar(self):
#         pass   # Raises TypeError
```

### Q28: Explain the difference between `__new__` and `__init__` in metaclasses.

**A:** `__new__` creates the class object; `__init__` initializes it after creation.

```python
class Meta(type):
    def __new__(meta, name, bases, dct):
        print(f"Creating class {name}")
        dct['added'] = 42
        return super().__new__(meta, name, bases, dct)

    def __init__(cls, name, bases, dct):
        print(f"Initializing class {name}")
        super().__init__(name, bases, dct)

class X(metaclass=Meta):
    pass

print(X.added)  # 42
```

### Q29: How to use a metaclass to implement a singleton pattern?

**A:**

```python
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        print("Init Database")

db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```

### Q30: Implement a metaclass that registers all subclasses automatically.

**A:**

```python
class RegistryMeta(type):
    registry = {}

    def __new__(meta, name, bases, dct):
        cls = super().__new__(meta, name, bases, dct)
        if name not in meta.registry:
            meta.registry[name] = cls
        return cls

class Base(metaclass=RegistryMeta):
    pass

class A(Base): pass
class B(Base): pass

print(RegistryMeta.registry)  # {'Base': <class ...>, 'A': ..., 'B': ...}
```

## 5. Descriptors

### Q31: Explain the descriptor protocol (`__get__`, `__set__`, `__delete__`). Implement a `PositiveNumber` descriptor.

**A:** Descriptors control attribute access.

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if value <= 0:
            raise ValueError(f"{self.name} must be positive")
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        del instance.__dict__[self.name]

class Order:
    quantity = PositiveNumber()
    price = PositiveNumber()

    def __init__(self, quantity, price):
        self.quantity = quantity
        self.price = price

o = Order(10, 5.0)
# o.quantity = -5  # ValueError
```

### Q32: What is the difference between data descriptor and non-data descriptor?

**A:** Data descriptor implements `__set__` or `__delete__`; non-data only `__get__`. Data descriptors override instance dictionary.

```python
class DataDesc:
    def __get__(self, inst, owner): print("Data get")
    def __set__(self, inst, value): print("Data set")

class NonDataDesc:
    def __get__(self, inst, owner): print("Non-data get")

class C:
    d = DataDesc()
    nd = NonDataDesc()

c = C()
c.d = 10    # Data set (descriptor overrides instance dict)
c.nd = 20   # Sets in instance dict, descriptor not called on get
```

### Q33: Implement a lazy property using a non-data descriptor.

**A:**

```python
class LazyProperty:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, instance, owner):
        if instance is None:
            return self
        value = self.func(instance)
        setattr(instance, self.name, value)  # replace descriptor
        return value

class Data:
    @LazyProperty
    def heavy(self):
        print("Computing heavy...")
        return sum(range(1000000))

d = Data()
print(d.heavy)  # computes
print(d.heavy)  # cached, no recompute
```

### Q34: Write a descriptor that validates type and value range.

**A:**

```python
class TypedProperty:
    def __init__(self, typ, min_val=None, max_val=None):
        self.typ = typ
        self.min_val = min_val
        self.max_val = max_val

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if not isinstance(value, self.typ):
            raise TypeError(f"{self.name} must be {self.typ.__name__}")
        if self.min_val is not None and value < self.min_val:
            raise ValueError(f"{self.name} must be >= {self.min_val}")
        if self.max_val is not None and value > self.max_val:
            raise ValueError(f"{self.name} must be <= {self.max_val}")
        instance.__dict__[self.name] = value

class Person:
    age = TypedProperty(int, 0, 150)

p = Person()
p.age = 30
```

### Q35: Explain how `@property` works under the hood.

**A:** `property` is a data descriptor that calls getter/setter functions.

```python
class Property:
    def __init__(self, fget=None, fset=None, fdel=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel

    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(instance)

    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel)
```

## 6. Concurrency: Threading, Multiprocessing, Asyncio

### Q36: Implement a thread-safe singleton using `threading.Lock`.

**A:**

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

### Q37: Write a producer-consumer using `queue.Queue` with multiple threads.

**A:**

```python
import threading
import queue
import time
import random

def producer(q, id):
    for i in range(5):
        item = f"Producer-{id}:{i}"
        q.put(item)
        print(f"Produced {item}")
        time.sleep(random.random())

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Consumed {item}")
        q.task_done()

q = queue.Queue()
threads = []
for i in range(2):
    t = threading.Thread(target=producer, args=(q, i))
    t.start()
    threads.append(t)
cons = threading.Thread(target=consumer, args=(q,))
cons.start()
for t in threads:
    t.join()
q.put(None)  # signal consumer to stop
cons.join()
```

### Q38: Explain the Global Interpreter Lock (GIL) and how to bypass it.

**A:** GIL allows only one thread to execute Python bytecode at a time. Bypass using multiprocessing or C extensions.

```python
# Use multiprocessing for CPU-bound tasks
from multiprocessing import Pool

def cpu_intensive(n):
    return sum(i*i for i in range(n))

with Pool(4) as p:
    results = p.map(cpu_intensive, [10**6, 10**6])
```

### Q39: What is `asyncio` and how does it differ from threading? Write an async HTTP fetcher.

**A:** Asyncio uses single-threaded event loop with coroutines for I/O-bound tasks.

```python
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = ['http://example.com', 'http://example.org']
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        print(len(results))

asyncio.run(main())
```

### Q40: Implement a custom `ThreadPool` using `queue.Queue` and threading.

**A:**

```python
import threading
import queue

class ThreadPool:
    def __init__(self, num_threads):
        self.tasks = queue.Queue()
        self.workers = []
        for _ in range(num_threads):
            t = threading.Thread(target=self._worker)
            t.daemon = True
            t.start()
            self.workers.append(t)

    def _worker(self):
        while True:
            func, args, kwargs = self.tasks.get()
            try:
                func(*args, **kwargs)
            except Exception as e:
                print(e)
            finally:
                self.tasks.task_done()

    def submit(self, func, *args, **kwargs):
        self.tasks.put((func, args, kwargs))

    def join(self):
        self.tasks.join()
```

### Q41: Write a coroutine that times out after `n` seconds using `asyncio.wait_for`.

**A:**

```python
import asyncio

async def long_running():
    await asyncio.sleep(5)
    return "Done"

async def main():
    try:
        result = await asyncio.wait_for(long_running(), timeout=2)
    except asyncio.TimeoutError:
        print("Timeout!")

asyncio.run(main())
```

### Q42: How to share data between processes safely? Use `multiprocessing.Value` and `Array`.

**A:**

```python
from multiprocessing import Process, Value, Array

def worker(val, arr):
    val.value += 1
    for i in range(len(arr)):
        arr[i] *= 2

shared_val = Value('i', 0)
shared_arr = Array('i', [1,2,3])
p = Process(target=worker, args=(shared_val, shared_arr))
p.start()
p.join()
print(shared_val.value, list(shared_arr))
```

### Q43: Implement a lock-free queue using `collections.deque` with threading (not truly lock-free but atomic append/pop?).

**A:** `deque` append/pop are thread-safe due to GIL, but usually need locks for correctness.

```python
from collections import deque
import threading

class SimpleQueue:
    def __init__(self):
        self._queue = deque()
        self._lock = threading.Lock()

    def put(self, item):
        with self._lock:
            self._queue.append(item)

    def get(self):
        with self._lock:
            return self._queue.popleft()
```

### Q44: Write an async generator that yields numbers every second.

**A:**

```python
import asyncio

async def async_countdown(n):
    for i in range(n, 0, -1):
        yield i
        await asyncio.sleep(1)

async def main():
    async for num in async_countdown(5):
        print(num)

asyncio.run(main())
```

### Q45: Explain `concurrent.futures.ThreadPoolExecutor` vs `ProcessPoolExecutor`.

**A:** ThreadPool for I/O-bound, ProcessPool for CPU-bound.

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def io_bound():
    import time; time.sleep(1)

def cpu_bound():
    return sum(i*i for i in range(10**7))

with ThreadPoolExecutor(max_workers=4) as executor:
    executor.map(io_bound, range(10))

with ProcessPoolExecutor(max_workers=4) as executor:
    results = executor.map(cpu_bound, [1,2])
```

### Q46: Implement a barrier using `threading.Barrier`.

**A:**

```python
import threading
import time

def worker(barrier, id):
    print(f"Worker {id} waiting at barrier")
    barrier.wait()
    print(f"Worker {id} passed barrier")

barrier = threading.Barrier(3)
threads = [threading.Thread(target=worker, args=(barrier, i)) for i in range(3)]
for t in threads: t.start()
for t in threads: t.join()
```

### Q47: What is `asyncio.Queue`? Example of producer-consumer with asyncio.

**A:**

```python
import asyncio

async def producer(queue):
    for i in range(5):
        await queue.put(i)
        print(f"Produced {i}")
        await asyncio.sleep(0.5)

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Consumed {item}")
        queue.task_done()
        await asyncio.sleep(0.2)

async def main():
    q = asyncio.Queue()
    prod = asyncio.create_task(producer(q))
    cons = asyncio.create_task(consumer(q))
    await prod
    await q.put(None)  # sentinel
    await cons

asyncio.run(main())
```

### Q48: Write a thread-safe counter using `threading.Lock`.

**A:**

```python
class ThreadSafeCounter:
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()

    def increment(self):
        with self._lock:
            self._value += 1

    def decrement(self):
        with self._lock:
            self._value -= 1

    @property
    def value(self):
        with self._lock:
            return self._value
```

### Q49: How to cancel a running asyncio task?

**A:**

```python
import asyncio

async def long_task():
    try:
        while True:
            await asyncio.sleep(1)
            print("Working...")
    except asyncio.CancelledError:
        print("Task cancelled")
        raise

async def main():
    task = asyncio.create_task(long_task())
    await asyncio.sleep(3)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Main: task cancelled")

asyncio.run(main())
```

### Q50: Implement a semaphore with `threading.Semaphore` to limit concurrent connections.

**A:**

```python
import threading
import time

class ConnectionPool:
    def __init__(self, limit):
        self.semaphore = threading.Semaphore(limit)

    def connect(self, name):
        with self.semaphore:
            print(f"{name} acquired connection")
            time.sleep(1)
            print(f"{name} released connection")

pool = ConnectionPool(2)
threads = [threading.Thread(target=pool.connect, args=(i,)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```

## 7. Memory Management, Garbage Collection, Weak References

### Q51: Explain Python's garbage collection and the `gc` module.

**A:** Python uses reference counting + cyclic garbage collector. `gc` allows control.

```python
import gc
import sys

class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

# Create cycle
a = Node(1); b = Node(2)
a.next = b; b.next = a
del a, b
gc.collect()  # force collection of cycles
print(gc.get_count())  # (0,0,0)
```

### Q52: What is `weakref` and when to use it?

**A:** `weakref` creates references that don't prevent garbage collection (solves cycles).

```python
import weakref

class Cache:
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()

    def set(self, key, value):
        self._cache[key] = value

    def get(self, key):
        return self._cache.get(key)

cache = Cache()
obj = object()
cache.set('key', obj)
del obj
print(cache.get('key'))  # None, object reclaimed
```

### Q53: How to detect memory leaks in Python? Use `tracemalloc`.

**A:**

```python
import tracemalloc

tracemalloc.start()
# code that might leak
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:10]:
    print(stat)
```

### Q54: Explain `__slots__` and its memory benefits.

**A:** `__slots__` prevents `__dict__` creation, reducing memory per instance.

```python
class WithoutSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class WithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
wo = WithoutSlots(1,2)
w = WithSlots(1,2)
print(sys.getsizeof(wo.__dict__))  # 64+ (dict overhead)
print(sys.getsizeof(w))            # 56 (no dict)
```

### Q55: Implement a weak reference callback to monitor object deletion.

**A:**

```python
import weakref

def callback(ref):
    print("Object was deleted")

obj = object()
ref = weakref.ref(obj, callback)
print(ref())   # <object at ...>
del obj
print(ref())   # None, and callback printed
```

## 8. Advanced Data Structures (collections module)

### Q56: Demonstrate `defaultdict`, `Counter`, `OrderedDict`, `deque` with examples.

**A:**

```python
from collections import defaultdict, Counter, OrderedDict, deque

# defaultdict
dd = defaultdict(list)
dd['key'].append(1)

# Counter
cnt = Counter("abracadabra")
print(cnt.most_common(2))  # [('a',5),('b',2)]

# OrderedDict (Python 3.7+ dict preserves order but OrderedDict has extra methods)
od = OrderedDict()
od['a'] = 1; od['b'] = 2
od.move_to_end('a')

# deque
dq = deque([1,2,3])
dq.appendleft(0)
dq.popleft()
```

### Q57: Implement a fixed-size queue using `collections.deque`.

**A:**

```python
from collections import deque

class FixedSizeQueue:
    def __init__(self, maxlen):
        self._queue = deque(maxlen=maxlen)

    def enqueue(self, item):
        self._queue.append(item)

    def dequeue(self):
        return self._queue.popleft() if self._queue else None

    def __len__(self):
        return len(self._queue)
```

### Q58: How to use `ChainMap` to merge multiple dictionaries?

**A:**

```python
from collections import ChainMap

defaults = {'theme': 'dark', 'lang': 'en'}
user = {'lang': 'fr'}
settings = ChainMap(user, defaults)
print(settings['lang'])  # 'fr' (first found)
print(settings['theme']) # 'dark'
# ChainMap updates affect first dict
settings['lang'] = 'de'
print(user)  # {'lang': 'de'}
```

### Q59: Write a `namedtuple` with default values and methods.

**A:**

```python
from collections import namedtuple

# Create base
Point = namedtuple('Point', ['x', 'y', 'z'])
# Default values using __new__ defaults
Point.__new__.__defaults__ = (0, 0, 0)
p = Point(1)   # x=1, y=0, z=0
print(p)

# With method - subclass
class PointEx(namedtuple('PointEx', ['x','y'])):
    __slots__ = ()
    def distance(self):
        return (self.x**2 + self.y**2)**0.5

p2 = PointEx(3,4)
print(p2.distance())
```

### Q60: Explain `UserDict`, `UserList`, `UserString`.

**A:** They are wrapper classes to easily subclass dict/list/str.

```python
from collections import UserDict

class CaseInsensitiveDict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key.lower(), value)

    def __getitem__(self, key):
        return super().__getitem__(key.lower())

d = CaseInsensitiveDict()
d['Foo'] = 1
print(d['FOO'])  # 1
```

## 9. Functional Programming Tools (functools, itertools)

### Q61: Implement `functools.partial` manually.

**A:** Partial freezes some arguments.

```python
def partial(func, *args, **kwargs):
    def wrapper(*more_args, **more_kwargs):
        combined_kwargs = {**kwargs, **more_kwargs}
        return func(*args, *more_args, **combined_kwargs)
    return wrapper

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
print(square(5))  # 25
```

### Q62: What is `itertools.groupby`? Provide example.

**A:** Groups consecutive items by a key function.

```python
from itertools import groupby

data = [("a", 1), ("a", 2), ("b", 3), ("b", 4)]
for key, group in groupby(data, lambda x: x[0]):
    print(key, list(group))
# a [('a',1),('a',2)]  b [('b',3),('b',4)]
```

### Q63: Implement `functools.reduce` manually.

**A:**

```python
def reduce(function, iterable, initializer=None):
    it = iter(iterable)
    if initializer is None:
        value = next(it)
    else:
        value = initializer
    for element in it:
        value = function(value, element)
    return value

print(reduce(lambda a,b: a+b, [1,2,3,4], 0))  # 10
```

### Q64: Write a generator using `itertools.cycle` to create a round-robin scheduler.

**A:**

```python
from itertools import cycle

tasks = ['A', 'B', 'C']
scheduler = cycle(tasks)
for _ in range(10):
    print(next(scheduler))
# A B C A B C ...
```

### Q65: How to use `functools.lru_cache` and its maxsize parameter?

**A:**

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

# cache info
print(fib.cache_info())  # CacheInfo(hits=..., misses=...)
fib.cache_clear()
```

### Q66: Implement `itertools.combinations` manually.

**A:**

```python
def combinations(iterable, r):
    pool = tuple(iterable)
    n = len(pool)
    if r > n:
        return
    indices = list(range(r))
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != i + n - r:
                break
        else:
            return
        indices[i] += 1
        for j in range(i+1, r):
            indices[j] = indices[j-1] + 1
        yield tuple(pool[i] for i in indices)

print(list(combinations([1,2,3], 2)))  # [(1,2),(1,3),(2,3)]
```

### Q67: What is `functools.wraps` and why is it needed?

**A:** It updates wrapper function to look like the original (preserves metadata).

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Q68: Use `itertools.tee` to implement a sliding window.

**A:**

```python
from itertools import tee, islice

def sliding_window(iterable, n):
    iterators = tee(iterable, n)
    for i, it in enumerate(iterators):
        for _ in range(i):
            next(it, None)
    return zip(*iterators)

print(list(sliding_window(range(10), 3)))  # (0,1,2)...(7,8,9)
```

## 10. Typing and Type Hints

### Q69: Explain `TypeVar`, `Generic`, and `Optional`. Write a generic stack class.

**A:**

```python
from typing import TypeVar, Generic, Optional

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> Optional[T]:
        return self._items.pop() if self._items else None

stack = Stack[int]()
stack.push(1)
print(stack.pop())
```

### Q70: How to use `@overload` for multiple type signatures?

**A:**

```python
from typing import overload, Union

@overload
def process(data: int) -> int: ...
@overload
def process(data: str) -> str: ...

def process(data: Union[int, str]) -> Union[int, str]:
    if isinstance(data, int):
        return data * 2
    else:
        return data.upper()
```

### Q71: What is `Protocol` (structural subtyping)? Provide example.

**A:** `Protocol` defines a set of methods that a class must have.

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Circle drawn")

class Square:
    def draw(self) -> None:
        print("Square drawn")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())  # valid
render(Square())  # valid
```

### Q72: Explain `Final`, `Literal`, and `TypedDict`.

**A:**

```python
from typing import Final, Literal, TypedDict

# Final constant
MAX_SIZE: Final[int] = 100

# Literal exact values
def set_mode(mode: Literal['r', 'w', 'a']) -> None:
    pass

# TypedDict for dictionary structure
class User(TypedDict):
    name: str
    age: int

user: User = {'name': 'Alice', 'age': 30}
```

## 11. Exception Handling and Custom Exceptions

### Q73: Write a custom exception hierarchy for a banking app.

**A:**

```python
class BankError(Exception):
    """Base bank exception"""

class InsufficientFundsError(BankError):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"Cannot withdraw {amount}, balance {balance}")

class AccountNotFoundError(BankError):
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount
```

### Q74: Explain `else` and `finally` in exception handling.

**A:** `else` runs if no exception; `finally` always runs.

```python
try:
    f = open('file.txt')
except FileNotFoundError:
    print("Not found")
else:
    print("File opened successfully")
    f.close()
finally:
    print("Cleanup")
```

### Q75: Implement a context manager that suppresses specific exceptions.

**A:**

```python
from contextlib import contextmanager

@contextmanager
def ignore_exception(*exceptions):
    try:
        yield
    except exceptions:
        pass

with ignore_exception(ValueError, ZeroDivisionError):
    x = int('not a number')  # ignored
```

## 12. Properties and Slots

### Q76: Implement a property with validation using setter.

**A:**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self.celsius * 9/5 + 32

t = Temperature(25)
t.celsius = 30
print(t.fahrenheit)
```

### Q77: When to use `__slots__` and what are its limitations?

**A:** Use to reduce memory when creating many instances. Limitations: no `__dict__`, no dynamic attributes, complicates inheritance.

```python
class Point:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1,2)
# p.z = 3  # AttributeError
```

## 13. Debugging and Profiling

### Q78: How to use `pdb` (Python Debugger) effectively? List commands.

**A:** Set breakpoint with `breakpoint()` (Python 3.7+). Commands: `n` (next), `s` (step), `c` (continue), `p` (print), `l` (list), `q` (quit).

```python
def buggy_func(x):
    breakpoint()   # stops here
    return x / 0

buggy_func(5)
```

### Q79: Profile a function using `cProfile` and `pstats`.

**A:**

```python
import cProfile
import pstats

def slow():
    sum(range(10_000_000))

profiler = cProfile.Profile()
profiler.enable()
slow()
profiler.disable()
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative').print_stats(10)
```

### Q80: Write a decorator that logs function arguments and return value on exception.

**A:**

```python
import functools
import traceback

def debug_on_error(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            print(f"Error in {func.__name__}")
            print(f"Args: {args}, Kwargs: {kwargs}")
            traceback.print_exc()
            raise
    return wrapper
```

## 14. Testing (unittest, pytest, mock)

### Q81: Write a unit test using `unittest.mock` to patch a function.

**A:**

```python
from unittest.mock import patch
import requests

def get_data(url):
    response = requests.get(url)
    return response.json()

# Test
def test_get_data():
    mock_response = {'key': 'value'}
    with patch('requests.get') as mock_get:
        mock_get.return_value.json.return_value = mock_response
        result = get_data('http://example.com')
        assert result == mock_response
```

### Q82: Explain `pytest` fixtures with scope.

**A:** Fixtures manage setup/teardown with scopes (function, class, module, session).

```python
import pytest

@pytest.fixture(scope="module")
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()

def test_query(db_connection):
    assert db_connection.query("SELECT 1") == 1
```

### Q83: How to test asynchronous code with `pytest-asyncio`?

**A:**

```python
import pytest
import asyncio

@pytest.mark.asyncio
async def test_async_func():
    async def fetch():
        await asyncio.sleep(0.1)
        return 42
    result = await fetch()
    assert result == 42
```

## 15. Design Patterns in Python

### Q84: Implement the Observer pattern.

**A:**

```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self, data):
        for obs in self._observers:
            obs.update(data)

class Observer:
    def update(self, data):
        print(f"Received: {data}")

subject = Subject()
obs = Observer()
subject.attach(obs)
subject.notify("Hello")
```

### Q85: Implement the Factory pattern with a registry.

**A:**

```python
class Animal:
    def speak(self): pass

class Dog(Animal):
    def speak(self): return "Woof"

class Cat(Animal):
    def speak(self): return "Meow"

class AnimalFactory:
    _registry = {}

    @classmethod
    def register(cls, name, animal_cls):
        cls._registry[name] = animal_cls

    @classmethod
    def create(cls, name):
        animal_cls = cls._registry.get(name)
        if not animal_cls:
            raise ValueError(f"Unknown animal {name}")
        return animal_cls()

AnimalFactory.register('dog', Dog)
AnimalFactory.register('cat', Cat)

animal = AnimalFactory.create('dog')
print(animal.speak())
```

### Q86: Explain the Context Manager pattern (already done). Show a custom implementation of `with` statement.

**A:** (Covered earlier) but add a `__enter__`/`__exit__` example.

### Q87: Implement the Strategy pattern using functions.

**A:**

```python
def bubble_sort(arr):
    # ... sort
    return arr

def quick_sort(arr):
    # ... sort
    return arr

class Sorter:
    def __init__(self, strategy):
        self.strategy = strategy

    def sort(self, data):
        return self.strategy(data)

sorter = Sorter(bubble_sort)
print(sorter.sort([3,1,2]))
```

## 16. Working with C Extensions / ctypes / Cython

### Q88: How to call a C function from Python using `ctypes`?

**A:** Load a shared library and call.

```python
import ctypes

libc = ctypes.CDLL("libc.so.6")  # Linux
libc.printf(b"Hello %s\n", b"World")

# Custom library
lib = ctypes.CDLL("./mylib.so")
lib.my_function.argtypes = (ctypes.c_int,)
lib.my_function.restype = ctypes.c_int
```

### Q89: What is Cython and how does it speed up code?

**A:** Cython compiles Python-like code to C, yielding C extensions.

```cython
# example.pyx
def fib(int n):
    cdef int a = 0, b = 1, i
    for i in range(n):
        a, b = b, a+b
    return a
```

## 17. Advanced File I/O and Serialization

### Q90: Compare `pickle`, `json`, and `msgpack`. When to use each?

**A:** JSON for interoperability; pickle for Python-only with objects; msgpack for binary compact.

```python
import pickle, json, msgpack

data = {'a': 1, 'b': [1,2,3]}

# Pickle
with open('data.pkl', 'wb') as f:
    pickle.dump(data, f)

# JSON
with open('data.json', 'w') as f:
    json.dump(data, f)

# msgpack
with open('data.msgpack', 'wb') as f:
    msgpack.dump(data, f)
```

### Q91: Implement a custom `Encoder`/`Decoder` for JSON to handle `datetime`.

**A:**

```python
import json
from datetime import datetime

class DateTimeEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

def datetime_decoder(dct):
    for key, value in dct.items():
        if isinstance(value, str):
            try:
                dct[key] = datetime.fromisoformat(value)
            except ValueError:
                pass
    return dct

data = {'now': datetime.now()}
json_str = json.dumps(data, cls=DateTimeEncoder)
decoded = json.loads(json_str, object_hook=datetime_decoder)
```

## 18. Context Variables, Variable Scope, Closures

### Q92: Explain `contextvars` module and its use in asyncio.

**A:** `ContextVar` provides isolated per-task/thread context, useful for tracing.

```python
from contextvars import ContextVar
import asyncio

request_id_var: ContextVar[str] = ContextVar('request_id')

async def handle():
    req_id = request_id_var.get()
    print(f"Processing {req_id}")

async def middleware(req_id):
    token = request_id_var.set(req_id)
    try:
        await handle()
    finally:
        request_id_var.reset(token)

asyncio.run(middleware("ABC123"))
```

### Q93: Write a closure that maintains state without classes.

**A:**

```python
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

c1 = make_counter()
print(c1(), c1())  # 1 2
```

## 19. AST, Bytecode, Dynamic Code Execution

### Q94: Use `ast` module to parse and modify code.

**A:** Detect all function names.

```python
import ast

code = """
def foo(): pass
def bar(): pass
"""

tree = ast.parse(code)
func_names = [node.name for node in ast.walk(tree) if isinstance(node, ast.FunctionDef)]
print(func_names)  # ['foo', 'bar']
```

### Q95: What is the difference between `eval`, `exec`, and `compile`?

**A:** `eval` evaluates an expression; `exec` executes statements; `compile` compiles to code object.

```python
x = 5
result = eval("x * 2")      # returns 10
exec("y = x + 3")           # modifies local scope
code = compile("print('hi')", '<string>', 'exec')
exec(code)                  # hi
```

## 20. Miscellaneous (GIL, Signal Handling, etc.)

### Q96: How to handle UNIX signals in Python?

**A:**

```python
import signal, time

def handler(signum, frame):
    print("SIGINT received")

signal.signal(signal.SIGINT, handler)
print("Press Ctrl+C")
time.sleep(10)
```

### Q97: Implement a simple event loop using `selectors`.

**A:**

```python
import selectors
import socket

sel = selectors.DefaultSelector()
sock = socket.socket()
sock.setblocking(False)
sock.bind(('localhost', 8080))
sock.listen()
sel.register(sock, selectors.EVENT_READ, data=None)

def accept(sock):
    conn, addr = sock.accept()
    print(f"Accepted {addr}")

while True:
    events = sel.select()
    for key, mask in events:
        accept(key.fileobj)
```

### Q98: Explain Python's `atexit` module.

**A:** Register functions to run at interpreter shutdown.

```python
import atexit

def cleanup():
    print("Cleaning up")

atexit.register(cleanup)
```

### Q99: What is `sys.setrecursionlimit` and its dangers?

**A:** Increases maximum recursion depth, can cause segmentation fault if too high.

```python
import sys
sys.setrecursionlimit(10000)

def deep(n):
    if n == 0:
        return
    deep(n-1)
deep(5000)  # works, but risk of stack overflow
```

### Q100: Write a function that retries an operation with exponential backoff and jitter.

**A:**

```python
import random, time

def retry_with_jitter(func, max_retries=5, base_delay=1, max_delay=60):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
            print(f"Attempt {attempt+1} failed: {e}. Retrying in {delay:.2f}s")
            time.sleep(delay)
    raise Exception("Max retries exceeded")

def unstable():
    if random.random() < 0.8:
        raise ValueError("Fail")
    return "Success"

print(retry_with_jitter(unstable))
```