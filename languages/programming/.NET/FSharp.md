# F# Cheat Sheet

A quick-reference guide to F# fundamentals: data types, OOP, language features, project management, and advanced data structures for problem solving.

## Table of Contents

- [1. Basic Data Types](#1-basic-data-types)
- [2. Data Structures](#2-data-structures)
  - [2.1 Tuples](#21-tuples)
  - [2.2 Records](#22-records)
  - [2.3 Discriminated Unions](#23-discriminated-unions)
  - [2.4 Lists](#24-lists)
  - [2.5 Arrays](#25-arrays)
  - [2.6 Sequences](#26-sequences)
  - [2.7 Options & Results](#27-options--results)
  - [2.8 Maps & Sets](#28-maps--sets)
- [3. Functions](#3-functions)
- [4. Pattern Matching](#4-pattern-matching)
- [5. Object-Oriented Programming](#5-object-oriented-programming)
  - [5.1 Classes](#51-classes)
  - [5.2 Interfaces](#52-interfaces)
  - [5.3 Inheritance & Abstract Classes](#53-inheritance--abstract-classes)
  - [5.4 Properties & Members](#54-properties--members)
- [6. F# Language-Specific Features](#6-f-language-specific-features)
  - [6.1 Pipelines & Composition](#61-pipelines--composition)
  - [6.2 Immutability & `mutable`](#62-immutability--mutable)
  - [6.3 Units of Measure](#63-units-of-measure)
  - [6.4 Computation Expressions](#64-computation-expressions)
  - [6.5 Active Patterns](#65-active-patterns)
  - [6.6 Type Inference & Type Providers](#66-type-inference--type-providers)
- [7. Managing an F# Project](#7-managing-an-f-project)
  - [7.1 Project Structure](#71-project-structure)
  - [7.2 The `.fsproj` File](#72-the-fsproj-file)
  - [7.3 Importing Local Modules](#73-importing-local-modules)
  - [7.4 Namespaces vs Modules](#74-namespaces-vs-modules)
  - [7.5 Package Management](#75-package-management)
  - [7.6 Solutions (`.sln`)](#76-solutions-sln)
- [8. Advanced Data Structures (Dynamic Programming)](#8-advanced-data-structures-dynamic-programming)
  - [8.1 Memoization with `Dictionary`](#81-memoization-with-dictionary)
  - [8.2 2D Arrays / Grids](#82-2d-arrays--grids)
  - [8.3 Mutable Collections (`ResizeArray`, `Dictionary`, `HashSet`)](#83-mutable-collections-resizearray-dictionary-hashset)
  - [8.4 Priority Queues & Heaps](#84-priority-queues--heaps)
  - [8.5 Trees & Graphs](#85-trees--graphs)
  - [8.6 Tail Recursion](#86-tail-recursion)

---

## 1. Basic Data Types

| Type | Example | Notes |
|---|---|---|
| `int` | `let x = 42` | 32-bit signed integer |
| `int64` | `let x = 42L` | 64-bit integer |
| `float` / `double` | `let x = 3.14` | 64-bit floating point |
| `float32` | `let x = 3.14f` | 32-bit floating point |
| `decimal` | `let x = 3.14m` | High precision, used for money |
| `bool` | `let b = true` | `true` / `false` |
| `char` | `let c = 'a'` | Single character |
| `string` | `let s = "hello"` | Immutable string |
| `unit` | `()` | Equivalent to "void", represents "no meaningful value" |
| `byte` | `let b = 5uy` | 8-bit unsigned |

```fsharp
// Type annotations are optional (type inference), but can be explicit:
let age : int = 30
let name : string = "Ada"

// String interpolation
let greeting = $"Hello, {name}! You are {age} years old."
```

---

## 2. Data Structures

### 2.1 Tuples

Fixed-size, heterogeneous groupings.

```fsharp
let point = (3, 4)
let name, age = ("Ada", 36)          // Destructuring
let triple = (1, "two", 3.0)

// Accessing
let x = fst point
let y = snd point
```

### 2.2 Records

Immutable, named-field data structures — F#'s go-to for structured data.

```fsharp
type Person = { Name: string; Age: int }

let ada = { Name = "Ada"; Age = 36 }

// "With" syntax for copy-and-update (records are immutable)
let olderAda = { ada with Age = 37 }

// Access
printfn "%s is %d" ada.Name ada.Age
```

### 2.3 Discriminated Unions

F#'s algebraic data types — perfect for modeling "one of several" cases.

```fsharp
type Shape =
    | Circle of radius: float
    | Rectangle of width: float * height: float
    | Triangle of baseLen: float * height: float

let area shape =
    match shape with
    | Circle r -> System.Math.PI * r * r
    | Rectangle (w, h) -> w * h
    | Triangle (b, h) -> 0.5 * b * h

// Common built-in DUs: Option and Result (see 2.7)
```

### 2.4 Lists

Immutable, singly linked lists.

```fsharp
let numbers = [ 1; 2; 3; 4; 5 ]
let head = numbers.Head          // 1
let tail = numbers.Tail          // [2; 3; 4; 5]

let combined = 0 :: numbers      // Prepend -> [0;1;2;3;4;5]
let merged   = [1; 2] @ [3; 4]   // Concatenation

// Common functions
List.map   (fun x -> x * 2) numbers
List.filter (fun x -> x % 2 = 0) numbers
List.fold  (fun acc x -> acc + x) 0 numbers
List.sum numbers
List.sortDescending numbers
```

### 2.5 Arrays

Fixed-size, mutable, index-based — better performance for numeric/DP work.

```fsharp
let arr = [| 1; 2; 3; 4 |]
arr.[0] <- 10                    // Mutate in place
let arr2 = Array.zeroCreate<int> 10
let arr3 = Array2D.create 3 3 0  // 2D array, see section 8.2

Array.iter (printfn "%d") arr
```

### 2.6 Sequences

Lazy, potentially infinite, `IEnumerable`-based.

```fsharp
let seq1 = seq { for i in 1 .. 10 -> i * i }
let infinite = Seq.initInfinite (fun i -> i * 2)
let firstFive = infinite |> Seq.take 5 |> Seq.toList
```

### 2.7 Options & Results

Idiomatic replacements for `null` and exceptions.

```fsharp
// Option<'T>: Some x | None
let tryDivide a b =
    if b = 0 then None else Some (a / b)

match tryDivide 10 2 with
| Some result -> printfn "Result: %d" result
| None -> printfn "Cannot divide by zero"

// Result<'T,'TError>: Ok x | Error e
let safeDivide a b =
    if b = 0 then Error "Division by zero" else Ok (a / b)
```

### 2.8 Maps & Sets

Immutable, tree-based key/value and unique-element collections.

```fsharp
let map = Map.ofList [ ("a", 1); ("b", 2) ]
let updated = map.Add("c", 3)
let value = map.TryFind "a"        // Some 1

let set1 = Set.ofList [1; 2; 3]
let set2 = Set.ofList [2; 3; 4]
let union = Set.union set1 set2    // {1;2;3;4}
```

---

## 3. Functions

```fsharp
// Basic function
let add x y = x + y

// Explicit types
let add2 (x: int) (y: int) : int = x + y

// Lambda / anonymous function
let square = fun x -> x * x

// Recursive function (needs 'rec')
let rec factorial n =
    if n <= 1 then 1 else n * factorial (n - 1)

// Mutually recursive functions ('and')
let rec isEven n = if n = 0 then true else isOdd (n - 1)
and isOdd n = if n = 0 then false else isEven (n - 1)

// Curried & partial application
let multiply x y = x * y
let double = multiply 2            // Partially applied

// Higher-order functions
let applyTwice f x = f (f x)
```

---

## 4. Pattern Matching

F#'s `match` is exhaustive, expression-based, and works on nearly every type.

```fsharp
let describe x =
    match x with
    | 0 -> "zero"
    | n when n < 0 -> "negative"
    | n when n % 2 = 0 -> "positive even"
    | _ -> "positive odd"

// Matching on tuples
let classify point =
    match point with
    | (0, 0) -> "origin"
    | (_, 0) -> "on x-axis"
    | (0, _) -> "on y-axis"
    | (x, y) -> $"({x},{y})"

// Matching on lists
let rec sumList lst =
    match lst with
    | [] -> 0
    | head :: tail -> head + sumList tail

// Matching on DUs / Options
let describeOpt opt =
    match opt with
    | Some v -> $"Value: {v}"
    | None -> "No value"
```

---

## 5. Object-Oriented Programming

F# is multi-paradigm and fully supports classes, interfaces, and inheritance.

### 5.1 Classes

```fsharp
type Animal(name: string, sound: string) =
    // Primary constructor runs here
    let mutable timesCalled = 0

    member this.Name = name
    member this.MakeSound() =
        timesCalled <- timesCalled + 1
        printfn "%s says %s" name sound

    member this.TimesCalled = timesCalled

let dog = Animal("Rex", "Woof")
dog.MakeSound()
```

### 5.2 Interfaces

```fsharp
type IShape =
    abstract member Area : unit -> float
    abstract member Perimeter : unit -> float

type Square(side: float) =
    interface IShape with
        member this.Area() = side * side
        member this.Perimeter() = 4.0 * side

let sq = Square(4.0) :> IShape   // Upcast to use interface members
printfn "%f" (sq.Area())
```

### 5.3 Inheritance & Abstract Classes

```fsharp
[<AbstractClass>]
type Vehicle(wheels: int) =
    member this.Wheels = wheels
    abstract member Describe : unit -> string
    default this.Describe() = $"A vehicle with {wheels} wheels"

type Car(brand: string) =
    inherit Vehicle(4)
    override this.Describe() = $"A {brand} car with {this.Wheels} wheels"

let car = Car("Toyota")
printfn "%s" (car.Describe())
```

### 5.4 Properties & Members

```fsharp
type Circle(radius: float) =
    let mutable r = radius

    // Auto property with custom get/set
    member this.Radius
        with get () = r
        and set (value) = r <- value

    // Computed (read-only) property
    member this.Area = System.Math.PI * r * r

    // Static member
    static member Unit = Circle(1.0)
```

---

## 6. F# Language-Specific Features

### 6.1 Pipelines & Composition

```fsharp
// Pipe operator |> passes result forward
let result =
    [1..10]
    |> List.filter (fun x -> x % 2 = 0)
    |> List.map (fun x -> x * x)
    |> List.sum

// Function composition >>
let addThenDouble = (+) 1 >> (*) 2
addThenDouble 3   // (3 + 1) * 2 = 8
```

### 6.2 Immutability & `mutable`

```fsharp
let x = 5
// x <- 6      // ERROR: not mutable

let mutable y = 5
y <- 6          // OK

// Reference cells for mutable captured state
let counter = ref 0
counter.Value <- counter.Value + 1
```

### 6.3 Units of Measure

Compile-time dimensional safety for numeric types.

```fsharp
[<Measure>] type kg
[<Measure>] type m

let weight = 5.0<kg>
let distance = 10.0<m>
// let invalid = weight + distance   // Compile error: incompatible units
```

### 6.4 Computation Expressions

Syntactic sugar for chaining computations (async, sequences, custom monads).

```fsharp
// Built-in async workflow
let fetchDataAsync url = async {
    let! response = someAsyncCall url
    return response
}

// Option computation expression (via custom builder or built-in)
let addOptions a b = option {
    let! x = a
    let! y = b
    return x + y
}
```

### 6.5 Active Patterns

Custom, reusable pattern-matching logic.

```fsharp
let (|Even|Odd|) n = if n % 2 = 0 then Even else Odd

let describe n =
    match n with
    | Even -> "even"
    | Odd -> "odd"

// Partial active patterns
let (|Positive|_|) n = if n > 0 then Some n else None
```

### 6.6 Type Inference & Type Providers

```fsharp
// Type inference: no annotations needed most of the time
let add a b = a + b       // Inferred as int -> int -> int

// Type providers (e.g., for CSV, JSON, SQL) generate types at compile-time
// (requires FSharp.Data package)
// type Stocks = CsvProvider<"stocks.csv">
```

---

## 7. Managing an F# Project

### 7.1 Project Structure

A typical F# console/library project:

```
MyProject/
├── MyProject.sln
├── src/
│   └── MyProject/
│       ├── MyProject.fsproj
│       ├── Types.fs
│       ├── Helpers.fs
│       └── Program.fs
└── tests/
    └── MyProject.Tests/
        ├── MyProject.Tests.fsproj
        └── Tests.fs
```

> ⚠️ **File order matters in F#!** Files are compiled top-to-bottom as listed in the `.fsproj`. A file can only reference types/functions defined in files *above* it.

### 7.2 The `.fsproj` File

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <!-- Order matters: compiled top to bottom -->
    <Compile Include="Types.fs" />
    <Compile Include="Helpers.fs" />
    <Compile Include="Program.fs" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

### 7.3 Importing Local Modules

```fsharp
// --- Types.fs ---
module MyProject.Types

type Person = { Name: string; Age: int }

// --- Helpers.fs ---
module MyProject.Helpers

open MyProject.Types    // Import the module above

let greet (p: Person) = $"Hello, {p.Name}!"

// --- Program.fs ---
open MyProject.Types
open MyProject.Helpers

[<EntryPoint>]
let main argv =
    let p = { Name = "Ada"; Age = 36 }
    printfn "%s" (greet p)
    0   // Return code
```

| Task | Command / Syntax |
|---|---|
| Create console app | `dotnet new console -lang F#` |
| Create class library | `dotnet new classlib -lang F#` |
| Add file to project | `dotnet add file.fs` (or edit `.fsproj` directly) |
| Run project | `dotnet run` |
| Build project | `dotnet build` |
| Reference another project | `dotnet add reference ../OtherProject/OtherProject.fsproj` |
| Add a NuGet package | `dotnet add package PackageName` |

### 7.4 Namespaces vs Modules

| Feature | `module` | `namespace` |
|---|---|---|
| Can contain | Values, functions, types | Only types & nested modules/namespaces |
| Can be opened with `open` | ✅ | ✅ |
| Supports top-level `let` bindings | ✅ | ❌ |
| Typical use | Grouping related functions | Organizing large codebases |

```fsharp
namespace MyProject.Domain

type Order = { Id: int; Total: float }

module OrderLogic =
    let isValid order = order.Total > 0.0
```

### 7.5 Package Management

F# uses **NuGet** (via `dotnet` CLI) or **Paket** for dependencies.

```bash
# NuGet (standard)
dotnet add package FSharp.Data

# Paket (alternative, popular in F# community)
dotnet tool install paket
paket init
paket add FSharp.Data
```

### 7.6 Solutions (`.sln`)

Group multiple related projects (app + tests + libraries).

```bash
dotnet new sln -n MySolution
dotnet sln add src/MyProject/MyProject.fsproj
dotnet sln add tests/MyProject.Tests/MyProject.Tests.fsproj
```

---

## 8. Advanced Data Structures (Dynamic Programming)

### 8.1 Memoization with `Dictionary`

```fsharp
open System.Collections.Generic

let fibMemo =
    let cache = Dictionary<int, int64>()
    let rec fib n =
        match cache.TryGetValue n with
        | true, v -> v
        | false, _ ->
            let result =
                if n <= 1 then int64 n
                else fib (n - 1) + fib (n - 2)
            cache.[n] <- result
            result
    fib

printfn "%d" (fibMemo 40)
```

### 8.2 2D Arrays / Grids

Used for classic DP problems (knapsack, edit distance, grid paths).

```fsharp
// Create a 2D DP table, rows x cols, initialized to 0
let dp = Array2D.create (rows + 1) (cols + 1) 0

dp.[0, 0] <- 1
for i in 1 .. rows do
    for j in 1 .. cols do
        dp.[i, j] <- dp.[i-1, j] + dp.[i, j-1]

let value = dp.[rows, cols]

// Jagged arrays (array of arrays) — useful when row lengths vary
let jagged : int[][] = Array.init 5 (fun i -> Array.zeroCreate (i + 1))
```

### 8.3 Mutable Collections (`ResizeArray`, `Dictionary`, `HashSet`)

For performance-sensitive DP/graph code, F# uses the .NET mutable collections directly.

| F# Alias | .NET Type | Use Case |
|---|---|---|
| `ResizeArray<'T>` | `List<'T>` | Dynamic array, O(1) amortized append |
| `Dictionary<'K,'V>` | `Dictionary<'K,'V>` | Hash map, O(1) lookup |
| `HashSet<'T>` | `HashSet<'T>` | Unique elements, O(1) membership check |
| `Queue<'T>` | `Queue<'T>` | FIFO, used in BFS |
| `Stack<'T>` | `Stack<'T>` | LIFO, used in DFS |

```fsharp
let visited = HashSet<int>()
visited.Add(1) |> ignore

let queue = Queue<int>()
queue.Enqueue(1)
let node = queue.Dequeue()

let list = ResizeArray<int>()
list.Add(10)
list.[0] <- 20
```

### 8.4 Priority Queues & Heaps

Useful for Dijkstra's algorithm, top-K problems.

```fsharp
open System.Collections.Generic

// .NET's built-in PriorityQueue<TElement, TPriority> (available in .NET 6+)
let pq = PriorityQueue<string, int>()
pq.Enqueue("task-low", 5)
pq.Enqueue("task-high", 1)
let next = pq.Dequeue()   // "task-high" (lowest priority number first)
```

### 8.5 Trees & Graphs

Recursive discriminated unions model trees elegantly and immutably.

```fsharp
// Binary tree
type Tree<'T> =
    | Leaf
    | Node of Tree<'T> * 'T * Tree<'T>

let rec insert value tree =
    match tree with
    | Leaf -> Node (Leaf, value, Leaf)
    | Node (left, v, right) ->
        if value < v then Node (insert value left, v, right)
        else Node (left, v, insert value right)

let rec sumTree tree =
    match tree with
    | Leaf -> 0
    | Node (l, v, r) -> sumTree l + v + sumTree r

// Graph as adjacency list (mutable, using Dictionary)
let graph = Dictionary<int, ResizeArray<int>>()
let addEdge a b =
    if not (graph.ContainsKey a) then graph.[a] <- ResizeArray<int>()
    graph.[a].Add(b)
```

### 8.6 Tail Recursion

F# optimizes tail-recursive calls into loops — important for deep DP recursion without stack overflows.

```fsharp
// NOT tail-recursive (accumulates work after the recursive call)
let rec sumNaive n =
    if n = 0 then 0 else n + sumNaive (n - 1)

// Tail-recursive version using an accumulator
let sumTailRec n =
    let rec loop acc n =
        if n = 0 then acc
        else loop (acc + n) (n - 1)   // Recursive call is the LAST operation
    loop 0 n
```

| Concept | Naive Recursion | Tail Recursion |
|---|---|---|
| Stack usage | Grows with each call | Constant (optimized to a loop) |
| Risk | Stack overflow on large `n` | Safe for large `n` |
| Pattern | `f(n) = n + f(n-1)` | `f(n, acc) = f(n-1, acc+n)` |

---

### Quick Reference: Common Standard Module Functions

| Module | Common Functions |
|---|---|
| `List` | `map`, `filter`, `fold`, `collect`, `sortBy`, `groupBy`, `iter`, `rev`, `zip` |
| `Array` | `map`, `filter`, `fold`, `sortInPlace`, `iter`, `Parallel.map` |
| `Seq` | `map`, `filter`, `take`, `skip`, `windowed`, `pairwise`, `distinct` |
| `Option` | `map`, `bind`, `defaultValue`, `isSome`, `isNone` |
| `Map` | `add`, `remove`, `tryFind`, `containsKey`, `iter` |
| `String` | `Split`, `Trim`, `Contains`, `sprintf`/`printfn` formatting |

---

*Happy coding! 🐫 (the F# mascot)*