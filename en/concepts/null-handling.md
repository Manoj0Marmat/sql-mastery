# NULL Handling

## What is NULL?
NULL = unknown value. Not zero, not empty string, not false. Unknown.

```sql
NULL = NULL   → FALSE   (unknown = unknown = still unknown)
NULL + 5      → NULL    (unknown + anything = unknown)
NULL <> NULL  → FALSE
```

## Three functions

### ISNULL — SQL Server specific, 2 values
```sql
ISNULL(Salary, 0)           -- NULL → 0, non-NULL → original value
```

### COALESCE — ANSI standard, multiple values
```sql
COALESCE(NickName, FirstName, 'Unknown')   -- returns first non-NULL
```

### NULLIF — return NULL if equal
```sql
NULLIF(Units, 0)            -- Units = 0 → NULL, else → Units
Revenue / NULLIF(Units, 0)  -- prevents divide-by-zero error
```

## ISNULL vs COALESCE
```
ISNULL   → 2 values, SQL Server only, slightly faster
COALESCE → multiple values, ANSI standard, portable
Prefer COALESCE for new code.
```

## NULL comparison — common mistakes
```sql
-- WRONG — always 0 rows
WHERE Salary = NULL
WHERE Salary <> NULL

-- CORRECT
WHERE Salary IS NULL
WHERE Salary IS NOT NULL
```

## NULL in aggregates
```sql
COUNT(*)       -- counts all rows (includes NULLs)
COUNT(Salary)  -- counts only non-NULL values
SUM / AVG / MIN / MAX  -- all ignore NULLs automatically
```

## NOT IN with NULL — silent bug
```sql
-- Returns ZERO rows if subquery has any NULL
WHERE DeptID NOT IN (SELECT DeptID FROM Departments)

-- Safe alternative
WHERE NOT EXISTS (SELECT 1 FROM Departments D WHERE D.DeptID = E.DeptID)
```

## Quick reference
```
ISNULL(col, default)     → replace NULL (2 values, SQL Server)
COALESCE(v1, v2, v3)     → first non-NULL (multiple values, standard)
NULLIF(v1, v2)           → NULL if equal (divide-by-zero guard)
IS NULL / IS NOT NULL    → correct NULL check (= NULL never works)
COUNT(*) vs COUNT(col)   → col version skips NULLs
NOT IN + NULL subquery   → 0 rows → use NOT EXISTS
```
