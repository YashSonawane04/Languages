# DAX (Data Analysis Expressions) — Concepts Cheat Sheet

A quick-reference guide to the core concepts and syntax used in DAX, the formula language behind Power BI, Excel Power Pivot, and SSAS Tabular models.

---

## 1. What is DAX?

DAX is a **functional language** made up of formulas and expressions used to work with data stored in tables (columns and measures). Every DAX formula starts with an `=` sign (in the UI) and is built from:

- **Functions** — `SUM()`, `IF()`, `CALCULATE()`, etc.
- **Operators** — `+`, `-`, `*`, `/`, `&`, `=`, `>`, etc.
- **Values** — numbers, text, dates
- **References** — to tables and columns

```dax
Total Sales = SUM(Sales[SalesAmount])
```

---

## 2. Calculated Columns vs Measures vs Tables

| Type | Evaluated | Stored? | Context Used | Example |
|---|---|---|---|---|
| **Calculated Column** | Row by row, at refresh time | Yes (in the table) | Row Context | `Profit = Sales[Revenue] - Sales[Cost]` |
| **Measure** | On the fly, when used in a visual | No (computed dynamically) | Filter Context | `Total Profit = SUM(Sales[Profit])` |
| **Calculated Table** | Once, at refresh time | Yes (as a new table) | None (whole expression) | `MyTable = FILTER(Sales, Sales[Qty] > 10)` |

**Syntax examples:**

```dax
-- Calculated Column
Profit = Sales[Revenue] - Sales[Cost]

-- Measure
Total Profit = SUM(Sales[Profit])

-- Calculated Table
BigOrders = FILTER(Sales, Sales[OrderQty] > 10)
```

---

## 3. Basic Syntax Rules

| Element | Syntax | Example |
|---|---|---|
| Table reference | `'Table Name'` or `TableName` | `'Sales'[Amount]` |
| Column reference | `Table[Column]` | `Sales[Amount]` |
| Measure reference | `[Measure Name]` | `[Total Sales]` |
| Comment (single line) | `--` or `//` | `-- this is a comment` |
| Comment (multi-line) | `/* ... */` | `/* multi line */` |
| String concatenation | `&` | `"Hello " & "World"` |
| Line continuation | Just press Enter (whitespace ignored) | — |

```dax
Full Name = Customer[FirstName] & " " & Customer[LastName]
```

---

## 4. Operators

| Category | Operators | Example |
|---|---|---|
| Arithmetic | `+  -  *  /  ^` | `Price * Qty` |
| Comparison | `=  ==  <>  >  <  >=  <=` | `Sales[Qty] > 10` |
| Text concatenation | `&` | `[First] & [Last]` |
| Logical | `&&` (AND), `\|\|` (OR), `NOT` | `A > 1 && B < 5` |

```dax
IsHighValue = IF(Sales[Amount] > 1000 && Sales[Qty] > 5, "Yes", "No")
```

---

## 5. Aggregation Functions

| Function | Purpose | Syntax |
|---|---|---|
| `SUM` | Adds a column | `SUM(Table[Column])` |
| `AVERAGE` | Mean of a column | `AVERAGE(Table[Column])` |
| `MIN` / `MAX` | Smallest / largest value | `MIN(Table[Column])` |
| `COUNT` | Counts numeric/date values | `COUNT(Table[Column])` |
| `COUNTA` | Counts non-blank values | `COUNTA(Table[Column])` |
| `COUNTROWS` | Counts rows in a table | `COUNTROWS(Table)` |
| `DISTINCTCOUNT` | Counts unique values | `DISTINCTCOUNT(Table[Column])` |
| `SUMX` | Row-by-row sum with expression | `SUMX(Table, Table[Qty]*Table[Price])` |
| `AVERAGEX` | Row-by-row average | `AVERAGEX(Table, Table[A]+Table[B])` |

```dax
Total Revenue = SUMX(Sales, Sales[Qty] * Sales[UnitPrice])
```

> **Tip:** The `X` suffix (`SUMX`, `AVERAGEX`, `MAXX`...) means the function iterates row-by-row over a table before aggregating — these are called **iterator functions**.

