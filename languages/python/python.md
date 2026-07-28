# Python Concepts Cheat Sheet

A quick-reference guide covering core Python: data types, OOP, language features, project management, and advanced structures for algorithmic/DP work.

---

## 1. Data Types & Structures

### 1.1 Primitive Types

| Type | Example | Mutable? | Notes |
|------|---------|----------|-------|
| `int` | `x = 5` | No | Arbitrary precision |
| `float` | `x = 5.0` | No | Double precision |
| `complex` | `x = 2+3j` | No | Rare in general code |
| `bool` | `x = True` | No | Subclass of `int` (`True == 1`) |
| `str` | `x = "hi"` | No | Immutable sequence of chars |
| `NoneType` | `x = None` | No | Represents "no value" |

```python
x = 10
y = 3.14
name = "Claude"
flag = True
nothing = None

print(type(x))          # <class 'int'>
print(isinstance(x, int))  # True
```

### 1.2 Core Collections

| Structure | Syntax | Ordered | Mutable | Duplicates | Use Case |
|-----------|--------|---------|---------|------------|----------|
| `list` | `[1, 2, 3]` | Yes | Yes | Yes | General-purpose sequence |
| `tuple` | `(1, 2, 3)` | Yes | No | Yes | Fixed, hashable records |
| `set` | `{1, 2, 3}` | No | Yes | No | Membership, dedup |
| `frozenset` | `frozenset({1,2})` | No | No | No | Hashable set (dict key) |
| `dict` | `{"a": 1}` | Yes* | Yes | Keys unique | Key-value lookup |

\* Dicts preserve insertion order since Python 3.7.

```python
# List
lst = [1, 2, 3]
lst.append(4)          # [1, 2, 3, 4]
lst.pop()               # removes & returns last
lst[0:2]                 # slicing -> [1, 2]

# Tuple
pt = (3, 4)
x, y = pt                # unpacking

# Set
s = {1, 2, 3}
s.add(4)
s & {2, 3, 5}             # intersection -> {2, 3}

# Dict
d = {"a": 1, "b": 2}
d["c"] = 3
d.get("z", "default")     # safe lookup
for k, v in d.items():
    print(k, v)
```

### 1.3 String Essentials

```python
s = "Hello, World"

s.lower()              # "hello, world"
s.split(", ")           # ["Hello", "World"]
"-".join(["a", "b"])     # "a-b"
s[::-1]                  # reversed string
f"{s} has {len(s)} chars"  # f-strings
s.strip()                 # remove whitespace
s.replace("o", "0")       # substitution
```

### 1.4 Type Conversion

| From → To | Function |
|-----------|----------|
| str → int | `int("5")` |
| int → str | `str(5)` |
| list → set | `set([1,2,2])` |
| str → list | `list("abc")` → `['a','b','c']` |
| list → tuple | `tuple([1,2])` |

---

## 2. Object-Oriented Programming

### 2.1 Basic Class

```python
class Animal:
    species_count = 0                 # class attribute (shared)

    def __init__(self, name, sound):  # constructor
        self.name = name              # instance attribute
        self.sound = sound
        Animal.species_count += 1

    def speak(self):                  # instance method
        return f"{self.name} says {self.sound}"

    def __str__(self):                # controls print(obj)
        return f"Animal({self.name})"

    def __repr__(self):               # controls repr(obj), debugging
        return f"Animal(name={self.name!r}, sound={self.sound!r})"

a = Animal("Dog", "Woof")
print(a.speak())     # Dog says Woof
print(a)              # Animal(Dog)
```

### 2.2 Inheritance & Polymorphism

```python
class Dog(Animal):
    def __init__(self, name):
        super().__init__(name, "Woof")   # call parent constructor

    def speak(self):                      # method override
        return f"{self.name} barks!"

class Cat(Animal):
    def __init__(self, name):
        super().__init__(name, "Meow")

for pet in [Dog("Rex"), Cat("Tom")]:
    print(pet.speak())    # polymorphism: same call, different behavior
```

### 2.3 Encapsulation Conventions

| Prefix | Meaning | Enforced? |
|--------|---------|-----------|
| `name` | Public | N/A |
| `_name` | Protected (convention) | No, just a hint |
| `__name` | Private (name-mangled) | Partially, via mangling |

```python
class Account:
    def __init__(self, balance):
        self.__balance = balance      # becomes _Account__balance

    @property
    def balance(self):                # getter
        return self.__balance

    @balance.setter
    def balance(self, value):         # setter with validation
        if value < 0:
            raise ValueError("Cannot be negative")
        self.__balance = value

acc = Account(100)
acc.balance = 200      # uses setter
print(acc.balance)      # uses getter
```

### 2.4 Class Methods, Static Methods & Abstract Classes

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        ...                     # must be implemented by subclasses

class Circle(Shape):
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14159 * self.r ** 2

    @staticmethod
    def unit():
        return "cm^2"           # no access to self/cls

    @classmethod
    def unit_circle(cls):
        return cls(1)           # alternate constructor
