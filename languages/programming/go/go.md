# Golang Concepts Cheat Sheet

A practical, syntax-first reference to core Go concepts — from basic types to project structure and data structures used in competitive programming / DP.

## Table of Contents

- [1. Data Types and Structures](#1-data-types-and-structures)
  - [1.1 Basic Types](#11-basic-types)
  - [1.2 Variables and Constants](#12-variables-and-constants)
  - [1.3 Arrays and Slices](#13-arrays-and-slices)
  - [1.4 Maps](#14-maps)
  - [1.5 Structs](#15-structs)
  - [1.6 Pointers](#16-pointers)
- [2. Object-Oriented Programming in Go](#2-object-oriented-programming-in-go)
  - [2.1 Methods](#21-methods)
  - [2.2 Interfaces](#22-interfaces)
  - [2.3 Embedding (Composition over Inheritance)](#23-embedding-composition-over-inheritance)
  - [2.4 Encapsulation](#24-encapsulation)
- [3. Go Language-Specific Features](#3-go-language-specific-features)
  - [3.1 Error Handling](#31-error-handling)
  - [3.2 Defer, Panic, Recover](#32-defer-panic-recover)
  - [3.3 Goroutines and Channels](#33-goroutines-and-channels)
  - [3.4 Select Statement](#34-select-statement)
  - [3.5 Generics](#35-generics)
  - [3.6 Control Flow Quirks](#36-control-flow-quirks)
- [4. Managing a Go Project](#4-managing-a-go-project)
  - [4.1 Go Modules](#41-go-modules)
  - [4.2 Project Layout](#42-project-layout)
  - [4.3 Importing Local Packages](#43-importing-local-packages)
  - [4.4 Building and Running](#44-building-and-running)
- [5. Advanced Data Structures for Dynamic Programming](#5-advanced-data-structures-for-dynamic-programming)
  - [5.1 Slices as Stacks/Queues](#51-slices-as-stacksqueues)
  - [5.2 Maps for Memoization](#52-maps-for-memoization)
  - [5.3 2D Slices (Grids/DP Tables)](#53-2d-slices-gridsdp-tables)
  - [5.4 container/heap (Priority Queue)](#54-containerheap-priority-queue)
  - [5.5 container/list (Doubly Linked List)](#55-containerlist-doubly-linked-list)
  - [5.6 Sets via `map[T]struct{}`](#56-sets-via-maptstruct)

---

## 1. Data Types and Structures

### 1.1 Basic Types

| Category  | Types                                                     |
|-----------|------------------------------------------------------------|
| Boolean   | `bool`                                                      |
| Integer   | `int`, `int8/16/32/64`, `uint`, `uint8/16/32/64`, `uintptr` |
| Float     | `float32`, `float64`                                        |
| Complex   | `complex64`, `complex128`                                   |
| String    | `string`                                                     |
| Byte/Rune | `byte` (alias for `uint8`), `rune` (alias for `int32`)      |

```go
var isReady bool = true
var age int = 30
var price float64 = 19.99
var name string = "Gopher"
var initial byte = 'A'
var symbol rune = '⌘'
```

### 1.2 Variables and Constants

```go
// Explicit declaration
var x int = 10

// Type inferred
var y = 20

// Short declaration (function scope only)
z := 30

// Multiple declarations
var a, b, c = 1, "two", 3.0

// Constants
const Pi = 3.14159

// iota — auto-incrementing constant generator
const (
    Red = iota // 0
    Green      // 1
    Blue       // 2
)
```

### 1.3 Arrays and Slices

| Feature       | Array                          | Slice                                  |
|---------------|---------------------------------|-----------------------------------------|
| Size          | Fixed at compile time            | Dynamic, can grow                       |
| Declaration   | `var a [5]int`                   | `var s []int`                           |
| Underlying    | Value type                       | Reference to underlying array           |
| Common use    | Rare, low-level                  | Standard for most collections           |

```go
// Array
var arr [5]int = [5]int{1, 2, 3, 4, 5}

// Slice
nums := []int{1, 2, 3}
nums = append(nums, 4, 5)          // append
sub := nums[1:3]                    // slicing: [2, 3]
length := len(nums)
capacity := cap(nums)

// make() for slices
s := make([]int, 3, 10) // len=3, cap=10

// copy
dst := make([]int, len(nums))
copy(dst, nums)
```

### 1.4 Maps

```go
// Declaration and initialization
m := make(map[string]int)
m["apple"] = 5
m["banana"] = 3

// Literal
scores := map[string]int{"Alice": 90, "Bob": 85}

// Lookup with existence check
value, ok := m["apple"]
if ok {
    fmt.Println(value)
}

// Delete
delete(m, "apple")

// Iterate (order not guaranteed)
for key, val := range scores {
    fmt.Println(key, val)
}
```

### 1.5 Structs

```go
type Person struct {
    Name string
    Age  int
}

// Creating instances
p1 := Person{Name: "Alice", Age: 30}
p2 := Person{"Bob", 25} // positional
p3 := new(Person)       // pointer to zero-value struct

// Nested structs
type Address struct {
    City string
}
type Employee struct {
    Person
    Address
    Salary float64
}
```

### 1.6 Pointers

```go
x := 10
p := &x        // p holds address of x
fmt.Println(*p) // dereference: 10
*p = 20         // modify x through pointer

// Pointers with structs
person := &Person{Name: "Alice"}
person.Age = 30 // Go auto-dereferences for field access
```

---

## 2. Object-Oriented Programming in Go

> Go has no classes or inheritance. OOP-like behavior is achieved through **structs**, **methods**, and **interfaces**.

### 2.1 Methods

```go
type Rectangle struct {
    Width, Height float64
}

// Value receiver — operates on a copy
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Pointer receiver — can mutate the original
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

rect := Rectangle{Width: 10, Height: 5}
fmt.Println(rect.Area()) // 50
rect.Scale(2)             // Width=20, Height=10
```

| Receiver Type | Use When                                                |
|----------------|-----------------------------------------------------------|
| Value (`T`)     | Struct is small, no mutation needed                       |
| Pointer (`*T`)  | Need to mutate, or struct is large (avoid copying)         |

### 2.2 Interfaces

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64      { return 3.14159 * c.Radius * c.Radius }
func (c Circle) Perimeter() float64 { return 2 * 3.14159 * c.Radius }

// Any type implementing Area() and Perimeter() satisfies Shape — implicitly!
func Describe(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

// Empty interface — holds any type
var anything interface{} = 42

// Type assertion
if c, ok := anything.(Circle); ok {
    fmt.Println(c.Radius)
}

// Type switch
switch v := anything.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
default:
    fmt.Println("unknown type")
}
```

### 2.3 Embedding (Composition over Inheritance)

```go
type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " makes a sound"
}

type Dog struct {
    Animal // embedded struct — "inherits" fields & methods
    Breed  string
}

d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Labrador"}
fmt.Println(d.Speak()) // "Rex makes a sound" — promoted method
fmt.Println(d.Name)    // promoted field
```

### 2.4 Encapsulation

Go uses **capitalization**, not keywords, to control visibility:

| Name Starts With | Visibility                          |
|--------------------|---------------------------------------|
| Uppercase (`Name`)  | Exported (public, accessible outside package) |
| Lowercase (`name`)  | Unexported (private to the package)   |

```go
package account

type Account struct {
    Balance float64 // exported
    pin     int     // unexported
}
```

---

## 3. Go Language-Specific Features

### 3.1 Error Handling

```go
import "errors"

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
    return
}

// Wrapping errors (Go 1.13+)
wrapped := fmt.Errorf("operation failed: %w", err)

// Custom error types
type ValidationError struct {
    Field string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("invalid field: %s", e.Field)
}
```

### 3.2 Defer, Panic, Recover

```go
func process() {
    defer fmt.Println("cleanup runs last") // executes when function returns
    defer fmt.Println("this runs before cleanup")

    defer func() {
        if r := recover(); r != nil {
            fmt.Println("recovered from:", r)
        }
    }()

    panic("something went wrong")
}
```

- `defer` — schedules a call to run when the surrounding function returns (LIFO order).
- `panic` — stops normal execution.
- `recover` — regains control after a panic (must be called inside a `defer`).

### 3.3 Goroutines and Channels

```go
// Goroutine — lightweight thread
go func() {
    fmt.Println("running concurrently")
}()

// Channels — typed conduits for communication
ch := make(chan int)

go func() {
    ch <- 42 // send
}()
value := <-ch // receive

// Buffered channel
buffered := make(chan string, 3)
buffered <- "a"
buffered <- "b"

// WaitGroup — wait for goroutines to finish
var wg sync.WaitGroup
for i := 0; i < 3; i++ {
    wg.Add(1)
    go func(id int) {
        defer wg.Done()
        fmt.Println("worker", id)
    }(i)
}
wg.Wait()

// Closing channels
close(ch)
val, ok := <-ch // ok is false if channel is closed and drained
```

### 3.4 Select Statement

```go
select {
case msg1 := <-ch1:
    fmt.Println("received", msg1)
case msg2 := <-ch2:
    fmt.Println("received", msg2)
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("no channel ready")
}
```

### 3.5 Generics

```go
// Generic function (Go 1.18+)
func Max[T int | float64](a, b T) T {
    if a > b {
        return a
    }
    return b
}

fmt.Println(Max(3, 7))       // works with int
fmt.Println(Max(3.5, 2.1))   // works with float64

// Generic type constraints
type Number interface {
    int | int64 | float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}
```

### 3.6 Control Flow Quirks

```go
// No while loop — for handles everything
for i := 0; i < 5; i++ { }   // classic
for condition { }             // while-style
for { }                       // infinite loop

// if with initializer
if val, err := someFunc(); err == nil {
    fmt.Println(val)
}

// switch without expression (like if-else chain)
switch {
case age < 13:
    fmt.Println("child")
case age < 20:
    fmt.Println("teen")
default:
    fmt.Println("adult")
}

// Labeled break/continue
outer:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if j == 1 {
            continue outer
        }
    }
}
```

---

## 4. Managing a Go Project

### 4.1 Go Modules

```bash
# Initialize a module (creates go.mod)
go mod init github.com/username/projectname

# Add/update dependencies used in code
go mod tidy

# Download dependencies
go mod download

# View dependency graph
go mod graph
```

`go.mod` example:

```
module github.com/username/projectname

go 1.22

require (
    github.com/gin-gonic/gin v1.9.1
)
```

### 4.2 Project Layout

A conventional Go project structure:

```
projectname/
├── go.mod
├── go.sum
├── main.go
├── cmd/
│   └── app/
│       └── main.go
├── internal/
│   ├── service/
│   │   └── service.go
│   └── models/
│       └── user.go
├── pkg/
│   └── utils/
│       └── helpers.go
└── config/
    └── config.go
```

| Directory     | Purpose                                                |
|----------------|-----------------------------------------------------------|
| `cmd/`          | Entry points for executables (one subfolder per binary)   |
| `internal/`     | Private packages — cannot be imported by external projects |
| `pkg/`          | Public/shared packages other projects may import          |
| `config/`       | Configuration loading logic                               |

### 4.3 Importing Local Packages

Given `module github.com/username/projectname` in `go.mod`, and a package at `internal/models/user.go`:

```go
// internal/models/user.go
package models

type User struct {
    ID   int
    Name string
}
```

Import it elsewhere in the project using the module path:

```go
// main.go
package main

import (
    "fmt"
    "github.com/username/projectname/internal/models"
)

func main() {
    u := models.User{ID: 1, Name: "Alice"}
    fmt.Println(u)
}
```

> The import path is always `<module name>/<folder path>` — not a relative path like `./models`.

### 4.4 Building and Running

| Command                          | Purpose                                          |
|------------------------------------|-----------------------------------------------------|
| `go run main.go`                   | Compile and run immediately (no binary saved)       |
| `go build`                         | Compile into a binary in the current directory      |
| `go build -o bin/app ./cmd/app`     | Build with custom output path                       |
| `go install`                       | Build and install binary to `$GOPATH/bin`           |
| `go test ./...`                    | Run all tests recursively                            |
| `go vet ./...`                     | Static analysis for suspicious code                  |
| `go fmt ./...`                     | Auto-format code per Go conventions                  |

```bash
# Typical workflow
go mod init github.com/username/projectname
go build -o bin/app ./cmd/app
./bin/app
```

---

## 5. Advanced Data Structures for Dynamic Programming

### 5.1 Slices as Stacks/Queues

```go
// Stack (LIFO)
stack := []int{}
stack = append(stack, 1) // push
top := stack[len(stack)-1]
stack = stack[:len(stack)-1] // pop

// Queue (FIFO)
queue := []int{}
queue = append(queue, 1) // enqueue
front := queue[0]
queue = queue[1:] // dequeue
```

### 5.2 Maps for Memoization

```go
// Classic top-down DP with memoization
func fib(n int, memo map[int]int) int {
    if n <= 1 {
        return n
    }
    if val, ok := memo[n]; ok {
        return val
    }
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
}

result := fib(30, make(map[int]int))
```

### 5.3 2D Slices (Grids/DP Tables)

```go
// Building an m x n DP table
rows, cols := 5, 5
dp := make([][]int, rows)
for i := range dp {
    dp[i] = make([]int, cols)
}

// Example: unique paths DP
for i := 0; i < rows; i++ {
    for j := 0; j < cols; j++ {
        if i == 0 || j == 0 {
            dp[i][j] = 1
        } else {
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
        }
    }
}
```

### 5.4 container/heap (Priority Queue)

```go
import "container/heap"

type IntHeap []int

func (h IntHeap) Len() int            { return len(h) }
func (h IntHeap) Less(i, j int) bool  { return h[i] < h[j] } // min-heap
func (h IntHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *IntHeap) Push(x interface{}) { *h = append(*h, x.(int)) }
func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    item := old[n-1]
    *h = old[:n-1]
    return item
}

h := &IntHeap{5, 2, 8, 1}
heap.Init(h)
heap.Push(h, 3)
smallest := heap.Pop(h).(int)
```

### 5.5 container/list (Doubly Linked List)

```go
import "container/list"

l := list.New()
l.PushBack(1)
l.PushBack(2)
l.PushFront(0)

for e := l.Front(); e != nil; e = e.Next() {
    fmt.Println(e.Value)
}
```

### 5.6 Sets via `map[T]struct{}`

```go
// Idiomatic Go "set" — struct{} takes zero bytes
set := make(map[string]struct{})
set["apple"] = struct{}{}
set["banana"] = struct{}{}

// Check membership
if _, exists := set["apple"]; exists {
    fmt.Println("apple is in the set")
}

// Delete
delete(set, "apple")
```

| Structure                     | Best For                                      |
|----------------------------------|--------------------------------------------------|
| Slice                             | Ordered sequences, stacks, queues                 |
| Map                               | Memoization, lookups, frequency counting          |
| 2D Slice                          | Grid-based DP (paths, edit distance, knapsack)     |
| `container/heap`                  | Priority queues (Dijkstra, top-K problems)         |
| `container/list`                  | LRU caches, frequent insert/delete at both ends    |
| `map[T]struct{}`                  | Set membership, visited-node tracking (graphs/DP)  |

---

*Happy coding! 🐹*