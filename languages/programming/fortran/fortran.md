# Modern Fortran Concepts Guide

> Based on **Fortran 2018 / Fortran 2023** syntax (the current ISO standards). Where a feature is 2023-only, it is noted explicitly.

## Table of Contents

- [1. Basics: Program Structure](#1-basics-program-structure)
- [2. Data Types](#2-data-types)
  - [2.1 Intrinsic Types](#21-intrinsic-types)
  - [2.2 Kind Specifiers](#22-kind-specifiers)
  - [2.3 Arrays](#23-arrays)
- [3. Derived Types (Structs)](#3-derived-types-structs)
- [4. Object-Oriented Programming](#4-object-oriented-programming)
  - [4.1 Type-Bound Procedures](#41-type-bound-procedures)
  - [4.2 Inheritance (`extends`)](#42-inheritance-extends)
  - [4.3 Polymorphism (`class`, `select type`)](#43-polymorphism-class-select-type)
  - [4.4 Abstract Types & Interfaces](#44-abstract-types--interfaces)
  - [4.5 Operator Overloading](#45-operator-overloading)
- [5. Control Flow](#5-control-flow)
- [6. Procedures: Subroutines & Functions](#6-procedures-subroutines--functions)
- [7. Fortran-Specific Language Features](#7-fortran-specific-language-features)
  - [7.1 Array Slicing & Whole-Array Ops](#71-array-slicing--whole-array-ops)
  - [7.2 `where` and `forall`](#72-where-and-forall)
  - [7.3 Intent & Argument Passing](#73-intent--argument-passing)
  - [7.4 Interfaces & Generic Procedures](#74-interfaces--generic-procedures)
  - [7.5 Coarrays (Parallel Fortran)](#75-coarrays-parallel-fortran)
  - [7.6 `do concurrent`](#76-do-concurrent)
- [8. Managing a Fortran Project](#8-managing-a-fortran-project)
  - [8.1 Modules & `use`](#81-modules--use)
  - [8.2 Submodules](#82-submodules)
  - [8.3 Project Layout](#83-project-layout)
  - [8.4 Fortran Package Manager (`fpm`)](#84-fortran-package-manager-fpm)
  - [8.5 Compiling Manually](#85-compiling-manually)
- [9. Advanced / Dynamic Data Structures](#9-advanced--dynamic-data-structures)
  - [9.1 `allocatable` Arrays](#91-allocatable-arrays)
  - [9.2 Pointers](#92-pointers)
  - [9.3 Linked Lists](#93-linked-lists)
  - [9.4 Generic (Templated) Containers](#94-generic-templated-containers)
  - [9.5 Dynamic Dispatch Containers](#95-dynamic-dispatch-containers)
- [10. Error Handling](#10-error-handling)
- [11. Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. Basics: Program Structure

Every Fortran program has a `program`/`end program` block. Free-form source (`.f90` and later) is column-independent, unlike old fixed-form `.f`.

```fortran
program hello
    implicit none          ! ALWAYS use this — disables risky implicit typing
    print *, "Hello, Fortran!"
end program hello
```

| Element | Purpose |
|---|---|
| `implicit none` | Forces explicit variable declaration (best practice, always include) |
| `program ... end program` | Main entry point |
| `!` | Comment marker |
| `&` | Line continuation |

```fortran
x = 1 + 2 + &
    3 + 4      ! continuation with &
```

---

## 2. Data Types

### 2.1 Intrinsic Types

| Type | Keyword | Example Declaration |
|---|---|---|
| Integer | `integer` | `integer :: i` |
| Real (floating point) | `real` | `real :: x` |
| Double precision | `real(kind=8)` or `double precision` | `real(kind=8) :: y` |
| Complex | `complex` | `complex :: z` |
| Logical (boolean) | `logical` | `logical :: flag` |
| Character (string) | `character` | `character(len=20) :: name` |

```fortran
integer            :: age = 30
real                :: pi = 3.14159
logical             :: is_valid = .true.
character(len=10)  :: greeting = "Hi there!"
complex             :: c = (1.0, 2.0)
```

### 2.2 Kind Specifiers

Fortran uses **kind** parameters to control precision portably instead of relying on compiler-specific sizes.

```fortran
use, intrinsic :: iso_fortran_env, only: int32, int64, real32, real64

integer(int64) :: big_number
real(real64)   :: precise_value
```

| Old style | Modern (preferred) |
|---|---|
| `real*8` | `real(real64)` from `iso_fortran_env` |
| `integer*4` | `integer(int32)` from `iso_fortran_env` |

### 2.3 Arrays

```fortran
integer :: a(5)                 ! static array, 1 to 5
integer :: b(0:4)               ! custom bounds, 0 to 4
integer :: matrix(3,3)          ! 2D array
integer, allocatable :: c(:)    ! dynamic array (see section 9)
integer :: d(5) = [1,2,3,4,5]   ! array constructor syntax
```

---

## 3. Derived Types (Structs)

Fortran's equivalent of a `struct` / `class` data container.

```fortran
type :: point
    real :: x = 0.0
    real :: y = 0.0
end type point

! Usage
type(point) :: p1
p1 = point(1.0, 2.0)      ! constructor call
print *, p1%x, p1%y       ! member access via %
```

Nested derived types:

```fortran
type :: circle
    type(point) :: center
    real        :: radius
end type circle
```

---

## 4. Object-Oriented Programming

Modern Fortran (2003+) supports OOP: encapsulation, inheritance, polymorphism.

### 4.1 Type-Bound Procedures

```fortran
type :: shape
    real :: area = 0.0
contains
    procedure :: print_area
end type shape

contains

subroutine print_area(self)
    class(shape), intent(in) :: self
    print *, "Area:", self%area
end subroutine print_area
```

Call it like a method: `call my_shape%print_area()`

### 4.2 Inheritance (`extends`)

```fortran
type :: rectangle
    real :: width, height
contains
    procedure :: area => rectangle_area
end type rectangle

type, extends(rectangle) :: square
    ! inherits width, height, and area from rectangle
end type square
```

### 4.3 Polymorphism (`class`, `select type`)

| Keyword | Meaning |
|---|---|
| `type(T)` | Fixed, concrete type — no polymorphism |
| `class(T)` | Polymorphic — can hold `T` or any type that extends `T` |
| `class(*)` | Unlimited polymorphic — any type |

```fortran
subroutine describe(obj)
    class(shape), intent(in) :: obj

    select type (obj)
    type is (rectangle)
        print *, "It's a rectangle"
    type is (square)
        print *, "It's a square"
    class default
        print *, "Unknown shape"
    end select
end subroutine describe
```

### 4.4 Abstract Types & Interfaces

```fortran
type, abstract :: base_shape
contains
    procedure(area_interface), deferred :: area
end type base_shape

abstract interface
    function area_interface(self) result(a)
        import :: base_shape
        class(base_shape), intent(in) :: self
        real :: a
    end function area_interface
end interface
```

### 4.5 Operator Overloading

```fortran
type :: vector2d
    real :: x, y
end type vector2d

interface operator(+)
    module procedure add_vectors
end interface operator(+)

contains

function add_vectors(a, b) result(c)
    type(vector2d), intent(in) :: a, b
    type(vector2d) :: c
    c%x = a%x + b%x
    c%y = a%y + b%y
end function add_vectors
```

---

## 5. Control Flow

| Construct | Syntax |
|---|---|
| If/else | `if (cond) then ... else if (cond2) then ... else ... end if` |
| Select case | `select case (var) ... case (1) ... case default ... end select` |
| Do loop | `do i = 1, 10 ... end do` |
| While-style | `do while (cond) ... end do` |
| Infinite + exit | `do ... if (cond) exit ... end do` |

```fortran
do i = 1, 10, 2          ! start, stop, step
    if (mod(i, 2) == 0) cycle   ! like 'continue'
    if (i > 7) exit              ! break out
    print *, i
end do
```

---

## 6. Procedures: Subroutines & Functions

| | Subroutine | Function |
|---|---|---|
| Returns value? | No (via arguments) | Yes (via `result`) |
| Call syntax | `call my_sub(args)` | `y = my_func(args)` |

```fortran
subroutine greet(name)
    character(len=*), intent(in) :: name
    print *, "Hello, ", name
end subroutine greet

function square(x) result(y)
    real, intent(in) :: x
    real :: y
    y = x * x
end function square
```

`pure` and `elemental` procedures (important for optimization & `do concurrent`):

```fortran
pure elemental function double_it(x) result(y)
    real, intent(in) :: x
    real :: y
    y = 2.0 * x
end function double_it
```

---

## 7. Fortran-Specific Language Features

### 7.1 Array Slicing & Whole-Array Ops

```fortran
integer :: a(10), b(10)
a = 1                      ! sets every element to 1
b(2:5) = a(2:5) * 2        ! slicing
b(:) = a(:) + 1            ! whole array arithmetic, no loop needed
print *, sum(a), maxval(a), minval(a), size(a)
```

### 7.2 `where` and `forall`

Array-aware conditional assignment (vectorized "if"):

```fortran
real :: arr(5) = [1.0, -2.0, 3.0, -4.0, 5.0]

where (arr < 0)
    arr = 0.0
elsewhere
    arr = arr * 2
end where
```

### 7.3 Intent & Argument Passing

| Keyword | Meaning |
|---|---|
| `intent(in)` | Read-only input |
| `intent(out)` | Output only, undefined on entry |
| `intent(inout)` | Both read and modified |

```fortran
subroutine update(x, y)
    real, intent(in)    :: x
    real, intent(inout) :: y
    y = y + x
end subroutine update
```

### 7.4 Interfaces & Generic Procedures

Generic (overloaded) procedures — Fortran's version of function overloading:

```fortran
interface add
    module procedure add_int
    module procedure add_real
end interface add

contains

function add_int(a, b) result(c)
    integer, intent(in) :: a, b
    integer :: c
    c = a + b
end function add_int

function add_real(a, b) result(c)
    real, intent(in) :: a, b
    real :: c
    c = a + b
end function add_real
```

### 7.5 Coarrays (Parallel Fortran)

Native parallelism (Partitioned Global Address Space model), part of the standard since Fortran 2008.

```fortran
integer :: x[*]              ! coarray: one copy per "image" (process)
x = this_image()
sync all
if (this_image() == 1) then
    print *, "Value on image 2:", x[2]
end if
```

### 7.6 `do concurrent`

Tells the compiler loop iterations have no dependencies — safe to parallelize/vectorize (Fortran 2008+, enhanced in 2018):

```fortran
do concurrent (i = 1:n)
    a(i) = b(i) * c(i)
end do
```

---

## 8. Managing a Fortran Project

### 8.1 Modules & `use`

Modules are Fortran's unit of encapsulation — they hold types, procedures, and constants and are the standard way to organize code across files.

```fortran
! file: math_utils.f90
module math_utils
    implicit none
    private                      ! nothing exported by default
    public :: square, PI         ! explicitly expose these

    real, parameter :: PI = 3.14159265

contains

    function square(x) result(y)
        real, intent(in) :: x
        real :: y
        y = x * x
    end function square

end module math_utils
```

```fortran
! file: main.f90
program main
    use math_utils, only: square, PI   ! import specific names
    implicit none

    print *, square(4.0), PI
end program main
```

| `use` variant | Effect |
|---|---|
| `use my_module` | Imports everything public |
| `use my_module, only: foo` | Imports only `foo` |
| `use my_module, only: bar => foo` | Imports `foo`, renamed locally to `bar` |

### 8.2 Submodules

Submodules (Fortran 2008+) let you separate a module's **interface** from its **implementation**, drastically speeding up recompilation on large projects.

```fortran
! file: shapes_iface.f90
module shapes
    implicit none
    interface
        module function circle_area(r) result(a)
            real, intent(in) :: r
            real :: a
        end function circle_area
    end interface
end module shapes
```

```fortran
! file: shapes_impl.f90
submodule (shapes) shapes_impl
contains
    module function circle_area(r) result(a)
        real, intent(in) :: r
        real :: a
        a = 3.14159 * r**2
    end function circle_area
end submodule shapes_impl
```

### 8.3 Project Layout

A conventional, tool-friendly Fortran project layout:

```
my_project/
├── fpm.toml              # package manifest (see 8.4)
├── app/
│   └── main.f90           # program entry point
├── src/
│   ├── math_utils.f90      # library modules
│   └── shapes.f90
├── test/
│   └── test_math.f90       # unit tests
└── build/                  # compiler output (generated)
```

### 8.4 Fortran Package Manager (`fpm`)

[`fpm`](https://fpm.fortran-lang.org) is the modern, community-standard build tool and package manager (similar to `cargo` for Rust).

```toml
# fpm.toml
name = "my_project"
version = "0.1.0"

[dependencies]
stdlib = { git = "https://github.com/fortran-lang/stdlib" }
```

| Command | Effect |
|---|---|
| `fpm new my_project` | Scaffold a new project |
| `fpm build` | Build the project |
| `fpm run` | Build and run the app |
| `fpm test` | Run the test suite |
| `fpm add <package>` | Add a dependency to `fpm.toml` |

### 8.5 Compiling Manually

Without `fpm`, using `gfortran` directly — module compilation order matters, since a module must be compiled before any file that `use`s it:

```bash
gfortran -c math_utils.f90     # produces math_utils.o and math_utils.mod
gfortran -c main.f90           # main.f90 uses math_utils
gfortran -o my_program math_utils.o main.o
```

---

## 9. Advanced / Dynamic Data Structures

### 9.1 `allocatable` Arrays

The preferred way to manage dynamic memory — automatically deallocated when out of scope (unlike raw pointers).

```fortran
integer, allocatable :: arr(:)

allocate(arr(10))          ! allocate 10 elements
arr = 0
print *, size(arr)

deallocate(arr)             ! free memory manually if needed early
```

Resizing dynamically (append-like pattern):

```fortran
integer, allocatable :: arr(:), temp(:)
allocate(arr(3))
arr = [1, 2, 3]

! "append" a value by reallocating
allocate(temp(size(arr) + 1))
temp(1:size(arr)) = arr
temp(size(arr) + 1) = 4
call move_alloc(temp, arr)   ! efficient: moves allocation, no copy
```

### 9.2 Pointers

```fortran
integer, target  :: x = 5
integer, pointer :: p

p => x              ! p points to x
print *, p          ! prints 5
p = 10               ! also changes x, since p points to it

nullify(p)           ! unset the pointer
if (associated(p)) print *, "p points to something"
```

| Concept | Keyword |
|---|---|
| Declare pointer | `integer, pointer :: p` |
| Declare pointer target | `integer, target :: x` |
| Point to something | `p => x` |
| Check association | `associated(p)` |
| Unset | `nullify(p)` |

### 9.3 Linked Lists

Fortran supports self-referential derived types via pointers — the basis for linked lists, trees, and graphs.

```fortran
type :: node
    integer               :: value
    type(node), pointer   :: next => null()
end type node

! Building a list
type(node), pointer :: head, current, new_node

allocate(head)
head%value = 1

allocate(new_node)
new_node%value = 2
head%next => new_node

! Traversing
current => head
do while (associated(current))
    print *, current%value
    current => current%next
end do
```

### 9.4 Generic (Templated) Containers

Fortran lacks C++-style templates, but the standard library `stdlib` provides generic containers, and you can roll your own with **unlimited polymorphism** (`class(*)`):

```fortran
type :: generic_box
    class(*), allocatable :: item
end type generic_box

type(generic_box) :: box
box%item = 42                     ! can hold any type

select type (val => box%item)
type is (integer)
    print *, "Integer:", val
type is (real)
    print *, "Real:", val
end select
```

Alternatively, use [`stdlib`](https://stdlib.fortran-lang.org) which ships ready-made generic containers:

```fortran
use stdlib_hashmaps, only: chaining_hashmap_type
use stdlib_stringlist_type, only: stringlist_type
```

| stdlib module | Provides |
|---|---|
| `stdlib_hashmaps` | Hash map / dictionary types |
| `stdlib_stringlist_type` | Dynamic list of strings |
| `stdlib_bitsets` | Bitset data structure |
| `stdlib_linked_list` (varies by version) | Linked list utilities |

### 9.5 Dynamic Dispatch Containers

Combine `class(shape)` polymorphism (Section 4.3) with `allocatable` arrays to build a heterogeneous collection — Fortran's version of a polymorphic vector:

```fortran
type :: shape_container
    class(shape), allocatable :: item
end type shape_container

type(shape_container), allocatable :: shapes(:)
allocate(shapes(2))
allocate(shapes(1)%item, source=rectangle(width=2.0, height=3.0))
allocate(shapes(2)%item, source=square(width=4.0, height=4.0))

do i = 1, size(shapes)
    call shapes(i)%item%print_area()
end do
```

---

## 10. Error Handling

Fortran uses status codes/optional arguments rather than exceptions.

```fortran
integer :: ios
open(unit=10, file="data.txt", status="old", iostat=ios)
if (ios /= 0) then
    print *, "Error opening file!"
    stop 1
end if
```

```fortran
integer, allocatable :: arr(:)
integer :: alloc_stat
allocate(arr(100), stat=alloc_stat)
if (alloc_stat /= 0) print *, "Allocation failed"
```

| Common `iostat`/`stat` pattern | Use case |
|---|---|
| `open(..., iostat=ios)` | File I/O errors |
| `read(..., iostat=ios)` | Read errors / end-of-file detection |
| `allocate(..., stat=s)` | Memory allocation failure |

---

## 11. Quick Reference Cheat Sheet

| Task | Syntax |
|---|---|
| Declare a variable | `integer :: x` |
| Declare a constant | `real, parameter :: pi = 3.14159` |
| Print output | `print *, x` |
| Read input | `read *, x` |
| String length | `len(str)` |
| Array size | `size(arr)` |
| Loop | `do i = 1, n ... end do` |
| Conditional | `if (x > 0) then ... end if` |
| Define a module | `module name ... end module name` |
| Import a module | `use name, only: item` |
| Derived type | `type :: t ... end type t` |
| Polymorphic variable | `class(t), allocatable :: x` |
| Dynamic array | `integer, allocatable :: a(:)` |
| Allocate memory | `allocate(a(n))` |
| Pointer declaration | `integer, pointer :: p` |
| Point to a target | `p => x` |
| Subroutine | `subroutine s(args) ... end subroutine s` |
| Function | `function f(args) result(y) ... end function f` |
| Comment | `! this is a comment` |

---

### Further Reading

- [Fortran-lang.org](https://fortran-lang.org) — official community hub
- [Fortran Package Manager (fpm)](https://fpm.fortran-lang.org)
- [stdlib — Fortran Standard Library](https://stdlib.fortran-lang.org)
- ISO/IEC 1539-1:2023 — the official Fortran 2023 standard