```

### 2.5 Dunder (Magic) Methods Quick Table

| Method | Trigger |
|--------|---------|
| `__init__` | Object creation |
| `__str__` / `__repr__` | `print()` / `repr()` |
| `__len__` | `len(obj)` |
| `__eq__` | `obj1 == obj2` |
| `__lt__` | `obj1 < obj2` |
| `__add__` | `obj1 + obj2` |
| `__getitem__` | `obj[key]` |
| `__iter__` / `__next__` | `for x in obj` |
| `__enter__` / `__exit__` | `with obj:` (context manager) |

---

## 3. Python Language-Specific Features

### 3.1 Comprehensions

```python
squares = [x**2 for x in range(10)]                  # list comprehension
evens   = [x for x in range(20) if x % 2 == 0]         # with filter
pairs   = {x: x**2 for x in range(5)}                   # dict comprehension
uniq    = {x % 3 for x in range(10)}                     # set comprehension
gen     = (x**2 for x in range(10))                       # generator (lazy)
```

### 3.2 Functions: Args, Kwargs, Defaults

```python
def greet(name, greeting="Hello", *args, **kwargs):
    print(f"{greeting}, {name}!")
    print("extra positional:", args)
    print("extra keyword:", kwargs)

greet("Sam", "Hi", "extra1", key="value")
```

| Symbol | Meaning |
|--------|---------|
| `*args` | Variable positional args → tuple |
| `**kwargs` | Variable keyword args → dict |
| `def f(a, /, b)` | `a` is positional-only |
| `def f(a, *, b)` | `b` is keyword-only |

### 3.3 Lambda, Map, Filter, Reduce

```python
square = lambda x: x**2
list(map(square, [1, 2, 3]))         # [1, 4, 9]
list(filter(lambda x: x > 1, [1,2,3]))  # [2, 3]

from functools import reduce
reduce(lambda a, b: a + b, [1, 2, 3])   # 6
```

### 3.4 Decorators

```python
def timer(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.4f}s")
        return result
    return wrapper

@timer
def slow_add(a, b):
    return a + b

slow_add(2, 3)
```

### 3.5 Generators & Iterators

```python
def counter(limit):
    n = 0
    while n < limit:
        yield n          # pauses/resumes execution
        n += 1

for i in counter(3):
    print(i)              # 0 1 2
```

### 3.6 Context Managers

```python
with open("file.txt", "w") as f:
    f.write("hello")             # file auto-closed after block

class Resource:
    def __enter__(self):
        print("acquire")
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("release")

with Resource():
    print("using")
```

### 3.7 Error Handling

```python
try:
    x = 1 / 0
except ZeroDivisionError as e:
    print("Error:", e)
except (TypeError, ValueError) as e:
    print("Multiple types:", e)
else:
    print("Runs if no exception")
finally:
    print("Always runs")

# Custom exceptions
class MyError(Exception):
    pass

raise MyError("something went wrong")
```

### 3.8 Type Hints

```python
def add(a: int, b: int) -> int:
    return a + b

from typing import List, Dict, Optional, Union

def process(items: List[int], mapping: Dict[str, int],
            flag: Optional[bool] = None) -> Union[int, str]:
    ...
```

### 3.9 Unpacking & Walrus Operator

```python
a, *rest = [1, 2, 3, 4]     # a=1, rest=[2,3,4]
first, *_, last = [1,2,3,4] # first=1, last=4

if (n := len([1,2,3])) > 2:   # walrus: assign + use in one line
    print(n)
```

### 3.10 Match Statement (Python 3.10+)

```python
def handle(status):
    match status:
        case 200:
            return "OK"
        case 404 | 400:
            return "Client issue"
        case _:
            return "Unknown"
```

---

## 4. Managing a Python Project

### 4.1 Typical Project Structure

```
my_project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── module_a.py
│       └── module_b.py
├── tests/
│   └── test_module_a.py
├── requirements.txt
├── pyproject.toml
└── README.md
```

### 4.2 Importing Local Modules

```python
# module_a.py
def helper():
    return "from module_a"
```

```python
# main.py — same directory
from module_a import helper
import module_a

# from a package (folder with __init__.py)
from my_package import module_a
from my_package.module_a import helper

# relative imports (inside a package)
from .module_a import helper        # same level
from ..other_package import thing    # parent level
```

| Import Style | Example | When to Use |
|---------------|---------|--------------|
| Absolute | `from my_package.module_a import helper` | Preferred, unambiguous |
| Relative | `from .module_a import helper` | Inside a package, between siblings |
| Wildcard | `from module_a import *` | Avoid — pollutes namespace |

`__init__.py` marks a folder as a **package** and can control what's exported:

```python
# my_package/__init__.py
from .module_a import helper
__all__ = ["helper"]
```

### 4.3 Running Modules Correctly

```bash
# Run as a script from project root (recommended for packages)
python -m my_package.module_a

# The __name__ guard prevents code from running on import
if __name__ == "__main__":
    main()
