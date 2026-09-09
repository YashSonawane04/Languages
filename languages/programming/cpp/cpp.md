# C++ Concepts Reference (C++20 / C++23)

A condensed, practical reference covering core C++ concepts, syntax, and project tooling. Examples target **C++20/C++23** (the latest widely-supported standards) — where a feature is version-specific, it's noted.

---

## Table of Contents

- [1. Data Types and Structures](#1-data-types-and-structures)
  - [1.1 Fundamental Data Types](#11-fundamental-data-types)
  - [1.2 Type Modifiers & Qualifiers](#12-type-modifiers--qualifiers)
  - [1.3 Variables & `auto`](#13-variables--auto)
  - [1.4 Arrays](#14-arrays)
  - [1.5 Pointers & References](#15-pointers--references)
  - [1.6 Structures, Unions, Enums](#16-structures-unions-enums)
- [2. Object-Oriented Programming](#2-object-oriented-programming)
  - [2.1 Classes & Access Specifiers](#21-classes--access-specifiers)
  - [2.2 Constructors & Destructors](#22-constructors--destructors)
  - [2.3 Inheritance](#23-inheritance)
  - [2.4 Polymorphism](#24-polymorphism)
  - [2.5 Abstract Classes & Interfaces](#25-abstract-classes--interfaces)
  - [2.6 Operator Overloading](#26-operator-overloading)
- [3. C++ Language-Specific Features](#3-c-language-specific-features)
  - [3.1 Templates & Concepts](#31-templates--concepts)
  - [3.2 Standard Template Library (STL)](#32-standard-template-library-stl)
  - [3.3 Smart Pointers](#33-smart-pointers)
  - [3.4 Lambda Expressions](#34-lambda-expressions)
  - [3.5 Move Semantics & Rvalue References](#35-move-semantics--rvalue-references)
  - [3.6 RAII](#36-raii)
  - [3.7 Exception Handling](#37-exception-handling)
  - [3.8 `constexpr` / `consteval` / `constinit`](#38-constexpr--consteval--constinit)
  - [3.9 Namespaces](#39-namespaces)
  - [3.10 Modules (C++20)](#310-modules-c20)
  - [3.11 Ranges (C++20)](#311-ranges-c20)
  - [3.12 Coroutines (C++20)](#312-coroutines-c20)
- [4. Managing a C++ Project](#4-managing-a-c-project)
  - [4.1 Typical Project Layout](#41-typical-project-layout)
  - [4.2 Headers, Include Guards & Local Imports](#42-headers-include-guards--local-imports)
  - [4.3 Build System: Make](#43-build-system-make)
  - [4.4 Build System: CMake](#44-build-system-cmake)
  - [4.5 Build System: Ninja](#45-build-system-ninja)
  - [4.6 Package Managers (brief)](#46-package-managers-brief)
- [5. Advanced Data Structures for Dynamic Programming](#5-advanced-data-structures-for-dynamic-programming)
  - [5.1 Core DP Containers](#51-core-dp-containers)
  - [5.2 Memoization Pattern](#52-memoization-pattern)
  - [5.3 Tabulation Pattern (2D DP)](#53-tabulation-pattern-2d-dp)
  - [5.4 Useful STL Structures for DP/Graphs](#54-useful-stl-structures-for-dpgraphs)
  - [5.5 Example: 0/1 Knapsack](#55-example-01-knapsack)
  - [5.6 Example: Longest Common Subsequence](#56-example-longest-common-subsequence)

---

## 1. Data Types and Structures

### 1.1 Fundamental Data Types

| Type | Size (typical) | Description | Example |
|---|---|---|---|
| `bool` | 1 byte | true/false | `bool ok = true;` |
| `char` | 1 byte | single character | `char c = 'A';` |
| `int` | 4 bytes | integer | `int x = 42;` |
| `short` | 2 bytes | short integer | `short s = 10;` |
| `long` | 8 bytes | long integer | `long l = 100000L;` |
| `long long` | 8 bytes | very large integer | `long long ll = 123456789LL;` |
| `float` | 4 bytes | single precision FP | `float f = 3.14f;` |
| `double` | 8 bytes | double precision FP | `double d = 3.14159;` |
| `long double` | 12/16 bytes | extended precision FP | `long double ld = 3.14L;` |
| `wchar_t`/`char8_t`/`char16_t`/`char32_t` | varies | wide/Unicode chars (C++11/20) | `char32_t u = U'λ';` |
| `void` | — | no type (functions/pointers) | `void foo();` |

> Sizes are implementation-defined. Use `<cstdint>` for fixed-width types like `int32_t`, `uint64_t` when portability matters.

### 1.2 Type Modifiers & Qualifiers

```cpp
unsigned int u = 5;      // no negative values
signed int   s = -5;     // default for int
const int    C = 10;     // cannot be modified
volatile int v;          // may change outside program control
static int   counter;    // persists across calls / internal linkage
mutable int  cache;      // modifiable even in a const object (class member only)
```

### 1.3 Variables & `auto`

```cpp
auto x = 5;               // int (type deduced)
auto y = 3.14;             // double
auto name = std::string{"C++"};
decltype(x) z = 10;       // z has the same type as x

// Structured bindings (C++17) — unpack tuples/pairs/structs
auto [a, b] = std::pair{1, 2};
```

### 1.4 Arrays

```cpp
int arr[5] = {1, 2, 3, 4, 5};        // C-style array
int matrix[2][3] = {{1,2,3},{4,5,6}}; // 2D array

#include <array>
std::array<int, 5> stdArr = {1, 2, 3, 4, 5}; // fixed-size, STL-aware
```

### 1.5 Pointers & References

```cpp
int val = 10;
int* ptr = &val;      // pointer to val
int& ref = val;       // reference (alias) to val

*ptr = 20;             // dereference and modify
std::cout << ref;      // prints 20

int* nullPtr = nullptr; // C++11 null pointer literal

// Function parameters
void byValue(int x);
void byPointer(int* x);
void byReference(int& x);
void byConstReference(const int& x); // avoids copy, disallows modification
```

### 1.6 Structures, Unions, Enums

```cpp
struct Point {
    int x;
    int y;
};
Point p{3, 4}; // aggregate initialization

union Data {
    int i;
    float f;
}; // members share the same memory

enum class Color { Red, Green, Blue }; // scoped enum (preferred, C++11+)
Color c = Color::Red;

enum Legacy { A, B, C }; // unscoped (avoid in new code — pollutes namespace)
```

---

## 2. Object-Oriented Programming

### 2.1 Classes & Access Specifiers

| Specifier | Accessible from |
|---|---|
| `public` | anywhere |
| `protected` | class itself & derived classes |
| `private` | class itself only |

```cpp
class Animal {
public:
    Animal(std::string name) : name_(name) {}
    void speak() const { std::cout << name_ << " makes a sound\n"; }

protected:
    std::string name_;

private:
    int secretId_ = 0;
};
```

### 2.2 Constructors & Destructors

```cpp
class Widget {
public:
    Widget() { std::cout << "Default ctor\n"; }
    Widget(int size) : size_(size) {}              // parameterized ctor
    Widget(const Widget& other) = default;          // copy ctor
    Widget(Widget&& other) noexcept = default;      // move ctor (C++11)
    Widget& operator=(const Widget&) = default;      // copy assignment
    Widget& operator=(Widget&&) noexcept = default;   // move assignment
    ~Widget() { std::cout << "Destructor\n"; }

private:
    int size_ = 0;
};
```

### 2.3 Inheritance

```cpp
class Base {
public:
    virtual void greet() const { std::cout << "Base\n"; }
    virtual ~Base() = default; // virtual destructor is important!
};

class Derived : public Base {
public:
    void greet() const override { std::cout << "Derived\n"; } // 'override' catches mistakes
};
```

| Inheritance Type | `public` members become | `protected` members become |
|---|---|---|
| `public`    | public    | protected |
| `protected` | protected | protected |
| `private`   | private   | private   |

### 2.4 Polymorphism

```cpp
Base* b = new Derived();
b->greet();          // "Derived" — resolved at runtime via vtable (dynamic dispatch)
delete b;

// Static (compile-time) polymorphism via templates/overloading
void print(int x)    { std::cout << "int: " << x << "\n"; }
void print(double x) { std::cout << "double: " << x << "\n"; }
```

### 2.5 Abstract Classes & Interfaces

```cpp
class Shape {
public:
    virtual double area() const = 0; // pure virtual -> abstract class
    virtual ~Shape() = default;
};

class Circle : public Shape {
public:
    explicit Circle(double r) : r_(r) {}
    double area() const override { return 3.14159 * r_ * r_; }
private:
    double r_;
};
```

### 2.6 Operator Overloading

```cpp
class Vec2 {
public:
    double x, y;
    Vec2 operator+(const Vec2& o) const { return {x + o.x, y + o.y}; }
    bool operator==(const Vec2& o) const = default; // C++20 defaulted comparison
    friend std::ostream& operator<<(std::ostream& os, const Vec2& v) {
        return os << "(" << v.x << ", " << v.y << ")";
    }
};
```

---

## 3. C++ Language-Specific Features

### 3.1 Templates & Concepts

```cpp
// Function template
template <typename T>
T maxOf(T a, T b) { return (a > b) ? a : b; }

// Class template
template <typename T>
class Box {
public:
    explicit Box(T value) : value_(value) {}
    T get() const { return value_; }
private:
    T value_;
};

// Concepts (C++20) — constrain template parameters
template <typename T>
concept Numeric = std::is_arithmetic_v<T>;

template <Numeric T>
T square(T x) { return x * x; }

// Abbreviated function template (C++20)
auto add(auto a, auto b) { return a + b; }
```

### 3.2 Standard Template Library (STL)

Look at [stl.md](./stl.md) for better 

| Category | Examples | Header |
|---|---|---|
| Sequence containers | `vector`, `array`, `deque`, `list`, `forward_list` | `<vector>`, `<array>`, `<deque>`, `<list>` |
| Associative containers | `map`, `set`, `multimap`, `multiset` | `<map>`, `<set>` |
| Unordered containers | `unordered_map`, `unordered_set` | `<unordered_map>`, `<unordered_set>` |
| Container adapters | `stack`, `queue`, `priority_queue` | `<stack>`, `<queue>` |
| Algorithms | `sort`, `find`, `accumulate`, `count_if` | `<algorithm>`, `<numeric>` |
| Iterators | `begin()`, `end()`, `rbegin()` | `<iterator>` |

```cpp
#include <vector>
#include <algorithm>

std::vector<int> v = {5, 3, 1, 4, 2};
std::sort(v.begin(), v.end());                       // ascending sort
auto it = std::find(v.begin(), v.end(), 4);            // returns iterator
for (const auto& x : v) std::cout << x << " ";         // range-based for (C++11)
```

### 3.3 Smart Pointers

```cpp
#include <memory>

std::unique_ptr<Widget> up = std::make_unique<Widget>();  // sole ownership
std::shared_ptr<Widget> sp = std::make_shared<Widget>();  // shared ownership (ref-counted)
std::weak_ptr<Widget>   wp = sp;                            // non-owning observer
```

| Smart Pointer | Ownership | Use Case |
|---|---|---|
| `unique_ptr` | exclusive | default choice, RAII resource management |
| `shared_ptr` | shared (ref-counted) | multiple owners needed |
| `weak_ptr` | none | break `shared_ptr` cycles, cache/observer patterns |

### 3.4 Lambda Expressions

```cpp
auto add = [](int a, int b) { return a + b; };
int threshold = 10;
auto isAbove = [threshold](int x) { return x > threshold; }; // capture by value
auto modify  = [&threshold]() { threshold++; };               // capture by reference
auto generic = [](auto a, auto b) { return a + b; };            // generic lambda (C++14)

// Immediately invoked
int result = [](int x) { return x * x; }(5);
```

### 3.5 Move Semantics & Rvalue References

```cpp
std::vector<int> makeVector() {
    std::vector<int> v = {1, 2, 3};
    return v; // implicitly moved (or elided) — no deep copy
}

void takeOwnership(std::vector<int>&& v) { /* v is an rvalue reference */ }

std::vector<int> a = {1, 2, 3};
std::vector<int> b = std::move(a); // a is now in a valid-but-unspecified state
```

### 3.6 RAII

**R**esource **A**cquisition **I**s **I**nitialization — tie resource lifetime to object lifetime.

```cpp
class FileHandle {
public:
    explicit FileHandle(const std::string& path) : file_(std::fopen(path.c_str(), "r")) {}
    ~FileHandle() { if (file_) std::fclose(file_); } // auto-cleanup, even on exceptions
private:
    FILE* file_;
};
// Smart pointers, std::lock_guard, and std::vector are all RAII examples.
```

### 3.7 Exception Handling

```cpp
#include <stdexcept>

try {
    throw std::runtime_error("Something went wrong");
} catch (const std::runtime_error& e) {
    std::cerr << "Caught: " << e.what() << "\n";
} catch (...) {
    std::cerr << "Unknown exception\n";
}

void mustNotThrow() noexcept { /* promises not to throw */ }
```

### 3.8 `constexpr` / `consteval` / `constinit`

```cpp
constexpr int square(int x) { return x * x; }  // may run at compile-time
constexpr int val = square(5);                  // guaranteed compile-time here

consteval int mustBeCompileTime(int x) { return x * 2; } // C++20: ALWAYS compile-time

constinit int globalCounter = 0; // C++20: guarantees constant initialization, no runtime init order issues
```

### 3.9 Namespaces

```cpp
namespace math {
    int add(int a, int b) { return a + b; }

    namespace advanced {
        int power(int base, int exp);
    }
}

int r = math::add(2, 3);
using namespace math;      // brings all names into scope (avoid in headers)
using math::add;           // brings a single name into scope
```

### 3.10 Modules (C++20)

Modules replace headers with faster-compiling, encapsulated units.

```cpp
// math.ixx (module interface file)
export module math;

export int add(int a, int b) { return a + b; }
```

```cpp
// main.cpp
import math;

int main() { return add(2, 3); }
```

> Compiler/build-system support varies (GCC 14+, Clang 17+, MSVC — check current status). Traditional headers remain the portable default for most projects today.

### 3.11 Ranges (C++20)

```cpp
#include <ranges>
#include <vector>

std::vector<int> nums = {1,2,3,4,5,6,7,8,9,10};

auto evenSquares = nums
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * n; });

for (int n : evenSquares) std::cout << n << " "; // 4 16 36 64 100
```

### 3.12 Coroutines (C++20)

```cpp
#include <coroutine>
// Requires a user-defined return type implementing the coroutine protocol
// (promise_type, awaitables, etc). Common in async I/O and generator libraries.
generator<int> counter() {
    int i = 0;
    while (true) co_yield i++;
}
```

---

## 4. Managing a C++ Project

### 4.1 Typical Project Layout

```
my_project/
├── CMakeLists.txt
├── include/
│   └── my_project/
│       └── math_utils.hpp
├── src/
│   ├── main.cpp
│   └── math_utils.cpp
├── tests/
│   └── test_math_utils.cpp
├── build/            # generated, not committed
└── third_party/      # vendored deps (optional)
```

### 4.2 Headers, Include Guards & Local Imports

```cpp
// include/my_project/math_utils.hpp
#pragma once   // modern include guard (or use #ifndef/#define/#endif)

namespace my_project {
    int add(int a, int b);
}
```

```cpp
// src/math_utils.cpp
#include "my_project/math_utils.hpp"

namespace my_project {
    int add(int a, int b) { return a + b; }
}
```

```cpp
// src/main.cpp — importing your own local module/header
#include "my_project/math_utils.hpp"   // local project header (quotes)
#include <iostream>                     // standard library header (angle brackets)

int main() {
    std::cout << my_project::add(2, 3) << "\n";
}
```

| Include style | Meaning |
|---|---|
| `#include <header>` | search standard/system include paths |
| `#include "header"` | search local project dirs first, then system paths |

### 4.3 Build System: Make

```makefile
# Makefile
CXX      := g++
CXXFLAGS := -std=c++20 -Wall -Wextra -Iinclude
SRC      := $(wildcard src/*.cpp)
OBJ      := $(SRC:src/%.cpp=build/%.o)
TARGET   := build/my_app

$(TARGET): $(OBJ)
	$(CXX) $(OBJ) -o $@

build/%.o: src/%.cpp
	@mkdir -p build
	$(CXX) $(CXXFLAGS) -c $< -o $@

clean:
	rm -rf build

.PHONY: clean
```

```bash
make          # build
make clean    # remove build artifacts
```

### 4.4 Build System: CMake

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.20)
project(MyProject LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(my_app src/main.cpp src/math_utils.cpp)
target_include_directories(my_app PRIVATE include)

# Example: pulling in a dependency
# find_package(fmt REQUIRED)
# target_link_libraries(my_app PRIVATE fmt::fmt)
```

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/my_app
```

| CMake Concept | Purpose |
|---|---|
| `add_executable` | define a binary target |
| `add_library` | define a static/shared library target |
| `target_link_libraries` | link dependencies to a target |
| `target_include_directories` | add header search paths |
| `find_package` | locate installed dependencies |

### 4.5 Build System: Ninja

Ninja is a fast, low-level build executor — typically used **as CMake's backend**, or standalone with a hand-written `build.ninja`.

**Via CMake (most common):**

```bash
cmake -S . -B build -G Ninja
ninja -C build
```

**Standalone `build.ninja`:**

```ninja
cxx = g++
cxxflags = -std=c++20 -Wall -Iinclude

rule cxx_compile
  command = $cxx $cxxflags -c $in -o $out

rule cxx_link
  command = $cxx $in -o $out

build build/main.o: cxx_compile src/main.cpp
build build/math_utils.o: cxx_compile src/math_utils.cpp
build build/my_app: cxx_link build/main.o build/math_utils.o

default build/my_app
```

```bash
ninja      # builds using build.ninja in current directory
```

### 4.6 Package Managers (brief)

| Tool | Notes |
|---|---|
| **vcpkg** | Microsoft-backed, integrates directly with CMake |
| **Conan** | Cross-platform, works well for complex dependency graphs |
| **FetchContent** (CMake built-in) | Pulls dependencies straight from source at configure time |

```cmake
include(FetchContent)
FetchContent_Declare(fmt GIT_REPOSITORY https://github.com/fmtlib/fmt.git GIT_TAG 10.2.1)
FetchContent_MakeAvailable(fmt)
target_link_libraries(my_app PRIVATE fmt::fmt)
```

---

## 5. Advanced Data Structures for Dynamic Programming

### 5.1 Core DP Containers

| Structure | Use Case |
|---|---|
| `std::vector<int> dp(n)` | 1D bottom-up table |
| `std::vector<std::vector<int>> dp(n, std::vector<int>(m))` | 2D bottom-up table |
| `std::unordered_map<Key, Value>` | sparse memoization (e.g. keyed by state hash) |
| `std::map<Key, Value>` | ordered memoization / need iteration order |
| `std::pair` / `std::tuple` | compound DP state keys |
| `std::priority_queue` | greedy/DP hybrids (Dijkstra, scheduling) |
| `std::bitset<N>` | bitmask DP (subset states) |

### 5.2 Memoization Pattern (Top-Down)

```cpp
#include <unordered_map>

std::unordered_map<int, long long> memo;

long long fib(int n) {
    if (n <= 1) return n;
    if (auto it = memo.find(n); it != memo.end()) return it->second; // C++17 init-if
    return memo[n] = fib(n - 1) + fib(n - 2);
}
```

### 5.3 Tabulation Pattern (2D DP)

```cpp
#include <vector>

int uniquePaths(int rows, int cols) {
    std::vector<std::vector<int>> dp(rows, std::vector<int>(cols, 1));
    for (int r = 1; r < rows; ++r)
        for (int c = 1; c < cols; ++c)
            dp[r][c] = dp[r-1][c] + dp[r][c-1];
    return dp[rows-1][cols-1];
}
```

### 5.4 Useful STL Structures for DP/Graphs

```cpp
#include <queue>
#include <tuple>
#include <bitset>

// Compound key memoization (e.g. state = (index, remainingCapacity))
std::map<std::pair<int,int>, int> stateMemo;

// Bitmask DP state (e.g. Traveling Salesman)
std::bitset<20> visitedMask;

// Min-heap for Dijkstra-style DP
std::priority_queue<
    std::tuple<int,int>,
    std::vector<std::tuple<int,int>>,
    std::greater<>
> minHeap; // {distance, node}, ordered ascending
```

### 5.5 Example: 0/1 Knapsack

```cpp
#include <vector>
#include <algorithm>

int knapsack(int capacity, const std::vector<int>& weights, const std::vector<int>& values) {
    int n = weights.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(capacity + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int w = 0; w <= capacity; ++w) {
            dp[i][w] = dp[i-1][w]; // skip item i-1
            if (weights[i-1] <= w)
                dp[i][w] = std::max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);
        }
    }
    return dp[n][capacity];
}
```

### 5.6 Example: Longest Common Subsequence

```cpp
#include <string>
#include <vector>
#include <algorithm>

int lcs(const std::string& a, const std::string& b) {
    int n = a.size(), m = b.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(m + 1, 0));

    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            dp[i][j] = (a[i-1] == b[j-1])
                ? dp[i-1][j-1] + 1
                : std::max(dp[i-1][j], dp[i][j-1]);

    return dp[n][m];
}
```

---

### Quick Reference: Compiling Everything Above

```bash
g++ -std=c++20 -Wall -Wextra -O2 src/*.cpp -o my_app
```

*Reference compiled for C++20/C++23. Always check your compiler's feature-support table (GCC/Clang/MSVC) before relying on the newest features like Modules or Coroutines in production.*