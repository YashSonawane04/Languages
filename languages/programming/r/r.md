# 📘 R Programming — Complete Concepts Cheat Sheet

A practical, example-driven reference covering R basics, data structures, OOP, R-specific features, project management, and advanced data structures for dynamic programming.

---

## 📑 Table of Contents

1. [Basics & Syntax](#1-basics--syntax)
2. [Data Types](#2-data-types)
3. [Type Checking & Coercion](#3-type-checking--coercion)
4. [Data Structures](#4-data-structures)
5. [Indexing & Subsetting](#5-indexing--subsetting)
6. [Control Flow & Loops](#6-control-flow--loops)
7. [Functions](#7-functions)
8. [Functional Programming & Apply Family](#8-functional-programming--apply-family)
9. [R-Specific Features](#9-r-specific-features)
10. [Object-Oriented Programming](#10-object-oriented-programming)
11. [Error Handling & Debugging](#11-error-handling--debugging)
12. [Managing Your R Project](#12-managing-your-r-project)
13. [Advanced Data Structures for Dynamic Programming](#13-advanced-data-structures-for-dynamic-programming)
14. [Quick Reference Tables](#14-quick-reference-tables)

---

## 1. Basics & Syntax

### Assignment, comments, printing

```r
x <- 10            # preferred assignment operator
y = 20             # works, but <- is the convention
30 -> z            # right assignment (rare)
x <<- 5            # super-assignment (assigns in parent environment)

# This is a comment (R has no multi-line comments)

print(x)           # explicit print
x                  # auto-print at top level
cat("Value:", x, "\n")   # raw output, no [1] prefix
message("Info")          # prints to stderr
paste("a", "b", sep = "-")   # "a-b"
sprintf("%.2f", pi)          # "3.14"
```

### Operators

| Category   | Operators                         | Example                    |
|------------|-----------------------------------|----------------------------|
| Arithmetic | `+  -  *  /  ^  %%  %/%`          | `7 %% 3` → `1`, `7 %/% 3` → `2` |
| Comparison | `==  !=  <  >  <=  >=`            | `3 != 4` → `TRUE`          |
| Logical    | `&  \|  !  &&  \|\|  xor()`       | `TRUE && FALSE` → `FALSE`  |
| Membership | `%in%`                            | `3 %in% c(1,2,3)` → `TRUE` |
| Sequence   | `:`                               | `1:5` → `1 2 3 4 5`        |
| Pipe       | `\|>` (base), `%>%` (magrittr)    | `x \|> sqrt()`             |
| Matrix     | `%*%  %o%`                        | `A %*% B`                  |

> **Note:** `&` / `|` are **vectorised** (element-wise); `&&` / `||` work on **single** values and short-circuit.

### Getting help

```r
?mean              # help for a function
??"regression"     # search help
help(package = "stats")
example(mean)
args(mean)         # show arguments
str(obj)           # structure of any object
```

---

## 2. Data Types

R has **6 atomic types** (plus `NULL`):

| Type        | `typeof()`   | Example              | Notes                           |
|-------------|--------------|----------------------|---------------------------------|
| Logical     | `"logical"`  | `TRUE`, `FALSE`, `T`, `F` | Avoid `T`/`F` (can be overwritten) |
| Integer     | `"integer"`  | `5L`                 | Suffix `L` required             |
| Double      | `"double"`   | `3.14`, `5`          | Default numeric type            |
| Character   | `"character"`| `"hello"`            | Single or double quotes         |
| Complex     | `"complex"`  | `2+3i`               | Rare                            |
| Raw         | `"raw"`      | `as.raw(255)`        | Bytes                           |

```r
a <- 42L           # integer
b <- 3.14          # double (numeric)
c <- "R is fun"    # character
d <- TRUE          # logical
e <- 2 + 3i        # complex
f <- NULL          # absence of a value
```

### Special values

| Value      | Meaning                          | Check with       |
|------------|----------------------------------|------------------|
| `NA`       | Missing value                    | `is.na(x)`       |
| `NaN`      | Not a number (`0/0`)             | `is.nan(x)`      |
| `Inf`, `-Inf` | Infinity (`1/0`)              | `is.infinite(x)` |
| `NULL`     | Empty / undefined object         | `is.null(x)`     |

```r
NA + 1               # NA  (NA is contagious)
mean(c(1, NA, 3))    # NA
mean(c(1, NA, 3), na.rm = TRUE)   # 2
length(NULL)         # 0
```

### Dates and times

```r
d <- as.Date("2026-10-07")
format(d, "%d/%m/%Y")             # "07/10/2026"
d + 30                            # add 30 days
seq(d, by = "month", length.out = 3)
Sys.Date(); Sys.time()
dt <- as.POSIXct("2026-10-07 14:30:00", tz = "UTC")
```

### Strings

```r
s <- "Hello, World"
nchar(s)                    # 12
toupper(s); tolower(s)
substr(s, 1, 5)             # "Hello"
strsplit(s, ", ")[[1]]      # "Hello" "World"
gsub("World", "R", s)       # "Hello, R"
grepl("World", s)           # TRUE
sprintf("%s has %d chars", s, nchar(s))
trimws("  hi  ")
```

---

## 3. Type Checking & Coercion

```r
class(x)       # high-level class
typeof(x)      # internal storage type
mode(x)        # storage mode
is.numeric(x); is.character(x); is.logical(x); is.function(x)
inherits(x, "data.frame")
```

### Explicit coercion

```r
as.integer("42")        # 42L
as.numeric("3.5")       # 3.5
as.character(100)       # "100"
as.logical("TRUE")      # TRUE
as.logical(0)           # FALSE
as.numeric("abc")       # NA with a warning
```

### Implicit coercion hierarchy

`logical` → `integer` → `double` → `complex` → `character`

```r
c(1, TRUE, "a")     # "1" "TRUE" "a"   (everything becomes character)
c(1, TRUE, FALSE)   # 1 1 0
```

---

## 4. Data Structures

| Structure     | Dimensions | Homogeneous? | Create with                      |
|---------------|------------|--------------|----------------------------------|
| **Vector**    | 1D         | ✅ Yes       | `c()`, `vector()`                |
| **Matrix**    | 2D         | ✅ Yes       | `matrix()`                       |
| **Array**     | nD         | ✅ Yes       | `array()`                        |
| **List**      | 1D         | ❌ No        | `list()`                         |
| **Data frame**| 2D         | ❌ (by column)| `data.frame()`                  |
| **Factor**    | 1D         | ✅ (categorical)| `factor()`                    |
| **Environment** | key→value| ❌ No        | `new.env()`                      |

### 4.1 Vectors

```r
v <- c(10, 20, 30, 40)
v2 <- 1:10
v3 <- seq(0, 1, by = 0.25)        # 0.00 0.25 0.50 0.75 1.00
v4 <- rep(c("a", "b"), times = 3) # "a" "b" "a" "b" "a" "b"
v5 <- vector("numeric", length = 5)

length(v); sum(v); mean(v); sd(v)
rev(v); sort(v, decreasing = TRUE); order(v)
which(v > 15)          # indices where TRUE
which.max(v); which.min(v)
any(v > 35); all(v > 5)
cumsum(v); diff(v); range(v)
unique(c(1,1,2)); table(c("a","b","a"))

# Named vector
ages <- c(Alice = 30, Bob = 25)
ages["Alice"]
names(ages)
```

### 4.2 Matrices

```r
m <- matrix(1:6, nrow = 2, ncol = 3)             # filled by column
m2 <- matrix(1:6, nrow = 2, byrow = TRUE)        # filled by row

dim(m); nrow(m); ncol(m)
t(m)                       # transpose
m %*% t(m)                 # matrix multiplication
solve(matrix(c(2,0,0,2), 2))   # inverse
diag(3)                    # 3x3 identity
rbind(m, 7:9); cbind(m, c(0, 0))
rownames(m) <- c("r1", "r2")
colnames(m) <- c("A", "B", "C")
apply(m, 1, sum)           # row sums
apply(m, 2, mean)          # col means
```

### 4.3 Arrays

```r
arr <- array(1:24, dim = c(2, 3, 4))   # 2 rows, 3 cols, 4 layers
arr[1, 2, 3]
dim(arr)
```

### 4.4 Lists

```r
person <- list(name = "Ada", age = 36, langs = c("R", "Python"))

person$name            # "Ada"          (by name)
person[["age"]]        # 36             (extract the element)
person["age"]          # list of 1      (keeps list wrapper)
person$langs[2]        # "Python"

person$city <- "London"    # add
person$age <- NULL         # remove
names(person)
length(person)
str(person)

# Nested list
cfg <- list(db = list(host = "localhost", port = 5432))
cfg$db$port
cfg[["db"]][["host"]]

unlist(list(a = 1, b = 2))   # flatten to vector
```

> **`[` vs `[[`**: `[ ]` returns a *sub-list*, `[[ ]]` returns the *element itself*.

### 4.5 Data frames

```r
df <- data.frame(
  id    = 1:4,
  name  = c("Ann", "Bob", "Cy", "Di"),
  score = c(88, 92, 75, 60),
  stringsAsFactors = FALSE
)

head(df, 2); tail(df, 2)
nrow(df); ncol(df); dim(df)
str(df); summary(df); names(df)

df$score                      # column as vector
df[df$score > 70, ]           # filter rows
df[order(-df$score), ]        # sort descending
df$grade <- ifelse(df$score >= 80, "A", "B")   # new column
subset(df, score > 70, select = c(name, score))
merge(df, other, by = "id")   # join
rbind(df, data.frame(id = 5, name = "Ed", score = 99, grade = "A"))
aggregate(score ~ grade, data = df, FUN = mean)
with(df, mean(score))
```

### 4.6 Factors

```r
sizes <- factor(c("S", "L", "M", "S"),
                levels = c("S", "M", "L"), ordered = TRUE)
levels(sizes)       # "S" "M" "L"
as.integer(sizes)   # 1 3 2 1
table(sizes)
droplevels(sizes)
```

### 4.7 Tibbles (tidyverse)

```r
library(tibble)
t <- tibble(x = 1:3, y = c("a", "b", "c"))   # prints nicely, never partial-matches
as_tibble(df)
```

---

## 5. Indexing & Subsetting

> ⚠️ **R is 1-indexed** (first element is `x[1]`, not `x[0]`).

| Type            | Example              | Result                          |
|-----------------|----------------------|---------------------------------|
| Positive index  | `v[c(1, 3)]`         | 1st and 3rd elements            |
| Negative index  | `v[-1]`              | everything **except** the first |
| Logical         | `v[v > 20]`          | elements where condition is TRUE|
| Name            | `ages["Bob"]`        | by name                         |
| Matrix          | `m[2, 3]`, `m[ , 1]` | row 2 col 3; whole 1st column   |
| Drop dimension  | `m[ , 1, drop = FALSE]` | keep matrix shape            |
| List element    | `lst[["a"]]`, `lst$a`| element                         |

```r
v <- c(5, 10, 15, 20)
v[2:3]        # 10 15
v[-c(1, 4)]   # 10 15
v[v %% 10 == 0]   # 10 20
v[10]         # NA (out of range, no error)
head(v, -1); tail(v, 2)
```

---

## 6. Control Flow & Loops

### Conditionals

```r
if (x > 0) {
  print("positive")
} else if (x == 0) {
  print("zero")
} else {
  print("negative")
}

# Vectorised if
ifelse(c(1, -2, 3) > 0, "pos", "neg")

# switch
switch("b",
  a = "Apple",
  b = "Banana",
  "Unknown")          # default
```

### Loops

```r
for (i in 1:3) print(i)
for (name in names(person)) cat(name, "\n")

i <- 0
while (i < 3) { i <- i + 1; if (i == 2) next; print(i) }

repeat {
  i <- i + 1
  if (i > 5) break
}

seq_along(v)    # safe alternative to 1:length(v)  (handles empty vectors)
seq_len(0)      # integer(0) — loop runs zero times
```

---

## 7. Functions

```r
add <- function(a, b = 1) {
  a + b                 # last expression is returned (return() optional)
}
add(2)            # 3
add(b = 5, a = 2) # 7  (named args, any order)

# Variadic arguments
total <- function(...) {
  args <- list(...)
  sum(unlist(args))
}
total(1, 2, 3)     # 6

# Early return
safe_div <- function(a, b) {
  if (b == 0) return(NA)
  a / b
}

# Anonymous / lambda functions
sapply(1:3, function(x) x^2)
sapply(1:3, \(x) x^2)        # R >= 4.1 shorthand

# Multiple returns -> use a list
stats <- function(x) list(mean = mean(x), sd = sd(x))
stats(c(1, 2, 3))$mean
```

### Useful function utilities

```r
missing(arg)          # was an argument supplied?
match.arg(type)       # restrict to allowed choices
stopifnot(is.numeric(x))
on.exit(cat("done\n")) # cleanup when function exits
do.call(sum, list(1, 2, 3))
Recall(n - 1)         # recursive call without naming the function
```

---

## 8. Functional Programming & Apply Family

| Function    | Input → Output                           | Example                              |
|-------------|------------------------------------------|--------------------------------------|
| `lapply()`  | vector/list → **list**                   | `lapply(1:3, sqrt)`                  |
| `sapply()`  | vector/list → vector/matrix (simplified) | `sapply(1:3, sqrt)`                  |
| `vapply()`  | like `sapply`, type-safe                 | `vapply(1:3, sqrt, numeric(1))`      |
| `apply()`   | matrix margin → vector                   | `apply(m, 1, sum)`                   |
| `mapply()`  | multiple vectors in parallel             | `mapply(function(a, b) a + b, 1:3, 4:6)` |
| `tapply()`  | apply by group                           | `tapply(df$score, df$grade, mean)`   |
| `Map()`     | `mapply` returning a list                | `Map(`+`, 1:3, 4:6)`                 |
| `Filter()`  | keep elements where TRUE                 | `Filter(function(x) x > 1, 1:3)`     |
| `Reduce()`  | fold / accumulate                        | `Reduce(`+`, 1:5, accumulate = TRUE)`|

```r
Reduce(function(a, b) a * b, 1:5)               # 120
Reduce(`+`, 1:5, accumulate = TRUE)             # 1 3 6 10 15
Filter(function(x) x %% 2 == 0, 1:10)           # 2 4 6 8 10
Map(function(n, v) paste(n, v), c("a","b"), c(1,2))
```

### Pipes

```r
# Base R (>= 4.1)
c(1, 4, 9) |> sqrt() |> sum()

# magrittr / dplyr
library(dplyr)
df %>%
  filter(score > 70) %>%
  mutate(pct = score / 100) %>%
  arrange(desc(score)) %>%
  select(name, pct)
```

---

## 9. R-Specific Features

### 9.1 Vectorisation

Operations apply to whole vectors — avoid loops when possible.

```r
c(1, 2, 3) * 2          # 2 4 6
c(1, 2, 3) + c(10, 20, 30)   # 11 22 33
sqrt(c(4, 9, 16))       # 2 3 4
```

### 9.2 Recycling

Shorter vectors are repeated to match the longer one.

```r
c(1, 2, 3, 4) + c(10, 20)     # 11 22 13 24
c(1, 2, 3) + c(10, 20)        # warning: lengths not multiples
```

### 9.3 Everything is an object & everything is a function call

```r
`+`(2, 3)             # 5   (same as 2 + 3)
sapply(1:3, `-`)      # -1 -2 -3
x[2]    # same as  `[`(x, 2)
`my var` <- 5         # backticks allow non-syntactic names
```

### 9.4 Lazy evaluation

Arguments are evaluated only when used.

```r
f <- function(a, b) { if (a) "skip b" else b }
f(TRUE, stop("never evaluated"))    # "skip b" — no error
```

### 9.5 Custom infix operators

```r
`%+%` <- function(a, b) paste(a, b)
"Hello" %+% "R"        # "Hello R"

`%||%` <- function(a, b) if (is.null(a)) b else a
NULL %||% "default"    # "default"
```

### 9.6 Replacement functions

```r
`second<-` <- function(x, value) { x[2] <- value; x }
v <- 1:3
second(v) <- 99L       # v is now 1 99 3

names(v) <- c("a", "b", "c")   # built-in replacement function
levels(f)[1] <- "new"
```

### 9.7 Environments, scoping & closures

```r
# Lexical scoping: functions look up variables where they were *defined*
make_counter <- function() {
  count <- 0
  function() {
    count <<- count + 1
    count
  }
}
counter <- make_counter()
counter(); counter()    # 1, 2

e <- new.env()
assign("k", 10, envir = e)
get("k", envir = e)
exists("k", envir = e)
ls(e)
environment()           # current env
globalenv(); emptyenv()
```

### 9.8 Non-standard evaluation (NSE) & metaprogramming

```r
quote(x + y)                  # capture an expression
expr <- quote(a * b)
eval(expr, list(a = 2, b = 3))   # 6
substitute(x + y)            # capture a function's argument expression
deparse(substitute(x))       # arg → string  (how plot() gets axis labels)
body(f); formals(f); environment(f)
parse(text = "1 + 2") |> eval()
bquote(.(x) + y)
```

### 9.9 Missing values and NULL handling

```r
x <- c(1, NA, 3)
is.na(x)                 # FALSE TRUE FALSE
x[!is.na(x)]
complete.cases(df)
na.omit(df)
ifelse(is.na(x), 0, x)
```

### 9.10 Misc. idioms

```r
x <- c(a = 1, b = 2)
invisible(x)             # return without printing
Sys.getenv("HOME")
Sys.setenv(MY_VAR = "x")
set.seed(42); sample(1:10, 3); runif(2); rnorm(2)
identical(c(1, 2), c(1, 2))
all.equal(0.1 + 0.2, 0.3)     # TRUE (tolerance); `==` would be FALSE
```

---

## 10. Object-Oriented Programming

R has **several** OOP systems:

| System            | Style              | Mutable? | Best for                            |
|-------------------|--------------------|----------|-------------------------------------|
| **S3**            | Informal, generic-function | No (copy-on-modify) | Quick, lightweight, most common |
| **S4**            | Formal, validated  | No       | Strict class definitions (Bioconductor) |
| **R5 / RC**       | Reference classes, `obj$method()` | ✅ Yes | Mutable state, Java-like OOP |
| **R6** (package)  | Modern reference classes | ✅ Yes | Fast, clean, private members |

### 10.1 S3

Objects are just lists/vectors with a `class` attribute. Methods are dispatched on the **first argument**.

```r
# Constructor
dog <- function(name, sound = "Woof") {
  obj <- list(name = name, sound = sound)
  class(obj) <- c("dog", "animal")       # class vector = inheritance chain
  obj
}

# Generic
speak <- function(x, ...) UseMethod("speak")

# Methods: generic.class
speak.animal  <- function(x, ...) cat("Some sound\n")
speak.dog     <- function(x, ...) {
  cat(x$name, "says", x$sound, "\n")
  NextMethod()                            # call parent method
}
print.dog     <- function(x, ...) cat("<dog:", x$name, ">\n")
format.dog    <- function(x, ...) paste("Dog", x$name)

d <- dog("Rex")
speak(d)          # Rex says Woof / Some sound
d                 # <dog: Rex>
inherits(d, "animal")   # TRUE
methods(speak)
unclass(d)
```

Operator overloading in S3:

```r
"+.money" <- function(e1, e2) {
  structure(list(v = e1$v + e2$v), class = "money")
}
"==.money" <- function(e1, e2) e1$v == e2$v
```

### 10.2 S4

```r
# Define class
setClass("Person",
  representation(name = "character", age = "numeric"),
  prototype(age = 0),
  validity = function(object) {
    if (object@age < 0) "age must be non-negative" else TRUE
  }
)

# Inheritance
setClass("Student", contains = "Person",
         representation(school = "character"))

# Create
p <- new("Person", name = "Ann", age = 30)
s <- new("Student", name = "Bob", age = 20, school = "MIT")

p@name              # slot access
slot(p, "age")
p@age <- 31
isVirtualClass("Person")
validObject(p)

# Generic + method
setGeneric("greet", function(obj, ...) standardGeneric("greet"))
setMethod("greet", "Person", function(obj, ...) cat("Hi,", obj@name, "\n"))
setMethod("greet", "Student", function(obj, ...) {
  callNextMethod()
  cat("I study at", obj@school, "\n")
})
greet(s)

# show method (like print)
setMethod("show", "Person", function(object) cat("<Person", object@name, ">\n"))

is(s, "Person")     # TRUE
existsMethod("greet", "Person")
```

### 10.3 Reference Classes (R5 / RC)

```r
Account <- setRefClass("Account",
  fields = list(owner = "character", balance = "numeric"),
  methods = list(
    deposit = function(x) {
      balance <<- balance + x       # <<- modifies the field
      invisible(.self)              # enables chaining
    },
    show = function() cat(owner, ":", balance, "\n")
  )
)

acc <- Account$new(owner = "Ann", balance = 100)
acc$deposit(50)$deposit(25)
acc$balance          # 175
b <- acc             # reference — b and acc are the SAME object
b2 <- acc$copy()     # real copy

# Inheritance
Savings <- setRefClass("Savings", contains = "Account",
  fields = list(rate = "numeric"),
  methods = list(
    addInterest = function() {
      deposit(balance * rate)
    }
  ))
```

### 10.4 R6 (modern, recommended for mutable OOP)

```r
install.packages("R6")
library(R6)

Stack <- R6Class("Stack",
  public = list(
    items = list(),
    push = function(x) { self$items[[length(self$items) + 1]] <- x; invisible(self) },
    pop = function() {
      top <- self$items[[length(self$items)]]
      self$items[[length(self$items)]] <- NULL
      top
    },
    size = function() length(self$items),
    initialize = function() { cat("Stack created\n") }   # constructor
  ),
  private = list(
    secret = 42
  ),
  active = list(                               # computed "property"
    top = function() self$items[[length(self$items)]]
  )
)

s <- Stack$new()
s$push(1)$push(2)$push(3)
s$pop()      # 3
s$top        # 2   (active binding — no parentheses)

# Inheritance
SmartStack <- R6Class("SmartStack", inherit = Stack,
  public = list(
    push = function(x) { cat("Pushing\n"); super$push(x) }
  ))
```

### 10.5 Choosing an OOP system

```text
Need simple, quick polymorphism?          → S3
Need formal classes + validation?         → S4
Need mutable objects / state / references?→ R6 (or RC)
```

---

## 11. Error Handling & Debugging

```r
# tryCatch
result <- tryCatch({
  log(-1)                 # warning
  stop("custom failure")
}, warning = function(w) {
  message("Warning: ", conditionMessage(w)); NA
}, error = function(e) {
  message("Error: ", conditionMessage(e)); NULL
}, finally = {
  cat("always runs\n")
})

try(stop("oops"), silent = TRUE)
inherits(try(log("a"), silent = TRUE), "try-error")

stop("fatal")         # raise error
warning("careful")    # raise warning
signalCondition(cond) # custom conditions

# Custom condition
my_error <- structure(class = c("myError", "error", "condition"),
                      list(message = "custom", call = NULL))
tryCatch(stop(my_error), myError = function(e) "caught custom")
```

### Debugging tools

| Tool                    | Purpose                              |
|-------------------------|--------------------------------------|
| `browser()`             | Pause execution & inspect            |
| `debug(f)` / `undebug(f)` | Step through function `f`          |
| `traceback()`           | Show call stack after an error       |
| `options(error = recover)` | Interactive post-mortem           |
| `print()`, `str()`, `message()` | Quick inspection             |
| `system.time(expr)`     | Time code                            |
| `Rprof()` / `profvis`   | Profile performance                  |
| `microbenchmark::microbenchmark()` | Compare snippets          |

---

## 12. Managing Your R Project

### 12.1 Recommended project layout

```text
my_project/
├── my_project.Rproj        # RStudio project file (sets working dir)
├── renv.lock               # package versions (renv)
├── .Rprofile               # runs at startup
├── main.R                  # entry point
├── R/                      # your reusable code (modules)
│   ├── utils.R
│   ├── data_cleaning.R
│   └── models.R
├── data/
│   ├── raw/
│   └── processed/
├── scripts/
├── tests/
│   └── testthat/
├── output/
└── README.md
```

### 12.2 Working directory & paths

```r
getwd()
setwd("path/to/project")        # avoid in scripts — fragile!

# ✅ Better: use the `here` package (project-root relative paths)
install.packages("here")
library(here)
here()                           # project root
here("data", "raw", "file.csv")  # always correct, any OS
```

### 12.3 Importing local modules / scripts

| Method                    | Use case                                | Syntax |
|---------------------------|-----------------------------------------|--------|
| `source()`                | Run another script in the global env    | `source("R/utils.R")` |
| `source(local = TRUE)`    | Run inside current env (no pollution)   | `source("R/utils.R", local = TRUE)` |
| `sys.source()`            | Source into a specific environment      | `sys.source("R/utils.R", envir = e)` |
| `box::use()`              | **Real modules** (Python-style imports) | `box::use(R/utils)` |
| Local package             | Best for larger reusable code           | `devtools::load_all()` |

#### a) `source()` — simplest approach

```r
# main.R
source("R/utils.R")
source(here::here("R", "data_cleaning.R"))

# Source every file in a folder
for (f in list.files("R", pattern = "\\.R$", full.names = TRUE)) source(f)
# or
invisible(lapply(list.files("R", full.names = TRUE, pattern = "\\.[Rr]$"), source))
```

`R/utils.R`:

```r
add_one <- function(x) x + 1
```

#### b) Sourcing into an isolated namespace (poor man's module)

```r
utils_env <- new.env()
sys.source("R/utils.R", envir = utils_env)
utils_env$add_one(5)       # 6  — no global pollution
```

#### c) `box` package — proper modules

```r
install.packages("box")

# R/utils.R  (a module — just a normal file)
#' @export
add_one <- function(x) x + 1

# main.R
box::use(
  R/utils,                       # import whole module: utils$add_one()
  R/utils[add_one],              # import specific function
  dplyr[filter, mutate],         # import from a package
  stats[sd, median]
)
utils$add_one(1)
add_one(1)

# Options
box::use(R/utils[...])           # attach all exports
box::use(R/utils[add = add_one]) # alias
box::reload(utils)               # reload after edits
```

#### d) Turn code into a package (best for reuse)

```r
install.packages(c("devtools", "usethis", "roxygen2", "testthat"))

usethis::create_package("~/mypkg")
usethis::use_r("utils")          # creates R/utils.R
usethis::use_test("utils")
usethis::use_package("dplyr")    # add dependency
devtools::load_all()             # load for dev (like source() for all R/ files)
devtools::document()             # generate docs/NAMESPACE from roxygen
devtools::test()
devtools::install()
```

Roxygen comments (`#'`) document + export functions:

```r
#' Add one
#' @param x A number
#' @return x + 1
#' @export
add_one <- function(x) x + 1
```

### 12.4 Packages

```r
install.packages("dplyr")        # from CRAN
library(dplyr)                   # attach to search path (errors if missing)
require(dplyr)                   # same but returns TRUE/FALSE
requireNamespace("dplyr", quietly = TRUE)
dplyr::filter(df, x > 1)         # use without attaching (recommended in packages)
dplyr:::internal_fn()            # access non-exported function (avoid)

remotes::install_github("user/repo")
packageVersion("dplyr")
installed.packages()[, "Package"]
sessionInfo()
search()                         # attached packages
```

> **Name conflicts:** if two packages export the same function, the **last attached** wins. Use `pkg::fn()` to be explicit.

### 12.5 Reproducibility with `renv`

```r
install.packages("renv")
renv::init()         # create project-local library
renv::snapshot()     # save package versions to renv.lock
renv::restore()      # reinstall exact versions on another machine
renv::status()
```

### 12.6 Reading & writing data

```r
# CSV
df <- read.csv("data/raw/file.csv")
write.csv(df, "output/clean.csv", row.names = FALSE)

# Faster / modern
readr::read_csv("data.csv"); readr::write_csv(df, "out.csv")
data.table::fread("big.csv")

# Excel, JSON
readxl::read_excel("file.xlsx", sheet = 1)
jsonlite::fromJSON("file.json"); jsonlite::toJSON(df)

# R-native binary formats
saveRDS(obj, "obj.rds");  obj <- readRDS("obj.rds")       # single object
save(a, b, file = "ab.RData"); load("ab.RData")           # multiple objects

# Files & dirs
file.exists("x.csv"); dir.create("output", showWarnings = FALSE)
list.files("data", pattern = "\\.csv$", full.names = TRUE)
file.path("data", "raw", "x.csv")   # OS-independent path join
basename("a/b/c.txt"); dirname("a/b/c.txt")
```

### 12.7 Config, command-line args, startup

```r
# Command-line args:  Rscript script.R 10 hello
args <- commandArgs(trailingOnly = TRUE)
n <- as.integer(args[1])

# Only run main() when executed as a script (not when sourced)
if (sys.nframe() == 0) main()

# Environment variables / secrets (.Renviron file: API_KEY=abc123)
Sys.getenv("API_KEY")

# Run from terminal
#   Rscript main.R
```

### 12.8 Testing with `testthat`

```r
library(testthat)

test_that("add_one works", {
  expect_equal(add_one(1), 2)
  expect_true(is.numeric(add_one(1)))
  expect_error(add_one("a"))
})
# Run all:  testthat::test_dir("tests/testthat")  or  devtools::test()
```

---

## 13. Advanced Data Structures for Dynamic Programming

Base R lacks built-in hash maps, stacks, queues, or heaps, but you can build them with **environments**, **lists**, **matrices**, and **R6**. Below are the building blocks used in DP and algorithmic problems.

### 13.1 Data-structure map

| Need                         | R implementation                                 | Lookup/Insert |
|------------------------------|--------------------------------------------------|---------------|
| Hash map / dictionary        | `new.env(hash = TRUE)` (best) or named `list`    | ~O(1) (env)   |
| 1D DP table                  | numeric/integer vector `numeric(n)`              | O(1) index    |
| 2D DP table                  | `matrix(0, nrow, ncol)`                          | O(1) index    |
| 3D DP table                  | `array(0, dim = c(a, b, c))`                     | O(1) index    |
| Memo cache (any key type)    | environment keyed by `as.character(key)`         | ~O(1)         |
| Stack (LIFO)                 | vector/list + `length()`, or R6                  | O(1) amortised|
| Queue (FIFO)                 | R6 with head/tail index, `collections::queue()`  | O(1)          |
| Priority queue / heap        | `collections::priority_queue()`                  | O(log n)      |
| Set                          | environment, or `unique()` / `%in%`              | ~O(1) (env)   |
| Linked list                  | nested lists or environments                     | O(1) insert   |
| Sparse table / 2D cache      | `Matrix::sparseMatrix`, or env with `"i,j"` keys | —             |
| Bitmask DP state             | integer + `bitwAnd()`, `bitwOr()`, `bitwShiftL()`| O(1)          |

> 💡 **Performance tip:** Growing a vector inside a loop (`v <- c(v, x)`) is O(n) each time. **Pre-allocate** with `numeric(n)` / `vector("list", n)` instead.

### 13.2 Environment as a hash map

```r
h <- new.env(hash = TRUE)

h[["apple"]] <- 3                # or assign("apple", 3, envir = h)
h[["banana"]] <- 5
h[["apple"]]                     # 3
exists("cherry", envir = h, inherits = FALSE)   # FALSE
ls(h)                            # keys
mget(ls(h), envir = h)           # all key-value pairs
rm("apple", envir = h)           # delete
length(h)                        # number of keys

# Iterate
for (k in ls(h)) cat(k, "=", h[[k]], "\n")

# Environments are mutable references, unlike lists!
modify <- function(env) env$x <- 99
modify(h)
```

### 13.3 Memoization (top-down DP)

#### Fibonacci with an environment cache

```r
memo <- new.env()

fib <- function(n) {
  key <- as.character(n)
  if (!is.null(memo[[key]])) return(memo[[key]])
  if (n <= 1) return(n)
  res <- fib(n - 1) + fib(n - 2)
  memo[[key]] <- res
  res
}
fib(80)   # fast; naive recursion would never finish
```

#### Generic memoise wrapper (closure)

```r
memoize <- function(f) {
  cache <- new.env()
  function(...) {
    key <- paste(..., sep = ",")
    if (!is.null(cache[[key]])) return(cache[[key]])
    val <- f(...)
    cache[[key]] <- val
    val
  }
}

# Note: for recursion to benefit, the function must call the *memoised* version
fib2 <- memoize(function(n) if (n <= 1) n else fib2(n - 1) + fib2(n - 2))
fib2(60)
```

> Or use the ready-made package: `memoise::memoise(f)`.

### 13.4 Tabulation (bottom-up DP)

#### 1D table — Climbing stairs / Fibonacci

```r
fib_dp <- function(n) {
  if (n <= 1) return(n)
  dp <- numeric(n + 1)     # R is 1-indexed: dp[i + 1] stores f(i)
  dp[1] <- 0; dp[2] <- 1
  for (i in 3:(n + 1)) dp[i] <- dp[i - 1] + dp[i - 2]
  dp[n + 1]
}
```

#### 1D table — Coin change (minimum coins)

```r
coin_change <- function(coins, amount) {
  dp <- rep(Inf, amount + 1)     # dp[a + 1] = min coins for amount a
  dp[1] <- 0
  for (a in 1:amount) {
    for (c in coins) {
      if (a >= c) dp[a + 1] <- min(dp[a + 1], dp[a - c + 1] + 1)
    }
  }
  if (is.infinite(dp[amount + 1])) -1 else dp[amount + 1]
}
coin_change(c(1, 5, 10, 25), 63)   # 6
```

#### 2D table — Longest Common Subsequence (LCS)

```r
lcs <- function(a, b) {
  x <- strsplit(a, "")[[1]]; y <- strsplit(b, "")[[1]]
  n <- length(x); m <- length(y)
  dp <- matrix(0L, nrow = n + 1, ncol = m + 1)   # row/col 1 = empty prefix
  for (i in seq_len(n)) {
    for (j in seq_len(m)) {
      dp[i + 1, j + 1] <- if (x[i] == y[j]) dp[i, j] + 1L
                          else max(dp[i, j + 1], dp[i + 1, j])
    }
  }
  dp[n + 1, m + 1]
}
lcs("AGGTAB", "GXTXAYB")   # 4
```

#### 2D table — 0/1 Knapsack

```r
knapsack <- function(weights, values, W) {
  n <- length(weights)
  dp <- matrix(0, nrow = n + 1, ncol = W + 1)
  for (i in 1:n) {
    for (w in 0:W) {
      dp[i + 1, w + 1] <- dp[i, w + 1]                          # skip item
      if (weights[i] <= w) {
        dp[i + 1, w + 1] <- max(dp[i + 1, w + 1],
                                dp[i, w - weights[i] + 1] + values[i])   # take
      }
    }
  }
  dp[n + 1, W + 1]
}
knapsack(c(1, 3, 4, 5), c(1, 4, 5, 7), 7)   # 9
```

#### Edit distance (built-in!)

```r
adist("kitten", "sitting")   # 3  (Levenshtein distance)
```

### 13.5 Stack (LIFO)

```r
# Simple: vector-based
stack <- c()
stack <- c(stack, 1)            # push
stack <- c(stack, 2)
top <- stack[length(stack)]     # peek
stack <- stack[-length(stack)]  # pop

# Efficient & reusable: R6
library(R6)
Stack <- R6Class("Stack",
  public = list(
    items = NULL, n = 0,
    initialize = function(cap = 1000) self$items <- vector("list", cap),
    push = function(x) {
      if (self$n == length(self$items)) length(self$items) <- self$n * 2   # grow
      self$n <- self$n + 1
      self$items[[self$n]] <- x
      invisible(self)
    },
    pop  = function() { v <- self$items[[self$n]]; self$n <- self$n - 1; v },
    peek = function() self$items[[self$n]],
    empty = function() self$n == 0
  ))
```

### 13.6 Queue (FIFO) — for BFS

```r
Queue <- R6Class("Queue",
  public = list(
    items = list(), head = 1, tail = 0,
    enqueue = function(x) { self$tail <- self$tail + 1; self$items[[self$tail]] <- x; invisible(self) },
    dequeue = function() {
      v <- self$items[[self$head]]
      self$items[self$head] <- list(NULL)   # free memory
      self$head <- self$head + 1
      v
    },
    size  = function() self$tail - self$head + 1,
    empty = function() self$size() == 0
  ))

# BFS on a graph stored as a named list (adjacency list)
graph <- list(A = c("B", "C"), B = "D", C = "D", D = character(0))

bfs <- function(graph, start) {
  visited <- new.env()
  q <- Queue$new(); q$enqueue(start); visited[[start]] <- TRUE
  order <- character()
  while (!q$empty()) {
    node <- q$dequeue()
    order <- c(order, node)
    for (nb in graph[[node]]) {
      if (is.null(visited[[nb]])) { visited[[nb]] <- TRUE; q$enqueue(nb) }
    }
  }
  order
}
bfs(graph, "A")    # "A" "B" "C" "D"
```

### 13.7 The `collections` package (ready-made structures)

```r
install.packages("collections")
library(collections)

# Stack / Queue / Deque
s <- stack();  s$push(1); s$push(2); s$pop()      # 2
q <- queue();  q$push("a"); q$push("b"); q$pop()  # "a"
d <- deque();  d$push(1); d$pushleft(0); d$popleft()

# Priority queue (higher priority pops first)
pq <- priority_queue()
pq$push("low", priority = 1)
pq$push("high", priority = 10)
pq$pop()                      # "high"
pq$size()

# Dictionary (any key type: strings, numbers, even objects)
dict <- dict()
dict$set("x", 1)
dict$get("x")
dict$has("x")
dict$remove("x")
dict$keys(); dict$values()

# Ordered dictionary
od <- ordered_dict()
```

### 13.8 Dijkstra's algorithm (priority queue)

```r
library(collections)

dijkstra <- function(graph, source) {
  # graph: list of named numeric vectors: graph$A = c(B = 1, C = 4)
  dist <- setNames(rep(Inf, length(graph)), names(graph))
  dist[source] <- 0
  pq <- priority_queue()
  pq$push(source, priority = 0)           # use -dist since higher = first
  while (pq$size() > 0) {
    u <- pq$pop()
    for (v in names(graph[[u]])) {
      alt <- dist[u] + graph[[u]][v]
      if (alt < dist[v]) {
        dist[v] <- alt
        pq$push(v, priority = -alt)       # negate for min-heap behaviour
      }
    }
  }
  dist
}

g <- list(A = c(B = 1, C = 4), B = c(C = 2, D = 6), C = c(D = 3), D = numeric(0))
dijkstra(g, "A")    # A=0 B=1 C=3 D=6
```

### 13.9 Set operations

```r
a <- c(1, 2, 3); b <- c(3, 4)
union(a, b); intersect(a, b); setdiff(a, b)
is.element(2, a)            # same as 2 %in% a
unique(c(1, 1, 2))
duplicated(c(1, 1, 2))      # FALSE TRUE FALSE

# Fast set using an environment
seen <- new.env()
seen[["x"]] <- TRUE
!is.null(seen[["x"]])       # membership test
```

### 13.10 Bitmask DP helpers

```r
bitwAnd(5L, 3L)        # 1
bitwOr(5L, 3L)         # 7
bitwXor(5L, 3L)        # 6
bitwShiftL(1L, 3L)     # 8   (1 << 3)
bitwShiftR(8L, 2L)     # 2
bitwNot(0L)            # -1

# Check if bit i is set in mask
is_set <- function(mask, i) bitwAnd(mask, bitwShiftL(1L, i)) != 0L
# e.g. Travelling Salesman DP: dp[mask, last] stored in a matrix of size 2^n x n
```

### 13.11 Linked list (environment nodes)

```r
make_node <- function(val) {
  node <- new.env()
  node$val <- val
  node$nxt <- NULL
  node
}

a <- make_node(1); b <- make_node(2); c <- make_node(3)
a$nxt <- b; b$nxt <- c

cur <- a
while (!is.null(cur)) { print(cur$val); cur <- cur$nxt }   # 1 2 3
```

### 13.12 Sparse & specialised structures

```r
# Sparse matrix — big DP tables that are mostly zeros
library(Matrix)
sm <- sparseMatrix(i = c(1, 3), j = c(2, 4), x = c(5, 7), dims = c(1000, 1000))

# data.table — high-performance tables w/ reference semantics
library(data.table)
dt <- data.table(id = 1:5, val = rnorm(5))
dt[val > 0, .(mean_val = mean(val))]
setkey(dt, id)             # indexed lookup: dt[.(3)]
dt[, new := val * 2]       # modify in place (no copy)

# bit64, hash, fastmap — alternatives for hash tables
fastmap::fastmap()         # fast key-value store
```

### 13.13 Performance tips for DP in R

| ✅ Do                                              | ❌ Avoid                                 |
|----------------------------------------------------|------------------------------------------|
| Pre-allocate: `numeric(n)`, `matrix(0, n, m)`      | Growing vectors with `c(v, x)` in loops  |
| Use environments for large memo caches             | Named lists for caches with 10⁵+ keys    |
| Vectorise inner loops (`pmax`, `cumsum`, `outer`)  | Nested `for` when vector ops suffice     |
| Use integer indices (`1L`, `seq_len`)              | Character keys when numeric will do      |
| Use `Rcpp` for hot loops                           | Deep recursion (R default limit ≈ 5000 → `expression nesting`) |
| Rolling arrays to cut memory (keep only last row)  | Storing full 2D table if only prev row needed |

```r
options(expressions = 50000)    # raise recursion limit (use with care)
pmax(c(1, 5), c(3, 2))          # element-wise max: 3 5
outer(1:3, 1:3)                 # multiplication table, no loops
```

Rcpp example for a hot loop:

```r
library(Rcpp)
cppFunction("
int fib_cpp(int n) {
  if (n <= 1) return n;
  int a = 0, b = 1;
  for (int i = 2; i <= n; i++) { int t = a + b; a = b; b = t; }
  return b;
}")
fib_cpp(40)
```

---

## 14. Quick Reference Tables

### 14.1 Common functions by task

| Task                 | Functions |
|----------------------|-----------|
| Inspect              | `str()`, `summary()`, `head()`, `tail()`, `dim()`, `class()`, `names()` |
| Math / stats         | `sum`, `mean`, `median`, `sd`, `var`, `min`, `max`, `quantile`, `cor`, `round`, `abs`, `exp`, `log` |
| Sequences / sampling | `seq()`, `rep()`, `sample()`, `set.seed()`, `rnorm()`, `runif()` |
| Sorting / searching  | `sort()`, `order()`, `rev()`, `which()`, `match()`, `%in%` |
| Strings              | `paste`, `sprintf`, `substr`, `gsub`, `grepl`, `regmatches`, `strsplit`, `toupper`, `nchar` |
| Grouping             | `table`, `tapply`, `aggregate`, `split`, `dplyr::group_by` |
| Reshaping            | `merge`, `rbind`, `cbind`, `tidyr::pivot_longer/wider` |
| Plotting             | `plot`, `hist`, `barplot`, `boxplot`, `ggplot2::ggplot` |
| Modelling            | `lm`, `glm`, `t.test`, `predict`, `anova` |

### 14.2 Base R vs. Tidyverse

| Task              | Base R                              | Tidyverse                          |
|-------------------|-------------------------------------|------------------------------------|
| Filter rows       | `df[df$x > 1, ]`                    | `filter(df, x > 1)`                |
| Select columns    | `df[, c("a","b")]`                  | `select(df, a, b)`                 |
| New column        | `df$c <- df$a + df$b`               | `mutate(df, c = a + b)`            |
| Sort              | `df[order(df$x), ]`                 | `arrange(df, x)`                   |
| Group + summarise | `aggregate(x ~ g, df, mean)`        | `df %>% group_by(g) %>% summarise(m = mean(x))` |
| Join              | `merge(a, b, by = "id")`            | `left_join(a, b, by = "id")`       |
| Read CSV          | `read.csv()`                        | `readr::read_csv()`                |

### 14.3 Common gotchas

| Gotcha | Explanation | Fix |
|--------|-------------|-----|
| `1:length(x)` on empty `x` | gives `1 0`, not empty | use `seq_along(x)` |
| `T` / `F` | can be reassigned | use `TRUE` / `FALSE` |
| `0.1 + 0.2 == 0.3` is `FALSE` | floating-point error | `isTRUE(all.equal(...))` |
| `df[, 1]` drops to a vector | default `drop = TRUE` | `df[, 1, drop = FALSE]` |
| `if (c(TRUE, FALSE))` | error in R ≥ 4.2 (length > 1) | use `any()` / `all()` |
| `sapply` return type varies | may return list, vector, or matrix | use `vapply` |
| Lists copy on modify, environments don't | environments have reference semantics | choose intentionally |
| Package masks function | `dplyr::filter` masks `stats::filter` | `pkg::fn()` |
| Strings as factors | old R (< 4.0) default | `stringsAsFactors = FALSE` |

### 14.4 Naming conventions (style guide)

```r
my_variable   <- 1          # snake_case for variables/functions
calculate_sum <- function() {}
MyClass       <- R6Class()  # PascalCase for classes
MAX_ITER      <- 100        # SCREAMING_CASE for constants
```

---

## 📚 Further Resources

- **Advanced R** — Hadley Wickham: <https://adv-r.hadley.nz>
- **R for Data Science** — <https://r4ds.hadley.nz>
- **R Packages** — <https://r-pkgs.org>
- **R6 docs** — <https://r6.r-lib.org>
- **collections package** — <https://randy3k.github.io/collections>
- **box package** — <https://klmr.me/box>
- **CRAN Task Views** — <https://cran.r-project.org/web/views>

---

*Happy coding in R! 🚀*