```

### 4.4 Virtual Environments & Dependencies

```bash
# Create & activate a virtual environment
python -m venv .venv
source .venv/bin/activate      # macOS/Linux
.venv\Scripts\activate           # Windows

# Install & freeze dependencies
pip install requests
pip freeze > requirements.txt
pip install -r requirements.txt
```

### 4.5 `pyproject.toml` (modern packaging)

```toml
[project]
name = "my_package"
version = "0.1.0"
dependencies = ["requests>=2.0"]

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

### 4.6 Useful Standard-Library Tools for Projects

| Module | Purpose |
|--------|---------|
| `sys` | Interpreter access, `sys.path`, CLI args |
| `os` / `pathlib` | Filesystem paths and operations |
| `argparse` | Command-line argument parsing |
| `logging` | Structured logging instead of `print` |
| `unittest` / `pytest` | Testing |
| `venv` | Virtual environments |
| `importlib` | Dynamic imports |

---

## 5. Advanced Data Structures for Dynamic Programming & Algorithms

### 5.1 `collections` Module

| Structure | Import | Use Case |
|-----------|--------|----------|
| `defaultdict` | `from collections import defaultdict` | Auto-default values, avoids `KeyError` |
| `Counter` | `from collections import Counter` | Frequency counting |
| `deque` | `from collections import deque` | O(1) append/pop both ends — BFS, sliding window |
| `OrderedDict` | `from collections import OrderedDict` | Order-sensitive dict logic (mostly legacy now) |
| `namedtuple` | `from collections import namedtuple` | Lightweight immutable records |

```python
from collections import defaultdict, Counter, deque, namedtuple

# defaultdict — great for graph adjacency lists / memo tables
graph = defaultdict(list)
graph["A"].append("B")

# Counter — frequency maps
freq = Counter("aabbbcc")     # Counter({'b': 3, 'a': 2, 'c': 2})
freq.most_common(2)             # [('b', 3), ('a', 2)]

# deque — O(1) both-end ops, ideal for BFS / sliding window
q = deque([1, 2, 3])
q.appendleft(0)
q.popleft()

# namedtuple — readable tuple-based state for memoization keys
Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
p.x, p.y
```

### 5.2 Heaps (Priority Queues)

```python
import heapq

heap = [5, 1, 8, 2]
heapq.heapify(heap)              # in-place, O(n)
heapq.heappush(heap, 3)
heapq.heappop(heap)               # smallest item, O(log n)

# Max-heap trick: negate values
max_heap = [-x for x in [5, 1, 8]]
heapq.heapify(max_heap)
-heapq.heappop(max_heap)           # largest item
```

### 5.3 Memoization Patterns

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

# Manual memo dict (classic DP)
def fib_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n < 2:
        return n
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]

# Bottom-up tabulation
def fib_tab(n):
    dp = [0, 1] + [0] * (n - 1)
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

### 5.4 Sets & Dicts as Fast Lookup Tables

```python
seen = set()
for num in [1, 2, 2, 3]:
    if num in seen:            # O(1) average lookup
        print("duplicate:", num)
    seen.add(num)

# Two-sum style problems
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i
```

### 5.5 Graph Representations

```python
# Adjacency list
graph = {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["D"],
    "D": []
}

# BFS using deque
def bfs(graph, start):
    visited, queue = {start}, deque([start])
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order

# DFS using recursion
def dfs(graph, node, visited=None):
    if visited is None:
        visited = set()
    visited.add(node)
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
    return visited
```

### 5.6 Union-Find (Disjoint Set) — for DP on graphs / Kruskal's

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])   # path compression
        return self.parent[x]

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        return True
```

### 5.7 Complexity Cheat Sheet (Common Operations)

| Structure | Access | Search | Insert | Delete |
|-----------|--------|--------|--------|--------|
| `list` | O(1) | O(n) | O(1) end / O(n) mid | O(n) |
| `dict` / `set` | — | O(1) avg | O(1) avg | O(1) avg |
| `deque` | O(n) mid | O(n) | O(1) ends | O(1) ends |
| `heapq` | O(1) min | O(n) | O(log n) | O(log n) |
| Sorted `list` (via `bisect`) | O(1) | O(log n) | O(n) | O(n) |

```python
import bisect
sorted_list = [1, 3, 4, 7]
bisect.insort(sorted_list, 5)      # keeps list sorted: [1,3,4,5,7]
idx = bisect.bisect_left(sorted_list, 4)  # binary search position
```

---

## Quick Reference: When to Use What

| Need | Structure |
|------|-----------|
| Fast membership check | `set` |
| Key-value mapping | `dict` |
| Ordered, indexable, growable | `list` |
| Immutable fixed record | `tuple` / `namedtuple` |
| FIFO/BFS queue | `deque` |
| Priority-based retrieval | `heapq` |
| Frequency counting | `Counter` |
| Avoid `KeyError` on missing keys | `defaultdict` |
| Connected components / cycles | `UnionFind` |
| Overlapping subproblems | `lru_cache` / memo dict |