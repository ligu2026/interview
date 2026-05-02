Below are **100 Python interview questions** focusing on **class constructors** (`__init__`, `__new__`, inheritance, and related topics). Each question includes a concise answer and a code example.

---

## 1. What is a constructor in Python?
**Answer:** A constructor is a special method (`__init__`) automatically called when an instance of a class is created. It initializes the instance attributes.
```python
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")
print(p.name)  # Alice
```

## 2. What is the difference between `__init__` and `__new__`?
**Answer:** `__new__` creates and returns a new instance (controls object creation). `__init__` initializes the already created instance.
```python
class Example:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        return super().__new__(cls)
    def __init__(self, value):
        print("Initializing")
        self.value = value

obj = Example(10)
```

## 3. Can a class have multiple constructors?
**Answer:** Python does not support multiple `__init__` methods directly. You can simulate it using default arguments, `@classmethod` alternative constructors, or `*args`/`**kwargs`.
```python
class Point:
    def __init__(self, x, y=0):
        self.x = x
        self.y = y

    @classmethod
    def from_tuple(cls, t):
        return cls(t[0], t[1])

p1 = Point(5)
p2 = Point(3, 4)
p3 = Point.from_tuple((1, 2))
```

## 4. How do you call a parent class constructor from a child class?
**Answer:** Use `super().__init__(...)` inside the child’s `__init__`.
```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed

d = Dog("Rex", "Lab")
print(d.name, d.breed)
```

## 5. What happens if you don’t define `__init__` in a subclass?
**Answer:** The parent class constructor is automatically invoked if the child does not define its own `__init__`.
```python
class Parent:
    def __init__(self):
        print("Parent init")

class Child(Parent):
    pass

c = Child()  # prints "Parent init"
```

## 6. Can `__init__` return a value?
**Answer:** No, `__init__` must return `None`. Returning anything else raises `TypeError`.
```python
class Test:
    def __init__(self):
        return 42   # TypeError: __init__() should return None
```

## 7. What is the purpose of `__new__`?
**Answer:** `__new__` is a static method that controls instance creation. It is called before `__init__` and is useful for implementing singletons or immutable objects.
```python
class Singleton:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

## 8. How do you create a singleton using `__new__`?
**Answer:** Override `__new__` to return the same instance each time.
```python
class Singleton:
    _instance = None
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super().__new__(cls)
        return cls._instance
```

## 9. What is constructor inheritance?
**Answer:** Child classes inherit the constructor of the parent class unless they override it.
```python
class A:
    def __init__(self):
        print("A")

class B(A):
    pass

b = B()  # prints "A"
```

## 10. How do you prevent a class from being instantiated?
**Answer:** Raise an exception in `__new__` or `__init__`, or make it an abstract class with `ABCMeta`.
```python
class NoInstance:
    def __new__(cls):
        raise TypeError("Cannot instantiate")
# or
from abc import ABC, abstractmethod
class Abstract(ABC):
    @abstractmethod
    def foo(self):
        pass
```

## 11. What is a class method as an alternative constructor?
**Answer:** A `@classmethod` that returns an instance of the class, providing different ways to create objects.
```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_str):
        y, m, d = map(int, date_str.split('-'))
        return cls(y, m, d)

d = Date.from_string("2025-05-01")
```

## 12. How do you use `*args` and `**kwargs` in a constructor?
**Answer:** They allow accepting a variable number of arguments and keyword arguments.
```python
class Flexible:
    def __init__(self, *args, **kwargs):
        self.args = args
        self.kwargs = kwargs

obj = Flexible(1, 2, a=10, b=20)
print(obj.args)    # (1,2)
print(obj.kwargs)  # {'a':10,'b':20}
```

## 13. What is the order of constructor execution in multiple inheritance?
**Answer:** Python follows the MRO (Method Resolution Order). `__init__` of parent classes is called according to MRO if `super()` is used.
```python
class A:
    def __init__(self):
        print("A")
class B(A):
    def __init__(self):
        print("B")
        super().__init__()
class C(A):
    def __init__(self):
        print("C")
        super().__init__()
class D(B, C):
    def __init__(self):
        print("D")
        super().__init__()