---

## 6. CALCULATE — The Most Important Function

`CALCULATE` changes the **filter context** of an expression. It's the heart of DAX.

```dax
CALCULATE(<expression>, <filter1>, <filter2>, ...)
```

**Example:**

```dax
Sales 2024 = CALCULATE(SUM(Sales[Amount]), Sales[Year] = 2024)

Sales Excl. Chairs = CALCULATE(
    SUM(Sales[Amount]),
    Sales[Category] <> "Chairs"
)
```

Filters passed inside `CALCULATE` **replace** existing filters on that column (unless combined with functions like `KEEPFILTERS`).

---

## 7. Context: Row Context vs Filter Context

| Context Type | Where it Applies | Description |
|---|---|---|
| **Row Context** | Calculated columns, iterator functions (`SUMX`, `FILTER`...) | "Current row" is known |
| **Filter Context** | Measures, visuals, slicers | Set of filters applied before calculation |

```dax
-- Row context example (evaluated per row)
Margin = Sales[Revenue] - Sales[Cost]

-- Filter context example (depends on report filters/slicers)
Total Sales = SUM(Sales[Revenue])
```

Use `CALCULATE` to convert row context into filter context — this is called **context transition**.

---

## 8. Filter Functions

| Function | Purpose | Syntax |
|---|---|---|
| `FILTER` | Returns a filtered table | `FILTER(Table, <condition>)` |
| `ALL` | Removes filters from a table/column | `ALL(Table[Column])` |
| `ALLEXCEPT` | Removes all filters except specified ones | `ALLEXCEPT(Table, Table[Col1])` |
| `ALLSELECTED` | Keeps filters from outside the current visual | `ALLSELECTED(Table)` |
| `KEEPFILTERS` | Adds to filters instead of replacing | `CALCULATE(..., KEEPFILTERS(...))` |
| `REMOVEFILTERS` | Explicitly clears filters | `REMOVEFILTERS(Table)` |

```dax
% of Total = 
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALL(Sales))
)
```

---

## 9. Logical Functions

| Function | Syntax | Example |
|---|---|---|
| `IF` | `IF(<condition>, <true>, <false>)` | `IF(Sales[Qty]>10, "Bulk", "Retail")` |
| `SWITCH` | `SWITCH(<expr>, val1, res1, val2, res2, ..., default)` | see below |
| `AND` / `OR` | `AND(a,b)` / `OR(a,b)` | `AND(A>1, B<5)` |
| `IFERROR` | `IFERROR(<value>, <alt>)` | `IFERROR(1/0, 0)` |

```dax
Sales Tier = 
SWITCH(
    TRUE(),
    Sales[Amount] > 1000, "Gold",
    Sales[Amount] > 500,  "Silver",
    "Bronze"
)
```

---

## 10. Text Functions

| Function | Purpose | Example |
|---|---|---|
| `CONCATENATE` | Joins two strings | `CONCATENATE("A","B")` |
| `LEFT` / `RIGHT` | Substring from start/end | `LEFT(Text,3)` |
| `MID` | Substring from middle | `MID(Text,2,5)` |
| `LEN` | Length of text | `LEN(Text)` |
| `UPPER` / `LOWER` | Case conversion | `UPPER(Text)` |
| `TRIM` | Removes extra spaces | `TRIM(Text)` |
| `FORMAT` | Formats value as text | `FORMAT(Sales[Date], "MMM YYYY")` |

```dax
Order Label = "Order #" & Sales[OrderID] & " - " & FORMAT(Sales[Date], "dd/mm/yyyy")
```

---

## 11. Date & Time Intelligence Functions

| Function | Purpose | Syntax |
|---|---|---|
| `TODAY` / `NOW` | Current date/time | `TODAY()` |
| `YEAR` / `MONTH` / `DAY` | Extract date parts | `YEAR(Sales[Date])` |
| `DATEADD` | Shift dates by interval | `DATEADD(DateCol, -1, YEAR)` |
| `SAMEPERIODLASTYEAR` | Prior year same period | `SAMEPERIODLASTYEAR(DateCol)` |
| `TOTALYTD` / `TOTALQTD` / `TOTALMTD` | Running totals | `TOTALYTD(SUM(Sales[Amt]), DateCol)` |
| `DATESYTD` | Table of YTD dates | `DATESYTD(DateCol)` |
| `DATESBETWEEN` | Table of dates in a range | `DATESBETWEEN(DateCol, Start, End)` |

