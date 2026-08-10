# C# Complete Concepts Guide

A quick-reference guide covering C# fundamentals, OOP, language features, project management, and advanced data structures for problem solving.

## Table of Contents

- [1. Data Types and Structures](#1-data-types-and-structures)
  - [1.1 Value Types](#11-value-types)
  - [1.2 Reference Types](#12-reference-types)
  - [1.3 Variables and Type Inference](#13-variables-and-type-inference)
  - [1.4 Nullable Types](#14-nullable-types)
  - [1.5 Arrays](#15-arrays)
- [2. Operators and Control Flow](#2-operators-and-control-flow)
  - [2.1 Operators](#21-operators)
  - [2.2 Conditionals](#22-conditionals)
  - [2.3 Loops](#23-loops)
- [3. Object-Oriented Programming](#3-object-oriented-programming)
  - [3.1 Classes and Objects](#31-classes-and-objects)
  - [3.2 Constructors](#32-constructors)
  - [3.3 Properties and Fields](#33-properties-and-fields)
  - [3.4 Access Modifiers](#34-access-modifiers)
  - [3.5 Inheritance](#35-inheritance)
  - [3.6 Polymorphism](#36-polymorphism)
  - [3.7 Abstraction (Abstract Classes & Interfaces)](#37-abstraction-abstract-classes--interfaces)
  - [3.8 Structs vs Classes](#38-structs-vs-classes)
- [4. C# Language-Specific Features](#4-c-language-specific-features)
  - [4.1 Generics](#41-generics)
  - [4.2 Delegates, Events, and Lambdas](#42-delegates-events-and-lambdas)
  - [4.3 LINQ](#43-linq)
  - [4.4 Exception Handling](#44-exception-handling)
  - [4.5 Async / Await](#45-async--await)
  - [4.6 Pattern Matching](#46-pattern-matching)
  - [4.7 Records](#47-records)
  - [4.8 Extension Methods](#48-extension-methods)
- [5. Managing a C# Project](#5-managing-a-c-project)
  - [5.1 Solution and Project Structure](#51-solution-and-project-structure)
  - [5.2 Namespaces and `using`](#52-namespaces-and-using)
  - [5.3 Referencing Other Projects/Files](#53-referencing-other-projectsfiles)
  - [5.4 NuGet Packages](#54-nuget-packages)
  - [5.5 Common CLI Commands](#55-common-cli-commands)
- [6. Advanced Data Structures for Dynamic Programming](#6-advanced-data-structures-for-dynamic-programming)
  - [6.1 Collections Overview](#61-collections-overview)
  - [6.2 Dictionary (Hash Map) for Memoization](#62-dictionary-hash-map-for-memoization)
  - [6.3 Stack and Queue](#63-stack-and-queue)
  - [6.4 2D Arrays / Tabulation](#64-2d-arrays--tabulation)
  - [6.5 PriorityQueue (Heap)](#65-priorityqueue-heap)
  - [6.6 HashSet](#66-hashset)

---

## 1. Data Types and Structures

### 1.1 Value Types

Value types store data directly and live on the stack (in most cases). Copying a value type copies the data.

| Type | Description | Size | Example |
|------|-------------|------|---------|
| `int` | 32-bit signed integer | 4 bytes | `int age = 25;` |
| `long` | 64-bit signed integer | 8 bytes | `long big = 9000000000L;` |
| `short` | 16-bit signed integer | 2 bytes | `short s = 100;` |
| `byte` | 8-bit unsigned integer | 1 byte | `byte b = 255;` |
| `float` | 32-bit floating point | 4 bytes | `float f = 3.14f;` |
| `double` | 64-bit floating point | 8 bytes | `double d = 3.14159;` |
| `decimal` | High-precision decimal (finance) | 16 bytes | `decimal price = 19.99m;` |
| `bool` | Boolean value | 1 byte | `bool isActive = true;` |
| `char` | Single UTF-16 character | 2 bytes | `char c = 'A';` |
| `struct` | Custom value type | varies | `struct Point { }` |
| `enum` | Named constant set | 4 bytes | `enum Color { Red, Green }` |

```csharp
int number = 42;
double pi = 3.14159;
bool isValid = true;
char letter = 'A';
```

### 1.2 Reference Types

Reference types store a pointer to the data on the heap. Copying a reference copies the pointer, not the underlying data.

| Type | Description | Example |
|------|-------------|---------|
| `class` | Custom reference type | `class Person { }` |
| `string` | Immutable sequence of characters | `string name = "Alice";` |
| `object` | Base type of all types | `object o = 5;` |
| `array` | Fixed-size collection | `int[] nums = {1, 2, 3};` |
| `interface` | Contract/blueprint | `interface IShape { }` |
| `delegate` | Type-safe function pointer | `delegate void Handler();` |

```csharp
string message = "Hello, World!";
object anything = 42; // boxing
int[] numbers = { 1, 2, 3, 4 };
```

### 1.3 Variables and Type Inference

```csharp
var name = "Alice";        // compiler infers string
int age = 30;               // explicit type
const double Pi = 3.14159;  // compile-time constant
readonly int Id;            // set once, at runtime (in constructor)
```

| Keyword | Meaning |
|---------|---------|
| `var` | Implicitly typed local variable (type inferred at compile time) |
| `const` | Compile-time constant, must be initialized at declaration |
| `readonly` | Runtime constant, can be set in constructor |
| `static` | Belongs to the type itself, not an instance |

### 1.4 Nullable Types

```csharp
int? age = null;              // nullable value type
string? name = null;          // nullable reference type (C# 8+)

if (age.HasValue) { Console.WriteLine(age.Value); }
int ageOrDefault = age ?? 0;  // null-coalescing operator
age ??= 18;                   // null-coalescing assignment
```

### 1.5 Arrays

```csharp
int[] arr = new int[5];             // fixed-size array
int[] arr2 = { 1, 2, 3, 4, 5 };     // array initializer
int[,] matrix = new int[3, 3];      // 2D rectangular array
int[][] jagged = new int[3][];      // jagged array (array of arrays)

Console.WriteLine(arr2.Length);     // 5
Array.Sort(arr2);
Array.Reverse(arr2);
```

---

## 2. Operators and Control Flow

### 2.1 Operators

| Category | Operators |
|----------|-----------|
| Arithmetic | `+  -  *  /  %  ++  --` |
| Comparison | `==  !=  >  <  >=  <=` |
| Logical | `&&  \|\|  !` |
| Assignment | `=  +=  -=  *=  /=  %=` |
| Null-related | `??  ??=  ?.` |
| Type checking | `is  as  typeof` |
| Ternary | `condition ? a : b` |

```csharp
int x = 10;
string result = x > 5 ? "big" : "small";
Person? p = obj as Person;   // safe cast, null if fails
p?.SayHello();                // null-conditional call
```

### 2.2 Conditionals

```csharp
if (x > 10)
{
    Console.WriteLine("Greater than 10");
}
else if (x == 10)
{
    Console.WriteLine("Equal to 10");
}
else
{
    Console.WriteLine("Less than 10");
}

switch (x)
{
    case 1:
        Console.WriteLine("One");
        break;
    case 2:
    case 3:
        Console.WriteLine("Two or Three");
        break;
    default:
        Console.WriteLine("Other");
        break;
}

// Switch expression (C# 8+)
string category = x switch
{
    < 0 => "negative",
    0 => "zero",
    > 0 => "positive",
};
```

### 2.3 Loops

```csharp
for (int i = 0; i < 10; i++) { }

foreach (var item in collection) { }

int i = 0;
while (i < 10) { i++; }

do
{
    i++;
} while (i < 10);
```

---

## 3. Object-Oriented Programming

### 3.1 Classes and Objects

```csharp
public class Person
{
    public string Name;
    public int Age;

    public void Greet()
    {
        Console.WriteLine($"Hi, I'm {Name}");
    }
}

// Usage
Person p = new Person();
p.Name = "Alice";
p.Greet();
```

### 3.2 Constructors

```csharp
public class Person
{
    public string Name;
    public int Age;

    // Default constructor
    public Person() { }

    // Parameterized constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

// Primary constructor (C# 12+)
public class Person(string name, int age)
{
    public string Name { get; } = name;
    public int Age { get; } = age;
}
```

### 3.3 Properties and Fields

```csharp
public class Person
{
    private string _name;           // private field

    public string Name              // auto-implemented property
    {
        get { return _name; }
        set { _name = value; }
    }

    public int Age { get; set; }             // auto-property
    public string Email { get; init; }       // settable only at init (C# 9+)
    public int Id { get; } = 1;              // read-only property
}
```

### 3.4 Access Modifiers

| Modifier | Accessible From |
|----------|-----------------|
| `public` | Anywhere |
| `private` | Same class only |
| `protected` | Same class and derived classes |
| `internal` | Same assembly/project |
| `protected internal` | Same assembly OR derived classes |
| `private protected` | Same assembly AND derived classes |

### 3.5 Inheritance

```csharp
public class Animal
{
    public string Name;
    public virtual void Speak() => Console.WriteLine("...");
}

public class Dog : Animal
{
    public override void Speak() => Console.WriteLine("Woof!");
}
```

### 3.6 Polymorphism

```csharp
Animal a = new Dog();
a.Speak(); // "Woof!" — runtime picks the overridden method

// Method overloading (compile-time polymorphism)
public int Add(int a, int b) => a + b;
public double Add(double a, double b) => a + b;
```

### 3.7 Abstraction (Abstract Classes & Interfaces)

```csharp
public abstract class Shape
{
    public abstract double Area();          // must be implemented
    public void Describe() => Console.WriteLine("A shape"); // shared logic
}

public class Circle : Shape
{
    public double Radius;
    public override double Area() => Math.PI * Radius * Radius;
}

public interface IMovable
{
    void Move(int x, int y);
}

public class Robot : IMovable
{
    public void Move(int x, int y) => Console.WriteLine($"Moving to {x},{y}");
}
```

| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| Multiple inheritance | No (single) | Yes (multiple) |
| Can have fields | Yes | No |
| Can have method bodies | Yes | Yes (default impl, C# 8+) |
| Constructors | Yes | No |

### 3.8 Structs vs Classes

```csharp
public struct Point   // value type
{
    public int X, Y;
}

public class Point3D   // reference type
{
    public int X, Y, Z;
}
```

| | `struct` | `class` |
|---|---------|---------|
| Type | Value type | Reference type |
| Storage | Stack (usually) | Heap |
| Inheritance | No | Yes |
| Default choice | Small, immutable data | Most other objects |

---

## 4. C# Language-Specific Features

### 4.1 Generics

```csharp
public class Box<T>
{
    public T Value;
    public Box(T value) => Value = value;
}

public T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) > 0 ? a : b;
}

Box<int> intBox = new Box<int>(5);
Box<string> strBox = new Box<string>("hello");
```

### 4.2 Delegates, Events, and Lambdas

```csharp
public delegate int MathOp(int a, int b);

MathOp add = (a, b) => a + b;   // lambda expression
Console.WriteLine(add(2, 3));   // 5

// Built-in delegate types
Func<int, int, int> multiply = (a, b) => a * b;
Action<string> print = msg => Console.WriteLine(msg);
Predicate<int> isEven = n => n % 2 == 0;

// Events
public class Button
{
    public event Action? Clicked;
    public void Click() => Clicked?.Invoke();
}
```

### 4.3 LINQ

```csharp
using System.Linq;

int[] numbers = { 1, 2, 3, 4, 5, 6 };

var evens = numbers.Where(n => n % 2 == 0).ToList();
var doubled = numbers.Select(n => n * 2).ToList();
var sum = numbers.Sum();
var sorted = numbers.OrderByDescending(n => n).ToList();
var first = numbers.FirstOrDefault(n => n > 4);

// Query syntax
var query = from n in numbers
            where n > 2
            select n * n;
```

| Method | Purpose |
|--------|---------|
| `Where` | Filter elements |
| `Select` | Transform/project elements |
| `OrderBy` / `OrderByDescending` | Sort |
| `GroupBy` | Group elements by key |
| `FirstOrDefault` / `SingleOrDefault` | Get one element safely |
| `Any` / `All` | Boolean checks |
| `Sum` / `Min` / `Max` / `Average` | Aggregation |

### 4.4 Exception Handling

```csharp
try
{
    int result = 10 / int.Parse("0");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
catch (Exception ex) when (ex.Message.Contains("format"))
{
    Console.WriteLine("Format issue");
}
finally
{
    Console.WriteLine("Always runs");
}

throw new ArgumentException("Invalid input");
```

### 4.5 Async / Await

```csharp
public async Task<string> FetchDataAsync()
{
    await Task.Delay(1000);          // simulate I/O
    return "data";
}

public async Task RunAsync()
{
    string data = await FetchDataAsync();
    Console.WriteLine(data);
}
```

### 4.6 Pattern Matching

```csharp
object obj = 5;

if (obj is int number)
    Console.WriteLine($"It's an int: {number}");

string Describe(object o) => o switch
{
    int n when n < 0 => "negative int",
    int n => $"int: {n}",
    string s => $"string: {s}",
    null => "nothing",
    _ => "unknown"
};

// Record deconstruction / property patterns
if (person is { Age: > 18 } adult)
    Console.WriteLine("Adult");
```

### 4.7 Records

```csharp
// Immutable reference type with value-based equality (C# 9+)
public record Person(string Name, int Age);

var p1 = new Person("Alice", 30);
var p2 = p1 with { Age = 31 };   // non-destructive mutation
Console.WriteLine(p1 == p2);     // false, compares values

public record struct Point(int X, int Y); // value-type record
```

### 4.8 Extension Methods

```csharp
public static class StringExtensions
{
    public static bool IsPalindrome(this string s)
    {
        var reversed = new string(s.Reverse().ToArray());
        return s == reversed;
    }
}

// Usage
bool result = "level".IsPalindrome(); // true
```

---

## 5. Managing a C# Project

### 5.1 Solution and Project Structure

```text
MySolution.sln
├── MyApp/
│   ├── MyApp.csproj
│   ├── Program.cs
│   ├── Models/
│   │   └── Person.cs
│   └── Services/
│       └── UserService.cs
└── MyApp.Tests/
    ├── MyApp.Tests.csproj
    └── UserServiceTests.cs
```

- **Solution (`.sln`)** — groups multiple related projects.
- **Project (`.csproj`)** — defines a single buildable unit (app, library, test project).

### 5.2 Namespaces and `using`

```csharp
// File: Models/Person.cs
namespace MyApp.Models
{
    public class Person { public string Name; }
}

// File-scoped namespace (C# 10+, less nesting)
namespace MyApp.Models;
public class Person { public string Name; }
```

```csharp
// File: Program.cs
using MyApp.Models;   // import the namespace, not a file path

var p = new Person();
```

> **Key idea:** In C#, you `using` a **namespace**, not a file. As long as your file's `namespace` is referenced with `using`, and the project references the file's assembly/project, the class is accessible — no manual "import path" like in Python or JS.

### 5.3 Referencing Other Projects/Files

Files within the **same project** are automatically compiled together — no import needed beyond `using <Namespace>`.

To use another **project** (e.g. a class library) from your main app:

```bash
dotnet add MyApp/MyApp.csproj reference MyLibrary/MyLibrary.csproj
```

This adds to `MyApp.csproj`:

```xml
<ItemGroup>
  <ProjectReference Include="..\MyLibrary\MyLibrary.csproj" />
</ItemGroup>
```

Then in code:

```csharp
using MyLibrary.Utilities;
```

### 5.4 NuGet Packages

```bash
dotnet add package Newtonsoft.Json
dotnet add package Microsoft.EntityFrameworkCore --version 8.0.0
dotnet restore
```

```csharp
using Newtonsoft.Json;

string json = JsonConvert.SerializeObject(myObject);
```

### 5.5 Common CLI Commands

| Command | Purpose |
|---------|---------|
| `dotnet new console -n MyApp` | Create a new console project |
| `dotnet new classlib -n MyLib` | Create a new class library |
| `dotnet new sln` | Create a new solution file |
| `dotnet sln add MyApp/MyApp.csproj` | Add project to solution |
| `dotnet build` | Build the project |
| `dotnet run` | Run the project |
| `dotnet test` | Run unit tests |
| `dotnet add package <name>` | Install a NuGet package |
| `dotnet restore` | Restore dependencies |

---

## 6. Advanced Data Structures for Dynamic Programming

### 6.1 Collections Overview

| Structure | Namespace | Use Case | Big-O Access |
|-----------|-----------|----------|--------------|
| `List<T>` | `System.Collections.Generic` | Dynamic array | O(1) index, O(n) search |
| `Dictionary<K,V>` | `System.Collections.Generic` | Hash map / memoization | O(1) avg lookup |
| `HashSet<T>` | `System.Collections.Generic` | Unique elements, visited sets | O(1) avg lookup |
| `Stack<T>` | `System.Collections.Generic` | LIFO — backtracking, recursion sim | O(1) push/pop |
| `Queue<T>` | `System.Collections.Generic` | FIFO — BFS | O(1) enqueue/dequeue |
| `PriorityQueue<T,P>` | `System.Collections.Generic` | Min-heap — Dijkstra, greedy | O(log n) insert |
| `LinkedList<T>` | `System.Collections.Generic` | Fast insert/remove mid-list | O(1) insert/remove at node |
| `SortedSet<T>` / `SortedDictionary<K,V>` | `System.Collections.Generic` | Ordered unique keys | O(log n) |

### 6.2 Dictionary (Hash Map) for Memoization

```csharp
using System.Collections.Generic;

Dictionary<int, long> memo = new();

long Fibonacci(int n)
{
    if (n <= 1) return n;
    if (memo.TryGetValue(n, out long cached)) return cached;

    long result = Fibonacci(n - 1) + Fibonacci(n - 2);
    memo[n] = result;
    return result;
}
```

### 6.3 Stack and Queue

```csharp
// Stack — LIFO
Stack<int> stack = new();
stack.Push(1);
stack.Push(2);
int top = stack.Pop();     // 2
int peek = stack.Peek();   // 1

// Queue — FIFO (great for BFS)
Queue<int> queue = new();
queue.Enqueue(1);
queue.Enqueue(2);
int front = queue.Dequeue(); // 1
```

```csharp
// BFS example using Queue
void BFS(Dictionary<int, List<int>> graph, int start)
{
    var visited = new HashSet<int>();
    var queue = new Queue<int>();
    queue.Enqueue(start);
    visited.Add(start);

    while (queue.Count > 0)
    {
        int node = queue.Dequeue();
        Console.WriteLine(node);

        foreach (int neighbor in graph[node])
        {
            if (visited.Add(neighbor))   // returns false if already present
                queue.Enqueue(neighbor);
        }
    }
}
```

### 6.4 2D Arrays / Tabulation

```csharp
// Classic bottom-up DP: Longest Common Subsequence
int LCS(string a, string b)
{
    int[,] dp = new int[a.Length + 1, b.Length + 1];

    for (int i = 1; i <= a.Length; i++)
    {
        for (int j = 1; j <= b.Length; j++)
        {
            dp[i, j] = a[i - 1] == b[j - 1]
                ? dp[i - 1, j - 1] + 1
                : Math.Max(dp[i - 1, j], dp[i, j - 1]);
        }
    }

    return dp[a.Length, b.Length];
}
```

### 6.5 PriorityQueue (Heap)

```csharp
// Min-heap, built-in since .NET 6
PriorityQueue<string, int> pq = new();
pq.Enqueue("task-low-priority", 5);
pq.Enqueue("task-high-priority", 1);

string next = pq.Dequeue(); // "task-high-priority" (lowest priority value first)
```

### 6.6 HashSet

```csharp
// Track visited states in DP/graph problems
HashSet<(int, int)> visited = new();

if (visited.Add((row, col)))
{
    // wasn't visited before — process it
}

// Set operations
var setA = new HashSet<int> { 1, 2, 3 };
var setB = new HashSet<int> { 2, 3, 4 };
setA.IntersectWith(setB);   // setA becomes {2, 3}
```

---

### Quick Reference: Choosing a Structure for DP

| Problem Pattern | Recommended Structure |
|------------------|------------------------|
| Top-down memoization | `Dictionary<TState, TResult>` |
| Bottom-up tabulation, fixed dimensions | `int[]`, `int[,]`, `int[,,]` |
| Graph traversal (BFS) | `Queue<T>` + `HashSet<T>` (visited) |
| Graph traversal (DFS) / backtracking | `Stack<T>` or recursion |
| Shortest path / greedy selection | `PriorityQueue<T,P>` |
| Deduplication / seen-state tracking | `HashSet<T>` |
| Sliding window / deque problems | `LinkedList<T>` or `Queue<T>` |