D()  # D B C A  (MRO: D,B,C,A)
```

## 14. How do you use `super()` without arguments in a constructor?
**Answer:** `super()` automatically refers to the next class in the MRO. No need to pass class and instance.
```python
class Parent:
    def __init__(self):
        print("Parent")
class Child(Parent):
    def __init__(self):
        super().__init__()
        print("Child")
```

## 15. Can a constructor be private? How?
**Answer:** There are no true private methods, but by convention `__init__` can be prefixed with double underscore to name-mangle it, making it harder to access.
```python
class Secret:
    def __init__(self):
        self.__secret = 42
    def __init_private(self):   # name mangled
        pass
```

## 16. What is the role of `__init_subclass__`?
**Answer:** It is called when a class is subclassed, allowing customization of subclass creation.
```python
class PluginBase:
    plugins = []
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        PluginBase.plugins.append(cls)

class A(PluginBase): pass
class B(PluginBase): pass
print(PluginBase.plugins)  # [<class 'A'>, <class 'B'>]
```

## 17. How do you create a copy constructor in Python?
**Answer:** Define `__copy__` and `__deepcopy__` methods, or just implement a method that returns a new instance with copied attributes.
```python
import copy
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __copy__(self):
        return Point(self.x, self.y)

p1 = Point(1,2)
p2 = copy.copy(p1)
```

## 18. What is the difference between `__init__` and `__call__`?
**Answer:** `__init__` initializes a new instance after creation. `__call__` makes an instance callable like a function.
```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
print(double(5))  # 10
```

## 19. Can you overload constructors using `@classmethod`?
**Answer:** Yes, by providing multiple class methods that create instances in different ways.
```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    @classmethod
    def from_polar(cls, r, theta):
        return cls(r * cos(theta), r * sin(theta))
```

## 20. What is a dataclass constructor?
**Answer:** `@dataclass` automatically generates `__init__`, `__repr__`, etc., based on type-annotated fields.
```python
from dataclasses import dataclass
@dataclass
class Person:
    name: str
    age: int

p = Person("Bob", 30)  # auto constructor
```

## 21. How do you handle default mutable arguments in constructors?
**Answer:** Avoid using mutable defaults like `[]` or `{}` directly; use `None` and create inside.
```python
class Bad:
    def __init__(self, items=[]):  # problematic
        self.items = items
class Good:
    def __init__(self, items=None):
        self.items = items if items is not None else []
```

## 22. What is `__post_init__` in dataclasses?
**Answer:** A method that runs after the auto-generated `__init__`, used for validation or derived fields.
```python
@dataclass
class Rectangle:
    w: int
    h: int
    area: float = 0.0
    def __post_init__(self):
        self.area = self.w * self.h
r = Rectangle(3,4)
print(r.area)  # 12
```

## 23. How do you make a constructor that prevents attribute creation?
**Answer:** Use `__slots__` to restrict attributes, or override `__setattr__`.
```python
class Strict:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y
s = Strict(1,2)
s.z = 3  # AttributeError
```

## 24. Can `__init__` be called multiple times on the same instance?
**Answer:** Yes, it is just a method and can be called manually, but it's not typical.
```python
class Reset:
    def __init__(self, val):
        self.val = val
obj = Reset(10)
obj.__init__(20)   # now obj.val becomes 20
print(obj.val)     # 20
```

## 25. How to enforce that a constructor is called only once?
**Answer:** Use a flag in `__new__` or in `__init__` to raise an error on second call.
```python
class Once:
    _initialized = False
    def __init__(self):
        if Once._initialized:
            raise RuntimeError("Already initialized")
        Once._initialized = True
```

## 26. What happens if `__new__` returns an object of a different class?
**Answer:** `__init__` will not be called if the returned object is not of the same type.
```python
class A:
    def __new__(cls):
        return B()
    def __init__(self):
        print("A init")
class B:
    def __init__(self):
        print("B init")
a = A()  # prints "B init", not "A init"
```

## 27. How do you pass arguments to `__new__`?
**Answer:** They are passed via `*args, **kwargs`.
```python
class Coord:
    def __new__(cls, x, y):
        print(f"Creating with {x},{y}")
        return super().__new__(cls)
    def __init__(self, x, y):
        self.x, self.y = x, y
