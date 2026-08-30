# C++ STL Cheatsheet

A practical, syntax-first reference for the C++ Standard Template Library — containers, iterators, algorithms, and core utility types. Targets **C++17/20/23**.

---

## Table of Contents

- [1. STL Overview](#1-stl-overview)
- [2. Sequence Containers](#2-sequence-containers)
  - [2.1 `std::vector`](#21-stdvector)
  - [2.2 `std::array`](#22-stdarray)
  - [2.3 `std::deque`](#23-stddeque)
  - [2.4 `std::list` / `std::forward_list`](#24-stdlist--stdforward_list)
- [3. Container Adapters](#3-container-adapters)
  - [3.1 `std::stack`](#31-stdstack)
  - [3.2 `std::queue`](#32-stdqueue)
  - [3.3 `std::priority_queue`](#33-stdpriority_queue)
- [4. Associative Containers (Ordered)](#4-associative-containers-ordered)
  - [4.1 `std::set` / `std::multiset`](#41-stdset--stdmultiset)
  - [4.2 `std::map` / `std::multimap`](#42-stdmap--stdmultimap)
- [5. Unordered Associative Containers](#5-unordered-associative-containers)
  - [5.1 `std::unordered_set` / `unordered_multiset`](#51-stdunordered_set--unordered_multiset)
  - [5.2 `std::unordered_map` / `unordered_multimap`](#52-stdunordered_map--unordered_multimap)
- [6. Iterators](#6-iterators)
- [7. `<algorithm>` Library](#7-algorithm-library)
  - [7.1 Non-Modifying](#71-non-modifying-algorithms)
  - [7.2 Modifying & Sorting](#72-modifying--sorting-algorithms)
  - [7.3 Binary Search (sorted ranges)](#73-binary-search-sorted-ranges)
  - [7.4 Set Operations](#74-set-operations-sorted-ranges)
  - [7.5 Min/Max & Comparison](#75-minmax--comparison)
  - [7.6 Permutations](#76-permutations)
- [8. `<numeric>` Library](#8-numeric-library)
- [9. Strings: `std::string` & `std::string_view`](#9-strings-stdstring--stdstring_view)
- [10. Utility Types](#10-utility-types)
  - [10.1 `std::pair` / `std::tuple`](#101-stdpair--stdtuple)
  - [10.2 `std::optional`](#102-stdoptional-c17)
  - [10.3 `std::variant`](#103-stdvariant-c17)
  - [10.4 `std::any`](#104-stdany-c17)
- [11. `<functional>`](#11-functional)
- [12. `<chrono>`](#12-chrono)
- [13. Complexity Cheat Table](#13-complexity-cheat-table)
- [14. Which Container Should I Use?](#14-which-container-should-i-use)

---

## 1. STL Overview

| Component | What it does | Key headers |
|---|---|---|
| **Containers** | Store collections of objects | `<vector>`, `<map>`, `<set>`, ... |
| **Iterators** | Traverse containers generically | `<iterator>` |
| **Algorithms** | Operate on ranges via iterators | `<algorithm>`, `<numeric>` |
| **Function objects** | Callables, wrappers, adapters | `<functional>` |
| **Utilities** | Pairs, tuples, optional, variant, smart pointers | `<utility>`, `<tuple>`, `<optional>`, `<memory>` |

```cpp
#include <vector>
#include <algorithm>
#include <numeric>
using namespace std; // fine for small examples; avoid in headers/large projects
```

---

## 2. Sequence Containers

### 2.1 `std::vector`

Dynamic contiguous array. **Default go-to container.**

```cpp
#include <vector>

vector<int> v;                    // empty
vector<int> v2(5, 0);              // 5 elements, all 0
vector<int> v3 = {1, 2, 3, 4};      // list init

v.push_back(10);                  // append (amortized O(1))
v.emplace_back(20);                // construct in-place, avoids a copy
v.pop_back();                      // remove last
v.insert(v.begin() + 1, 99);        // insert at position (O(n))
v.erase(v.begin());                 // remove at position (O(n))
v[0];                                 // unchecked access
v.at(0);                              // bounds-checked access (throws)
v.front(); v.back();                 // first / last element
v.size(); v.empty(); v.clear();     // size, emptiness check, wipe all
v.reserve(100);                     // pre-allocate capacity
v.resize(10);                        // grow/shrink, fills new slots with default value
```

| Operation | Complexity |
|---|---|
| Random access `v[i]` | O(1) |
| Push/pop back | O(1) amortized |
| Insert/erase middle | O(n) |
| Search | O(n) |

### 2.2 `std::array`

Fixed-size stack-allocated array — a safer wrapper over C arrays.

```cpp
#include <array>

array<int, 5> a = {1, 2, 3, 4, 5};
a.size();          // 5, always (compile-time constant)
a.fill(0);          // set all elements to 0
a.at(2);            // bounds-checked
```

### 2.3 `std::deque`

Double-ended queue — fast insert/remove at both ends.

```cpp
#include <deque>

deque<int> dq;
dq.push_back(1);
dq.push_front(2);
dq.pop_back();
dq.pop_front();
dq[0];              // O(1) random access (slightly slower constant than vector)
```

### 2.4 `std::list` / `std::forward_list`

Doubly / singly linked list — O(1) insert/erase anywhere given an iterator, no random access.

```cpp
#include <list>

list<int> l = {1, 2, 3};
l.push_back(4);
l.push_front(0);
l.insert(next(l.begin()), 99);   // O(1) once positioned
l.remove(99);                     // remove all matching values
l.sort();                         // list has its own sort() (merge sort)
l.merge(otherList);                // merge two sorted lists

#include <forward_list>
forward_list<int> fl = {1, 2, 3}; // singly-linked, lower memory overhead
```

---

## 3. Container Adapters

### 3.1 `std::stack`

LIFO. Built atop `deque` by default.

```cpp
#include <stack>

stack<int> st;
st.push(1);
st.top();      // peek
st.pop();      // remove top (no return value!)
st.empty();
st.size();
```

### 3.2 `std::queue`

FIFO. Built atop `deque` by default.

```cpp
#include <queue>

queue<int> q;
q.push(1);
q.front(); q.back();
q.pop();
q.empty();
```

### 3.3 `std::priority_queue`

Max-heap by default.

```cpp
#include <queue>

priority_queue<int> maxHeap;
maxHeap.push(5);
maxHeap.top();     // largest element
maxHeap.pop();

// Min-heap
priority_queue<int, vector<int>, greater<int>> minHeap;

// Custom comparator
struct Compare { bool operator()(const Task& a, const Task& b) { return a.priority < b.priority; } };
priority_queue<Task, vector<Task>, Compare> taskQueue;
```

---

## 4. Associative Containers (Ordered)

Backed by a balanced binary search tree (typically red-black tree) → sorted order, O(log n) operations.

### 4.1 `std::set` / `std::multiset`

Unique (or, for multiset, non-unique) sorted keys.

```cpp
#include <set>

set<int> s = {5, 3, 1, 4};   // stored sorted: 1 3 4 5
s.insert(2);
s.erase(3);
s.find(4) != s.end();          // membership check
s.count(4);                     // 0 or 1 for set
auto it = s.lower_bound(3);    // first element >= 3
auto it2 = s.upper_bound(3);   // first element > 3

multiset<int> ms = {1, 1, 2, 3}; // duplicates allowed
```

### 4.2 `std::map` / `std::multimap`

Sorted key-value pairs.

```cpp
#include <map>

map<string, int> m;
m["apple"] = 3;               // insert or update
m.insert({"banana", 5});
m.emplace("cherry", 7);

if (m.find("apple") != m.end()) { /* found */ }
if (m.contains("apple")) { /* found, C++20 */ }

for (const auto& [key, value] : m)      // structured bindings, sorted by key
    cout << key << ": " << value << "\n";

m.erase("apple");
m.at("banana");                // throws if missing
m["nonexistent"];              // creates entry with default value if missing!
```

---

## 5. Unordered Associative Containers

Backed by a hash table → average O(1) operations, **no ordering guarantee**.

### 5.1 `std::unordered_set` / `unordered_multiset`

```cpp
#include <unordered_set>

unordered_set<int> us = {1, 2, 3};
us.insert(4);
us.find(2) != us.end();
```

### 5.2 `std::unordered_map` / `unordered_multimap`

```cpp
#include <unordered_map>

unordered_map<string, int> um;
um["x"] = 1;
um.reserve(100);        // pre-size the hash table to avoid rehashing
um.contains("x");        // C++20

// Custom key type needs a hash specialization
struct PointHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<int>()(p.first) ^ (hash<int>()(p.second) << 1);
    }
};
unordered_map<pair<int,int>, int, PointHash> grid;
```

> **Ordered vs Unordered:** use `map`/`set` when you need sorted iteration or range queries (`lower_bound`); use `unordered_map`/`unordered_set` for raw lookup speed.

---

## 6. Iterators

| Category | Capability | Example containers |
|---|---|---|
| Input | read, single-pass, forward | `istream_iterator` |
| Output | write, single-pass, forward | `ostream_iterator` |
| Forward | read/write, multi-pass, forward | `forward_list` |
| Bidirectional | + move backward | `list`, `map`, `set` |
| Random Access | + jump by index | `vector`, `deque`, `array` |
| Contiguous (C++20) | + guaranteed contiguous memory | `vector`, `array` |

```cpp
vector<int> v = {1, 2, 3, 4, 5};

auto it = v.begin();       // iterator to first element
auto e  = v.end();          // one-past-the-last element
auto rit = v.rbegin();      // reverse iterator, starts at last element

for (auto it = v.begin(); it != v.end(); ++it) cout << *it << " ";
for (int x : v) cout << x << " ";                 // range-based for (preferred)

advance(it, 2);            // move iterator forward N steps
distance(v.begin(), v.end()); // number of elements between iterators
next(it); prev(it);        // non-mutating advance/retreat (return new iterator)
```

---

## 7. `<algorithm>` Library

`#include <algorithm>` — all algorithms operate on `[first, last)` iterator ranges.

### 7.1 Non-Modifying Algorithms

```cpp
find(v.begin(), v.end(), 5);                 // iterator to first match, or end()
find_if(v.begin(), v.end(), [](int x){ return x > 3; });
count(v.begin(), v.end(), 5);                 // number of matches
count_if(v.begin(), v.end(), pred);
all_of(v.begin(), v.end(), pred);             // true if ALL match
any_of(v.begin(), v.end(), pred);             // true if ANY match
none_of(v.begin(), v.end(), pred);
for_each(v.begin(), v.end(), [](int& x){ x *= 2; });
equal(v.begin(), v.end(), v2.begin());        // element-wise equality
```

### 7.2 Modifying & Sorting Algorithms

```cpp
sort(v.begin(), v.end());                     // ascending, O(n log n)
sort(v.begin(), v.end(), greater<int>());      // descending
stable_sort(v.begin(), v.end());               // preserves relative order of equals
partial_sort(v.begin(), v.begin()+3, v.end()); // only sorts first 3 fully

reverse(v.begin(), v.end());
rotate(v.begin(), v.begin()+2, v.end());       // rotate left by 2
unique(v.begin(), v.end());                     // removes CONSECUTIVE dupes, returns new end
                                                  // (sort first if you want all dupes gone)
v.erase(unique(v.begin(), v.end()), v.end());   // idiomatic "erase-remove" for uniqueness

remove(v.begin(), v.end(), 3);                  // moves non-matching to front, returns new end
v.erase(remove(v.begin(), v.end(), 3), v.end()); // idiomatic erase-remove idiom

replace(v.begin(), v.end(), 3, 99);              // replace all 3s with 99
transform(v.begin(), v.end(), v.begin(), [](int x){ return x * x; }); // map operation
fill(v.begin(), v.end(), 0);                     // set all to a value
copy(v.begin(), v.end(), dest.begin());          // copy range
```

### 7.3 Binary Search (sorted ranges)

```cpp
binary_search(v.begin(), v.end(), 5);   // bool: is 5 present?
lower_bound(v.begin(), v.end(), 5);      // iterator to first element >= 5
upper_bound(v.begin(), v.end(), 5);      // iterator to first element > 5
equal_range(v.begin(), v.end(), 5);      // pair{lower_bound, upper_bound}
```

### 7.4 Set Operations (sorted ranges)

```cpp
vector<int> result;
set_union(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
set_intersection(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
set_difference(a.begin(), a.end(), b.begin(), b.end(), back_inserter(result));
```

### 7.5 Min/Max & Comparison

```cpp
min(3, 5); max(3, 5);
minmax(3, 5);                                    // returns pair{min, max}
min_element(v.begin(), v.end());                  // iterator to smallest
max_element(v.begin(), v.end());                  // iterator to largest
clamp(x, low, high);                               // C++17: bound x between low/high
```

### 7.6 Permutations

```cpp
vector<int> v = {1, 2, 3};
next_permutation(v.begin(), v.end());   // rearranges to next lexicographic permutation
prev_permutation(v.begin(), v.end());
```

---

## 8. `<numeric>` Library

```cpp
#include <numeric>

accumulate(v.begin(), v.end(), 0);                       // sum, starting at 0
accumulate(v.begin(), v.end(), 1, multiplies<int>());     // product
inner_product(a.begin(), a.end(), b.begin(), 0);          // dot product
iota(v.begin(), v.end(), 1);                                // fills v with 1, 2, 3, ...
partial_sum(v.begin(), v.end(), result.begin());           // running/prefix sums
adjacent_difference(v.begin(), v.end(), result.begin());   // differences between adjacent elements
gcd(12, 18); lcm(4, 6);                                     // C++17
```

---

## 9. Strings: `std::string` & `std::string_view`

```cpp
#include <string>

string s = "Hello, World!";
s.length(); s.size();
s.substr(7, 5);                 // "World" (start, length)
s.find("World");                 // index or string::npos
s + " Bye";                       // concatenation
s.append("!");
s.replace(0, 5, "Goodbye");
s.erase(0, 7);
to_string(42);                    // int -> string
stoi("42"); stod("3.14");        // string -> number

#include <string_view>
string_view sv = "non-owning view of text"; // avoids copies, great for parsing/read-only params
void process(string_view text);              // preferred parameter type over `const string&`
```

---

## 10. Utility Types

### 10.1 `std::pair` / `std::tuple`

```cpp
#include <utility>
pair<int, string> p = {1, "one"};
p.first; p.second;

#include <tuple>
tuple<int, string, double> t = {1, "one", 1.0};
get<0>(t);                          // access by index
auto [a, b, c] = t;                  // structured bindings (C++17)
tie(a, b, c) = t;                     // unpack into existing variables
```

### 10.2 `std::optional` (C++17)

Represents a value that may or may not be present — avoids sentinel values like `-1` or `nullptr`.

```cpp
#include <optional>

optional<int> find(int key) {
    if (key == 1) return 42;
    return nullopt;
}
if (auto result = find(1)) cout << *result;
result.value_or(-1);   // default if empty
```

### 10.3 `std::variant` (C++17)

Type-safe union — holds one of several alternative types.

```cpp
#include <variant>

variant<int, string> v = "hello";
if (holds_alternative<string>(v)) cout << get<string>(v);
visit([](auto&& val) { cout << val; }, v); // dispatch by active type
```

### 10.4 `std::any` (C++17)

Type-erased container for a single value of any type.

```cpp
#include <any>

any a = 5;
a = string("now a string");
any_cast<string>(a);   // throws bad_any_cast if wrong type
```

---

## 11. `<functional>`

```cpp
#include <functional>

function<int(int,int)> op = [](int a, int b) { return a + b; }; // type-erased callable wrapper
function<bool(int)> pred = greater_equal<int>() /* ... */;

// Common function objects
plus<int>(); minus<int>(); multiplies<int>(); divides<int>();
greater<int>(); less<int>(); equal_to<int>();

auto bound = bind(op, 5, placeholders::_1);  // partial application (prefer lambdas in modern code)
```

---

## 12. `<chrono>`

```cpp
#include <chrono>
using namespace std::chrono;

auto start = high_resolution_clock::now();
// ... work ...
auto end = high_resolution_clock::now();
auto elapsed = duration_cast<milliseconds>(end - start);
cout << elapsed.count() << "ms\n";

// C++20 calendar/timezone support
auto today = year_month_day{floor<days>(system_clock::now())};
```

---

## 13. Complexity Cheat Table

| Container | Access | Search | Insert | Erase |
|---|---|---|---|---|
| `vector` | O(1) | O(n) | O(1) amortized (back) / O(n) | O(n) |
| `deque` | O(1) | O(n) | O(1) (ends) / O(n) (middle) | O(1) (ends) / O(n) |
| `list` | O(n) | O(n) | O(1) | O(1) |
| `set` / `map` | O(log n) | O(log n) | O(log n) | O(log n) |
| `unordered_set` / `unordered_map` | — | O(1) avg / O(n) worst | O(1) avg | O(1) avg |
| `priority_queue` | O(1) (top) | — | O(log n) | O(log n) |
| `stack` / `queue` | O(1) (ends) | — | O(1) | O(1) |

---

## 14. Which Container Should I Use?

| Need | Use |
|---|---|
| Default dynamic array | `vector` |
| Fixed-size array, stack-allocated | `array` |
| Frequent insert/remove at both ends | `deque` |
| Frequent insert/remove in the middle, no random access needed | `list` |
| LIFO behavior | `stack` |
| FIFO behavior | `queue` |
| Always need the max/min quickly | `priority_queue` |
| Sorted unique keys, need range queries | `set` |
| Sorted key-value pairs | `map` |
| Fastest possible lookup, order doesn't matter | `unordered_set` / `unordered_map` |
| Value that might not exist | `optional` |
| One of several known types | `variant` |
| Truly arbitrary/unknown type at compile time | `any` |

```cpp
// Quick all-in-one include for competitive programming / prototyping
#include <bits/stdc++.h>   // GCC-only, pulls in the entire standard library — avoid in production code
```