```dax
Sales LY = CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Date'[Date]))

Sales YTD = TOTALYTD(SUM(Sales[Amount]), 'Date'[Date])
```

> **Note:** Time intelligence functions require a proper **Date table** marked as a Date Table, with a continuous, unique date column.

---

## 12. Table Functions

| Function | Purpose | Syntax |
|---|---|---|
| `FILTER` | Filters a table | `FILTER(Table, condition)` |
| `ALL` | Returns all rows (ignores filters) | `ALL(Table)` |
| `VALUES` | Distinct values of a column | `VALUES(Table[Column])` |
| `DISTINCT` | Distinct values (table) | `DISTINCT(Table[Column])` |
| `RELATED` | Pulls value from related table (many side) | `RELATED(Table[Column])` |
| `RELATEDTABLE` | Related rows (one side) | `RELATEDTABLE(Table)` |
| `SUMMARIZE` | Groups/aggregates a table | `SUMMARIZE(Table, GroupCol, "Total", SUM(Table[Amt]))` |
| `UNION` | Combines tables | `UNION(Table1, Table2)` |

```dax
Category Name = RELATED(Products[Category])

Sales Summary = 
SUMMARIZE(
    Sales,
    Products[Category],
    "Total Sales", SUM(Sales[Amount])
)
```

---

## 13. Variables (`VAR` / `RETURN`)

Variables make DAX more readable and improve performance by calculating a value once and reusing it.

```dax
Profit Margin % = 
VAR TotalRevenue = SUM(Sales[Revenue])
VAR TotalCost = SUM(Sales[Cost])
RETURN
    DIVIDE(TotalRevenue - TotalCost, TotalRevenue)
```

**Syntax pattern:**

```dax
<Measure Name> =
VAR <VariableName> = <expression>
VAR <VariableName2> = <expression>
RETURN
    <final expression using variables>
```

---

## 14. DIVIDE — Safe Division

Always prefer `DIVIDE()` over `/` to avoid divide-by-zero errors.

```dax
DIVIDE(<numerator>, <denominator>, [<alternate result>])
```

```dax
Profit Margin = DIVIDE(SUM(Sales[Profit]), SUM(Sales[Revenue]), 0)
```

---

## 15. Common DAX Patterns

| Goal | Pattern |
|---|---|
| Running total | `CALCULATE(SUM(Sales[Amt]), FILTER(ALL('Date'), 'Date'[Date] <= MAX('Date'[Date])))` |
| Rank | `RANKX(ALL(Products), [Total Sales])` |
| % of parent category | `DIVIDE([Sales], CALCULATE([Sales], ALL(Products[Category])))` |
| Distinct customer count | `DISTINCTCOUNT(Sales[CustomerID])` |
| Top N filter | `TOPN(5, Products, [Total Sales])` |

```dax
Rank by Sales = RANKX(ALL(Products[ProductName]), [Total Sales])
```

---

## 16. Quick Syntax Summary

```dax
-- Basic measure
Total Sales = SUM(Sales[Amount])

-- Conditional
Status = IF(Sales[Amount] > 1000, "High", "Low")

-- Filter context change
Sales North = CALCULATE(SUM(Sales[Amount]), Sales[Region] = "North")

-- Variables
Growth % =
VAR CurrYear = [Total Sales]
VAR PrevYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN DIVIDE(CurrYear - PrevYear, PrevYear)
```

---

### Key Takeaways

- Measures are dynamic; calculated columns are static and stored.
- `CALCULATE` is the backbone of nearly all advanced DAX — it manipulates filter context.
- Use `VAR`/`RETURN` for readability and performance.
- Always use `DIVIDE()` instead of `/`.
- A well-structured **Date table** is essential for time intelligence functions.