c = Coord(3,4)
```

## 28. What is the purpose of `__del__` (destructor) in relation to constructors?
**Answer:** `__del__` is called when an object is garbage collected. It is not guaranteed to run at a specific time, unlike constructors.
```python
class FileHandler:
    def __init__(self, file):
        self.file = open(file)
    def __del__(self):
        self.file.close()
```

## 29. Can you have a constructor that returns a cached instance (like `__new__` with caching)?
**Answer:** Yes, override `__new__` to implement a cache.
```python
class CachedPoint:
    _cache = {}
    def __new__(cls, x, y):
        key = (x,y)
        if key not in cls._cache:
            cls._cache[key] = super().__new__(cls)
        return cls._cache[key]
    def __init__(self, x, y):
        self.x, self.y = x, y
```

## 30. How do you create an abstract class constructor?
**Answer:** Use `ABC` and `abstractmethod`. The abstract class can still have `__init__`, but cannot be instantiated directly.
```python
from abc import ABC, abstractmethod
class Shape(ABC):
    def __init__(self, color):
        self.color = color
    @abstractmethod
    def area(self):
        pass
# s = Shape("red")  # TypeError
```

## 31. What is a factory pattern using constructors?
**Answer:** A function or class method that creates objects using different constructors or classes.
```python
class Animal:
    pass
class Dog(Animal): pass
class Cat(Animal): pass

def animal_factory(animal_type):
    if animal_type == "dog":
        return Dog()
    elif animal_type == "cat":
        return Cat()
    else:
        return Animal()
```

## 32. How does `__new__` handle immutable objects like `tuple` or `str`?
**Answer:** Override `__new__` because `__init__` is too late (immutable objects are created at `__new__` stage).
```python
class UpperTuple(tuple):
    def __new__(cls, iterable):
        upper_items = (item.upper() for item in iterable)
        return super().__new__(cls, upper_items)
ut = UpperTuple(("a","b"))
print(ut)  # ('A','B')
```

## 33. What is the Borg pattern (shared state) and how is it implemented with `__new__`?
**Answer:** Borg makes all instances share the same `__dict__` (state).
```python
class Borg:
    _shared_state = {}
    def __new__(cls, *args, **kwargs):
        obj = super().__new__(cls, *args, **kwargs)
        obj.__dict__ = cls._shared_state
        return obj
b1 = Borg(); b2 = Borg()
b1.x = 10
print(b2.x)  # 10
```

## 34. How do you initialize a class-level attribute only once using a constructor?
**Answer:** Use a class variable and check in `__init__`.
```python
class Database:
    connection = None
    def __init__(self):
        if Database.connection is None:
            Database.connection = self._connect()
    def _connect(self):
        return "connected"
```

## 35. Can a constructor be a generator? No.
**Answer:** `__init__` cannot be a generator (cannot contain `yield`). It must return `None`.

## 36. How to ensure that a class has no constructor?
**Answer:** Simply do not define `__init__`. The default `object.__init__` does nothing.
```python
class Empty:
    pass
e = Empty()   # works
```

## 37. What is the purpose of `__reduce__` and `__reduce_ex__` for constructors in pickling?
**Answer:** They control how an object is pickled and reconstructed, often returning a callable (like a class) and arguments for the constructor.
```python
class Custom:
    def __init__(self, value):
        self.value = value
    def __reduce__(self):
        return (Custom, (self.value,))
```

## 38. How do you create a constructor that validates input?
**Answer:** Check conditions inside `__init__` and raise exceptions if invalid.
```python
class Age:
    def __init__(self, value):
        if not 0 <= value <= 150:
            raise ValueError("Invalid age")
        self.value = value
```

## 39. What is `__init__` called after `__new__` returns an instance of the same class?
**Answer:** Yes, if `__new__` returns an instance of the current class (or subclass), then `__init__` is called automatically.
```python
class A:
    def __new__(cls):
        obj = super().__new__(cls)
        print("new")
        return obj
    def __init__(self):
        print("init")
A()  # prints "new" then "init"
```

## 40. How to prevent direct instantiation but allow subclassing?
**Answer:** Use `@abstractmethod` or raise in `__new__`.
```python
from abc import ABC
class Base(ABC):
    def __new__(cls, *args, **kwargs):
        if cls is Base:
            raise TypeError("Cannot instantiate abstract class")
        return super().__new__(cls)
