# Janet Language Cheat Sheet

Janet is a small, imperative-and-functional Lisp with fibers (coroutines), a built-in PEG engine, and easy C embedding. This guide summarizes the core concepts you need to be productive.

## Table of Contents

- [1. Basics & Syntax](#1-basics--syntax)
- [2. Data Types & Structures](#2-data-types--structures)
- [3. Functions, Binding & Control Flow](#3-functions-binding--control-flow)
- [4. Object-Oriented Programming](#4-object-oriented-programming)
- [5. Janet-Specific Language Features](#5-janet-specific-language-features)
- [6. Managing a Janet Project (jpm)](#6-managing-a-janet-project-jpm)
- [7. Integrating Janet with C / C++](#7-integrating-janet-with-c--c)
- [8. Advanced Data Structures for Dynamic Programming](#8-advanced-data-structures-for-dynamic-programming)
- [9. Quick Reference Tables](#9-quick-reference-tables)

---

## 1. Basics & Syntax

Janet uses prefix (Polish) notation like every Lisp: `(function arg1 arg2 ...)`.

```janet
(print "Hello, world!")          # prints a line
(+ 1 2 3)                        # => 6
(def x 10)                       # immutable binding
(var y 20)                       # mutable binding
(comment "block comments don't exist, use # for line comments")
```

| Syntax | Meaning |
|---|---|
| `# ...` | line comment |
| `(f a b)` | function call |
| `[a b c]` | array literal (mutable, dynamic) |
| `(a b c)` | tuple literal (immutable) |
| `{:k v}` | struct literal (immutable hash) |
| `@{:k v}` | table literal (mutable hash) |
| `@[1 2 3]` | array (same as `[]` written with `@`) |
| `` `sym `` | quote |
| `~(a b)` | quasiquote |
| `,x` | unquote (inside quasiquote) |

Run a file: `janet myfile.janet`. Start a REPL: `janet`.

---

## 2. Data Types & Structures

### 2.1 Primitive Types

| Type | Example | Notes |
|---|---|---|
| `nil` | `nil` | absence of value / false-ish |
| `boolean` | `true`, `false` | |
| `number` | `1`, `3.14`, `1/3` | all numbers are IEEE-754 doubles |
| `string` | `"hi"` | immutable byte sequence |
| `keyword` | `:name` | interned, self-evaluating symbol-like value |
| `symbol` | `'name` | identifier, evaluates to a binding |
| `buffer` | `@"hi"` | mutable byte sequence |

### 2.2 Collection Types

| Type | Literal | Mutable? | Ordered? | Typical use |
|---|---|---|---|---|
| Array | `@[1 2 3]` | ✅ | ✅ | growable lists |
| Tuple | `(1 2 3)` or `[1 2 3]` (bracket tuple) | ❌ | ✅ | fixed sequences, function args, AST nodes |
| Table | `@{:a 1 :b 2}` | ✅ | ❌ | hash maps, "objects" |
| Struct | `{:a 1 :b 2}` | ❌ | ❌ | immutable records, config maps |
| Fiber | `(fiber/new (fn [] ...))` | n/a | n/a | coroutines / green threads |

> Bracket tuples `[1 2 3]` behave like tuples but are parsed with different bracket rules useful for destructuring; parenthesized tuples `(1 2 3)` are indistinguishable at runtime from a function call form when quoted.

```janet
(def arr @[1 2 3])          # array
(array/push arr 4)          # mutate: @[1 2 3 4]

(def tup (1 2 3))           # tuple — actually needs quoting: '(1 2 3)
(def tup2 [1 2 3])          # bracket tuple, no quoting needed

(def tbl @{:name "Ana" :age 30})
(put tbl :age 31)           # mutate table

(def st {:name "Ana" :age 30})  # struct, immutable
```

### 2.3 Common Operations

```janet
(length arr)                 # size of any collection
(get tbl :name)               # safe lookup, nil if missing
(tbl :name)                   # tables/structs are callable as accessors
(in tbl :name)                 # like get but errors if not found
(freeze arr)                  # array -> tuple (immutable copy)
(thaw tup)                    # tuple -> array (mutable copy)
```

---

## 3. Functions, Binding & Control Flow

```janet
(defn add [a b] (+ a b))          # named function
(def add2 (fn [a b] (+ a b)))     # anonymous function bound to a name

(defn variadic [& args] (length args))   # varargs
(defn opt-args [a &opt b] (or b 10))     # optional args
(defn kw-args [&keys {:x x}] x)          # keyword args (via struct)
```

### Control Flow

| Form | Purpose |
|---|---|
| `(if cond then else)` | branching |
| `(when cond & body)` | if without else |
| `(unless cond & body)` | inverse of `when` |
| `(cond c1 r1 c2 r2 ... default)` | multi-branch |
| `(case val v1 r1 v2 r2 default)` | match on value |
| `(while cond & body)` | loop |
| `(for i 0 10 & body)` | numeric loop |
| `(loop [x :in coll] & body)` | generic iteration/comprehension |
| `(each x coll & body)` | shorthand iteration |

```janet
(loop [i :range [0 5] :when (even? i)]
  (print i))                      # loop is a powerful comprehension DSL

(def evens (seq [i :range [0 10] :when (even? i)] i))  # collect into array
```

### Error Handling

```janet
(try
  (error "boom")
  ([err] (print "caught: " err)))

(defer (print "cleanup")
  (do-risky-thing))
```

---

## 4. Object-Oriented Programming

Janet has **no built-in class system**, but idiomatic OOP is built from tables, prototypes, and closures.

### 4.1 Prototype-Based Objects (Tables + Prototypes)

Tables support a **prototype chain** via `table/setproto`, similar to JavaScript/Self.

```janet
(def Animal @{:speak (fn [self] (print (self :name) " makes a sound"))})

(defn new-animal [name]
  (table/setproto @{:name name} Animal))

(def dog (new-animal "Rex"))
((dog :speak) dog)     # => "Rex makes a sound"
```

### 4.2 Struct-Based "Classes" with Method Tables

```janet
(def Point
  @{:new (fn [x y] @{:x x :y y})
    :add (fn [self other] (Point/new (+ (self :x) (other :x))
                                       (+ (self :y) (other :y))))
    :to-string (fn [self] (string/format "(%d, %d)" (self :x) (self :y)))})
```

### 4.3 `defstruct`-style pattern with closures (encapsulation)

```janet
(defn make-counter []
  (var count 0)
  @{:inc (fn [] (++ count) count)
    :get (fn [] count)})

(def c (make-counter))
((c :inc))     # => 1
((c :get))     # => 1
```

### 4.4 Inheritance via Prototype Chains

```janet
(def Base   @{:greet (fn [self] (print "Hi, I'm " (self :name)))})
(def Derived (table/setproto @{:extra (fn [self] (print "Extra behavior"))} Base))

(def obj (table/setproto @{:name "Obj"} Derived))
((obj :greet) obj)   # inherited from Base
((obj :extra) obj)   # from Derived
```

> There is also the community `spork` library and other object-system implementations (e.g. `janet-oo`) if you want classical class/inheritance macros — but the table + prototype approach above is the "batteries included" way.

---

## 5. Janet-Specific Language Features

### 5.1 Macros

```janet
(defmacro my-unless [cond & body]
  ~(if ,cond nil (do ,;body)))

(my-unless false (print "runs"))
```

| Symbol | Meaning |
|---|---|
| `` ` `` | quote (quasiquote when nested with `,`) |
| `,` | unquote |
| `,;` | unquote-splice (splice a sequence in) |
| `;` | splice operator generally (function args, array literals) |

```janet
(def args @[1 2 3])
(+ ;args)              # splice array as arguments => 6
```

### 5.2 Fibers (Coroutines / Green Threads)

Fibers are Janet's core concurrency primitive — lightweight, resumable stack-based coroutines.

```janet
(def f (fiber/new (fn []
                     (yield 1)
                     (yield 2)
                     3)))
(resume f)   # => 1
(resume f)   # => 2
(resume f)   # => 3
```

### 5.3 The Event Loop & `ev` (async I/O)

```janet
(import spork/netrepl)  # example of ev-based networking libs

(ev/spawn
  (print "running in a background fiber")
  (ev/sleep 1)
  (print "one second later"))
```

### 5.4 PEG (Parsing Expression Grammars) — built-in

```janet
(peg/match '(some (choice "a" "b" "c")) "abc")   # => @[]  (matched)

(def grammar
  ~{:digit (range "09")
    :number (some :digit)
    :main :number})
(peg/match grammar "12345")
```

### 5.5 `defglobal`, Dynamic Bindings (`dyn`/`setdyn`)

```janet
(setdyn :my-var 42)
(dyn :my-var)         # => 42, thread/fiber-local dynamic binding
```

### 5.6 Pattern Matching with `match`

```janet
(match [1 2]
  [a b] (print "got pair " a " " b)
  _     (print "no match"))
```

### 5.7 Immutability by Default

Tuples and structs are immutable; arrays and tables are mutable. This distinction (rather than a single "list" type) is central to idiomatic Janet.

---

## 6. Managing a Janet Project (jpm)

`jpm` (**J**anet **P**roject **M**anager) builds, tests, and installs Janet projects using a `project.janet` manifest.

### 6.1 Basic `project.janet`

```janet
(declare-project
  :name "my-project"
  :description "An example Janet project"
  :dependencies ["https://github.com/janet-lang/spork.git"])

(declare-source
  :source ["mylib.janet"])

(declare-executable
  :name "myapp"
  :entry "main.janet")
```

### 6.2 Common `jpm` Commands

| Command | Purpose |
|---|---|
| `jpm init` | scaffold a new project |
| `jpm deps` | install declared dependencies |
| `jpm build` | build native/executable targets |
| `jpm install` | install project into the local module tree |
| `jpm test` | run tests in `test/` |
| `jpm quickbin main.janet out` | bundle a script into a standalone binary |
| `jpm clean` | remove build artifacts |

### 6.3 Importing Local Modules

```janet
(import ./utils)          # relative import from same directory
(import ./lib/helpers :as h)
(h/do-something)

(use ./utils)              # like import but pulls symbols into current namespace
```

### 6.4 Declaring a Local, Reusable Module

Project layout:

```
my-project/
├── project.janet
├── src/
│   └── mylib.janet
└── test/
    └── test-mylib.janet
```

```janet
# project.janet
(declare-project :name "mylib")
(declare-source :source ["src/mylib.janet"])
```

```janet
# test/test-mylib.janet
(import ../src/mylib :as mylib)
(assert (= (mylib/add 1 2) 3))
```

### 6.5 Package Manifest Alternative: `bundle`/`spork` workflow

Newer Janet tooling favors **bundles** (`jpm-pm`/`spork`) for more complex, cross-language builds (Rust, Zig, etc.), while classic `project.janet` remains the simplest path for pure-Janet or simple C extensions.

---

## 7. Integrating Janet with C / C++

Janet embeds cleanly into C programs and can also be extended with native C modules.

### 7.1 Embedding Janet in a C Host Application

```c
#include <janet.h>

int main(int argc, const char **argv) {
    janet_init();
    JanetTable *env = janet_core_env(NULL);
    janet_dostring(env, "(print \"Hello from Janet!\")", "main", NULL);
    janet_deinit();
    return 0;
}
```

Compile against `libjanet` (usually via `jpm` or by linking `janet.o`/`libjanet.a` directly with `-ljanet`).

### 7.2 Writing a Native C Module (extension)

```c
#include <janet.h>

static Janet cfun_add(int32_t argc, Janet *argv) {
    janet_fixarity(argc, 2);
    double a = janet_getnumber(argv, 0);
    double b = janet_getnumber(argv, 1);
    return janet_wrap_number(a + b);
}

static const JanetReg cfuns[] = {
    {"add", cfun_add, "(mymod/add a b)\n\nAdd two numbers."},
    {NULL, NULL, NULL}
};

JANET_MODULE_ENTRY(JanetTable *env) {
    janet_cfuns(env, "mymod", cfuns);
}
```

Declare and build it via `project.janet`:

```janet
(declare-project :name "mymod")
(declare-native :name "mymod" :source ["mymod.c"])
```

```bash
jpm build      # compiles mymod.so / mymod.dll
```

Then use it from Janet:

```janet
(import mymod)
(mymod/add 2 3)   # => 5
```

### 7.3 Key C API Concepts

| C API concept | Purpose |
|---|---|
| `Janet` | tagged union representing any Janet value |
| `janet_wrap_*` / `janet_unwrap_*` | convert between C types and `Janet` |
| `janet_getnumber`, `janet_getstring`, ... | argument-parsing helpers with type checks |
| `janet_fixarity` / `janet_arity` | enforce argument count |
| `JanetReg` | table describing a C function's name/pointer/docstring |
| `janet_cfuns` | registers an array of `JanetReg` into an environment/table |
| `JANET_MODULE_ENTRY` | the exported entry point `jpm`/`import` loads |

### 7.4 Calling C++ Code

Janet's C API is pure C, so C++ code must be wrapped with `extern "C"`:

```cpp
extern "C" {
#include <janet.h>
}

extern "C" Janet cfun_cpp_add(int32_t argc, Janet *argv) {
    janet_fixarity(argc, 2);
    return janet_wrap_number(janet_getnumber(argv, 0) + janet_getnumber(argv, 1));
}
```

Compile the `.cpp` file with a C++ compiler and link it the same way as a `.c` native module (adjust `project.janet` to point at the `.cpp` source; `jpm` will pick a matching compiler for the file extension).

---

## 8. Advanced Data Structures for Dynamic Programming

Janet's mutable `table` and `array` types make classic DP patterns straightforward.

### 8.1 Memoization with Tables

```janet
(def memo @{})

(defn fib [n]
  (if (< n 2)
    n
    (or (get memo n)
        (let [result (+ (fib (- n 1)) (fib (- n 2)))]
          (put memo n result)
          result))))

(fib 40)   # fast, memoized
```

### 8.2 Generic Memoization Wrapper

```janet
(defn memoize [f]
  (def cache @{})
  (fn [& args]
    (def key (freeze args))          # tuples make good hash keys
    (or (get cache key)
        (let [result (f ;args)]
          (put cache key result)
          result))))

(def fib-memo (memoize (fn [n & _] (if (< n 2) n (+ (fib-memo (- n 1)) (fib-memo (- n 2)))))))
```

### 8.3 Bottom-Up (Tabulation) with Arrays

```janet
(defn fib-tab [n]
  (def dp (array/new-filled (+ n 1) 0))
  (put dp 1 1)
  (for i 2 (+ n 1)
    (put dp i (+ (get dp (- i 1)) (get dp (- i 2)))))
  (get dp n))
```

### 8.4 2D DP Tables (Array of Arrays)

```janet
(defn make-grid [rows cols init]
  (seq [_ :range [0 rows]] (array/new-filled cols init)))

(defn lcs-length [s1 s2]
  (def m (length s1)) (def n (length s2))
  (def dp (make-grid (+ m 1) (+ n 1) 0))
  (for i 1 (+ m 1)
    (for j 1 (+ n 1)
      (put (get dp i)
           j
           (if (= (get s1 (- i 1)) (get s2 (- j 1)))
             (+ 1 (get (get dp (- i 1)) (- j 1)))
             (max (get (get dp (- i 1)) j) (get (get dp i) (- j 1)))))))
  (get (get dp m) n))
```

### 8.5 Priority Queues / Sets for Graph DP

Janet ships no built-in heap, but the `spork/heap` module (or a simple sorted array) covers Dijkstra/A*-style DP:

```janet
(import spork/heap)

(def pq (heap/new (fn [a b] (< (a :priority) (b :priority)))))
(heap/push pq @{:priority 3 :node :a})
(heap/pop pq)
```

### 8.6 Structural Sharing Trick: Structs/Tuples as Keys

Because **structs and tuples are immutable and hashable by value**, they're ideal composite keys for multi-dimensional memoization (e.g., knapsack `(i, capacity)` states):

```janet
(def memo @{})
(defn knapsack [i cap weights values]
  (def key [i cap])
  (or (get memo key)
      (let [result
            (cond
              (or (= i 0) (= cap 0)) 0
              (> (get weights (- i 1)) cap) (knapsack (- i 1) cap weights values)
              (max (knapsack (- i 1) cap weights values)
                   (+ (get values (- i 1))
                      (knapsack (- i 1) (- cap (get weights (- i 1))) weights values))))]
        (put memo key result)
        result)))
```

---

## 9. Quick Reference Tables

### 9.1 Truthiness

| Value | Truthy? |
|---|---|
| `nil` | ❌ false |
| `false` | ❌ false |
| `0` | ✅ true (unlike many C-family langs!) |
| `""` (empty string) | ✅ true |
| everything else | ✅ true |

### 9.2 Mutable vs. Immutable Cheat Sheet

| Mutable | Immutable equivalent |
|---|---|
| `array` (`@[]`) | `tuple` (`()`/`[]`) |
| `table` (`@{}`) | `struct` (`{}`) |
| `buffer` (`@""`) | `string` (`""`) |

### 9.3 Useful Core Functions

| Function | Purpose |
|---|---|
| `map`, `filter`, `reduce` | classic functional combinators |
| `string/split`, `string/join` | string utilities |
| `freeze` / `thaw` | convert between mutable/immutable |
| `table/setproto` / `table/getproto` | prototype-based OOP |
| `marshal` / `unmarshal` | serialize Janet values (great for caching DP state) |
| `os/exit`, `os/time` | OS interaction |

### 9.4 Module System Keywords

| Form | Meaning |
|---|---|
| `(import mod)` | import as `mod/symbol` |
| `(import mod :as m)` | import with alias |
| `(import mod :prefix "")` | import symbols unprefixed |
| `(use mod)` | import + merge symbols into current scope |
| `(require "mod")` | lower-level dynamic import (returns env table) |

---

**Further reading:** the official docs at `janet-lang.org/docs`, the C API reference at `janet-lang.org/capi`, and the `jpm` manual at `janet-lang.org/docs/jpm.html`.