```

## 41. Can `__init__` be defined after the class is created?
**Answer:** Yes, you can dynamically assign `__init__` to a class.
```python
def my_init(self, name):
    self.name = name
class Dynamic:
    pass
Dynamic.__init__ = my_init
obj = Dynamic("Alice")
print(obj.name)
```

## 42. What is the `__call__` method in metaclasses for constructor interception?
**Answer:** In a metaclass, `__call__` is invoked when an instance of the class is created, allowing customization of `__new__` and `__init__`.
```python
class Meta(type):
    def __call__(cls, *args, **kwargs):
        print("Before creation")
        instance = super().__call__(*args, **kwargs)
        print("After creation")
        return instance
class MyClass(metaclass=Meta):
    def __init__(self):
        print("init")
obj = MyClass()
```

## 43. How to use `__init__` with `dataclasses.field(default_factory=...)`?
**Answer:** `default_factory` provides a callable to generate default values for mutable fields.
```python
from dataclasses import dataclass, field
@dataclass
class Bag:
    items: list = field(default_factory=list)
b = Bag()
b.items.append(1)
```

## 44. What is a “private constructor” and how to simulate it?
**Answer:** Make `__init__` raise or use a classmethod that checks a flag.
```python
class Private:
    def __init__(self):
        if not hasattr(self, '_allowed'):
            raise PermissionError
    @classmethod
    def create(cls):
        obj = cls.__new__(cls)
        obj._allowed = True
        obj.__init__()
        return obj
p = Private.create()
```

## 45. How does `NamedTuple` constructor work?
**Answer:** `NamedTuple` creates a subclass of `tuple` with a generated `__new__` and `__init__`.
```python
from typing import NamedTuple
class Point(NamedTuple):
    x: int
    y: int
p = Point(1, 2)   # constructor
```

## 46. Can you use properties in constructors?
**Answer:** Yes, you can assign to properties inside `__init__`, which may trigger setter logic.
```python
class Celsius:
    def __init__(self, temp):
        self.temp = temp   # uses setter
    @property
    def temp(self):
        return self._temp
    @temp.setter
    def temp(self, value):
        if value < -273.15:
            raise ValueError("Too low")
        self._temp = value
```

## 47. What is `__init__` overriding in mixins?
**Answer:** Mixins often define `__init__` and use `super()` to chain properly.
```python
class LogMixin:
    def __init__(self, *args, **kwargs):
        print("Log init")
        super().__init__(*args, **kwargs)
class Base:
    def __init__(self, value):
        self.value = value
class MyClass(LogMixin, Base):
    def __init__(self, value):
        super().__init__(value)
obj = MyClass(100)
```

## 48. How to make a readonly attribute set only in constructor?
**Answer:** Use a private attribute and a property without a setter.
```python
class ReadOnly:
    def __init__(self, num):
        self._num = num
    @property
    def num(self):
        return self._num
r = ReadOnly(5)
print(r.num)   # 5
# r.num = 6   AttributeError
```

## 49. What is `__new__` in the context of `metaclass` vs class?
**Answer:** `__new__` in a class creates instances. `__new__` in a metaclass creates class objects.
```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        print("Creating class")
        return super().__new__(cls, name, bases, dct)
class MyClass(metaclass=Meta):
    pass
```

## 50. Can `__init__` be a coroutine? No.
**Answer:** `__init__` cannot be `async def` because instance creation is synchronous.

## 51. How to call a parent constructor without `super()`?
**Answer:** Explicitly call `Parent.__init__(self, ...)`.
```python
class Parent:
    def __init__(self, name):
        self.name = name
class Child(Parent):
    def __init__(self, name, age):
        Parent.__init__(self, name)
        self.age = age
```

## 52. What is the purpose of `object.__new__`?
**Answer:** It is the default `__new__` that creates a new uninitialized instance. All classes ultimately inherit it.

## 53. How to create a constructor that logs every instantiation?
**Answer:** Add a print or log inside `__init__`.
```python
class Logged:
    _count = 0
    def __init__(self):
        Logged._count += 1
        print(f"Created {Logged._count} instances")
```

## 54. How to prevent inheritance of a constructor?
**Answer:** Not directly possible; you can raise an error in child `__init__` or use metaclass to block subclassing.
```python
class Final(type):
    def __new__(cls, name, bases, dct):
        if bases:
            raise TypeError("Cannot subclass Final")
        return super().__new__(cls, name, bases, dct)
class Base(metaclass=Final):
    pass
# class Child(Base): pass   # error
```

## 55. What is `__new__` used for when subclassing immutable types like `int`?
**Answer:** Override `__new__` to customize creation because `__init__` is not called for immutable subclasses.
```python
class PositiveInt(int):
    def __new__(cls, value):
        if value <= 0:
            raise ValueError("Must be positive")
        return super().__new__(cls, value)
p = PositiveInt(5)
```

## 56. How do you create a class where constructor arguments become instance attributes automatically?
**Answer:** Use `__slots__` or manually assign, or use a dataclass.
```python
class Auto:
    def __init__(self, **kwargs):
        for k, v in kwargs.items():
            setattr(self, k, v)
a = Auto(name="test", value=42)
print(a.name, a.value)
```

## 57. Can you have a static `__init__`? No, it's an instance method.
**Answer:** `__init__` receives `self` as first argument, so it's an instance method.

## 58. How to delay initialization (lazy construction)?
**Answer:** Use an `_initialized` flag or properties that compute on first access.
```python
class Lazy:
    def __init__(self):
        self._initialized = False
    def init(self):
        if not self._initialized:
            print("Initializing")
            self._initialized = True
obj = Lazy()
obj.init()
```

## 59. What is the `__new__` method signature for a metaclass?
**Answer:** `__new__(metacls, name, bases, namespace, **kwargs)`
```python
class Meta(type):
    def __new__(cls, name, bases, dct, **kwargs):
        dct['version'] = kwargs.get('version', 1)
        return super().__new__(cls, name, bases, dct)
class MyClass(metaclass=Meta, version=2):
    pass
print(MyClass.version)  # 2
```

## 60. How to create a class that uses a custom constructor without calling `__init__`?
**Answer:** Call `__new__` directly and then manually set attributes, but that's rarely needed.
```python
class Dumb:
    def __init__(self):
        self.x = 1
obj = Dumb.__new__(Dumb)   # __init__ not called
obj.x = 2
print(obj.x)   # 2
```

## 61. What is the difference between `__init__` and `__new__` regarding return values?
**Answer:** `__new__` must return an instance. `__init__` must return `None`.

## 62. Can you use `__init__` to change the class of `self`? No, it's too late.
**Answer:** The class is already determined by `__new__`. But you could reassign `self.__class__` (not recommended).
```python
class A:
    def __init__(self):
        self.__class__ = B
class B:
    pass
a = A()
print(type(a))  # <class 'B'>
```

## 63. How to create a factory of classes with different constructors?
**Answer:** Use a function that returns different class instances based on parameters.
```python
def create_animal(kind):
    if kind == "dog":
        return Dog()
    elif kind == "cat":
        return Cat()
```

## 64. What is the purpose of `__init__` in a context manager?
**Answer:** The constructor sets up the resource, and `__enter__` returns it.
```python
class ManagedFile:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file
    def __exit__(self, *args):
        self.file.close()
```

## 65. How to create a class that uses `__new__` to return an existing object (flyweight)?
**Answer:** Store objects in a pool and return from `__new__` if present.
```python
class Flyweight:
    _pool = {}
    def __new__(cls, key):
        if key not in cls._pool:
            cls._pool[key] = super().__new__(cls)
        return cls._pool[key]
    def __init__(self, key):
        self.key = key
```

## 66. What is `__init_subclass__` used for in constructor contexts?
**Answer:** It can automatically add constructor parameters or modify subclass creation.
```python
class Base:
    def __init_subclass__(cls, default_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.default_name = default_name
class Child(Base, default_name="anonymous"):
    pass
print(Child.default_name)  # anonymous
```

## 67. How to handle exceptions in `__init__` gracefully?
**Answer:** Use try/except, raise custom exceptions, or set defaults.
```python
class Safe:
    def __init__(self, value):
        try:
            self.num = int(value)
        except ValueError:
            self.num = 0
```

## 68. What is the “init-only” variable in dataclasses?
**Answer:** A field with `init=False` is not passed to `__init__`; it must have a default or be set in `__post_init__`.
```python
@dataclass
class Demo:
    a: int
    b: int = field(init=False)
    def __post_init__(self):
        self.b = self.a * 2
d = Demo(5)   # only a provided
print(d.b)    # 10
```

## 69. Can a constructor be decorated? Yes, but the decorator must return a callable.
**Answer:** You can apply a decorator to the class or to `__init__` itself.
```python
def trace_init(cls):
    original_init = cls.__init__
    def new_init(self, *args, **kwargs):
        print("Init called")
        original_init(self, *args, **kwargs)
    cls.__init__ = new_init
    return cls
@trace_init
class Test:
    def __init__(self, x):
        self.x = x
t = Test(10)
```

## 70. How to use `__new__` to implement a `__call__` that creates objects?
**Answer:** Not directly; `__new__` is for class instances, not for making a class callable.

## 71. What is the role of `__new__` in deserialization?
**Answer:** `__new__` can create an uninitialized object that `__init__` will later populate, or `__reduce__` can return a callable for reconstruction.

## 72. How to create a class that counts its instances using `__init__`?
```python
class Counted:
    instances = 0
    def __init__(self):
        Counted.instances += 1
c1 = Counted(); c2 = Counted()
print(Counted.instances)  # 2
```

## 73. How to make a constructor that enforces keyword-only arguments?
**Answer:** Use `*` in the parameter list after `self`.
```python
class KWOnly:
    def __init__(self, *, name, age):
        self.name = name
        self.age = age
# obj = KWOnly("Alice", 30)  # error
obj = KWOnly(name="Alice", age=30)
```

## 74. What is `__new__` used for in the context of `Enum`?
**Answer:** Enum's `__new__` returns the existing enum member if the value is already defined.
```python
from enum import Enum
class Color(Enum):
    RED = 1
    GREEN = 2
# Color(1) returns Color.RED, not a new instance
```

## 75. How to create a class that is not instantiable but has static methods?
**Answer:** Give it a private `__init__` that raises.
```python
class Utility:
    def __init__(self):
        raise NotImplementedError("Utility class")
    @staticmethod
    def add(a,b):
        return a+b
# u = Utility()  # error
print(Utility.add(2,3))
```

## 76. What is `__new__` priority over `__init__` in inheritance?
**Answer:** `__new__` in child overrides parent's `__new__`, but `super()` can be used to chain.

## 77. How to avoid calling `__init__` twice when using `__new__` with caching?
**Answer:** Set a flag to skip `__init__` for cached objects.
```python
class OneTime:
    _cache = {}
    def __new__(cls, key):
        if key in cls._cache:
            obj = cls._cache[key]
            obj._initialized = True  # avoid re-init
            return obj
        obj = super().__new__(cls)
        cls._cache[key] = obj
        return obj
    def __init__(self, key):
        if hasattr(self, '_initialized'):
            return
        self.key = key
        # other init
        self._initialized = True
```

## 78. Can `__init__` be called explicitly on a new instance without `__new__`?
**Answer:** You can create an instance with `__new__` and then manually call `__init__`.
```python
class Simple:
    def __init__(self, value):
        self.value = value
obj = Simple.__new__(Simple)
obj.__init__(10)
print(obj.value)  # 10
```

## 79. What is `__init__` in the context of descriptor protocol?
**Answer:** Descriptors like `property` or `classmethod` are often defined at class level, but their instances can be initialized with `__init__`.

## 80. How to create a class where `__init__` is called only once even if multiple `__new__` returns same instance?
**Answer:** Use a flag as shown above.

## 81. Can you have a `__del__` method that calls a constructor? Yes, but not recommended.
**Answer:** You could, but it may lead to circular references and GC issues.

## 82. What is the difference between `__init__` and `__new__` regarding `self`?
**Answer:** `__new__` receives `cls` and returns the instance (which becomes `self` for `__init__`).

## 83. How to make a class that accepts an arbitrary number of positional arguments in constructor?
**Answer:** Use `*args`.
```python
class VarArgs:
    def __init__(self, *args):
        self.values = args
obj = VarArgs(1,2,3,4)
```

## 84. How to make a class that accepts arbitrary keyword arguments in constructor?
**Answer:** Use `**kwargs`.
```python
class KwArgs:
    def __init__(self, **kwargs):
        for k,v in kwargs.items():
            setattr(self, k, v)
obj = KwArgs(a=10, b=20)
```

## 85. How to use `__init__` with `__slots__` to prevent `__dict__` creation?
```python
class Slotted:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

## 86. What is the default constructor provided by Python?
**Answer:** The `object` class provides a default `__init__` that does nothing and a `__new__` that creates an instance.

## 87. How to create a class with a constructor that returns a different type based on condition?
**Answer:** Override `__new__` to return any object, not necessarily of the same class.
```python
class Flexible:
    def __new__(cls, value):
        if value < 0:
            return Negative(value)
        else:
            return Positive(value)
class Negative: pass
class Positive: pass
obj = Flexible(-5)
print(type(obj))  # Negative
```

## 88. How to document a constructor?
**Answer:** Use docstring inside class or specifically for `__init__`.
```python
class Doc:
    """Class doc"""
    def __init__(self, param):
        """Constructor doc: param is ..."""
        self.param = param
```

## 89. Can `__init__` be a class method? No, it's an instance method.
**Answer:** `__init__` must take `self`.

## 90. How to implement a singleton with a metaclass using `__call__`?
```python
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
class Singleton(metaclass=SingletonMeta):
    def __init__(self, value):
        self.value = value
a = Singleton(10)
b = Singleton(20)
print(a is b, a.value)  # True 10 (b doesn't change a)
```

## 91. How to create a class that inherits from a built-in type and has a custom constructor?
```python
class MyList(list):
    def __init__(self, iterable=None):
        if iterable is None:
            iterable = []
        super().__init__(iterable)
ml = MyList([1,2,3])
```

## 92. What is the purpose of `__init__` in `ctypes` or C extensions?
**Answer:** They often define `__init__` to wrap C constructors.

## 93. How to make a constructor that forbids creation of more than N instances?
```python
class Limited:
    _count = 0
    _max = 3
    def __new__(cls, *args, **kwargs):
        if cls._count >= cls._max:
            raise RuntimeError("Max instances reached")
        cls._count += 1
        return super().__new__(cls)
```

## 94. Can `__init__` and `__new__` be called from a base class when creating a subclass instance?
**Answer:** Yes, subclass `__new__` can call `super().__new__` and subclass `__init__` calls `super().__init__`.

## 95. How to dynamically create a class with a constructor using `type()`?
```python
def init(self, name):
    self.name = name
MyClass = type('MyClass', (), {'__init__': init})
obj = MyClass("dynamic")
print(obj.name)
```

## 96. What is the `__init__` method of `object` does?
**Answer:** `object.__init__` does nothing and accepts no arguments. It is called implicitly if no other `__init__` is defined.

## 97. How to create a class that has a required argument and 10 optional arguments without writing them all?
**Answer:** Use `**kwargs` or dataclasses with defaults.
```python
from dataclasses import dataclass
@dataclass
class Many:
    required: str
    opt1: int = 0
    opt2: int = 0
    # ... but you can add more with field(default=...)
```

## 98. How to mark a constructor as deprecated?
**Answer:** Use `warnings.warn` inside `__init__`.
```python
import warnings
class Old:
    def __init__(self):
        warnings.warn("Old class is deprecated", DeprecationWarning)
```

## 99. How to create a class whose constructor automatically registers the instance in a global registry?
```python
_registry = []
class Registered:
    def __init__(self, name):
        self.name = name
        _registry.append(self)
```

## 100. Can you use `inspect.signature` on a class constructor?
**Answer:** Yes, you can get the signature of `__init__` or use `signature(Class)` which returns `__init__`'s signature.
```python
import inspect
class Demo:
    def __init__(self, a, b=10):
        pass
sig = inspect.signature(Demo)
print(sig)  # (a, b=10)
```

---

These 100 questions cover the essential and advanced topics around Python class constructors, including `__init__`, `__new__`, inheritance, metaclasses, dataclasses, and design patterns. Practice each example to master object creation in Python interviews.