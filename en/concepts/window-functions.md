# Window Functions

## Syntax
```sql
FUNCTION() OVER (
    PARTITION BY column   -- optional: reset per group
    ORDER BY column       -- sort order for calculation
    ROWS BETWEEN ...      -- optional: frame
)
```

## Key Rule
```
ORDER BY inside OVER()  → controls the calculation
ORDER BY at end         → controls display order
These are INDEPENDENT
```

## Ranking Functions
```sql
ROW_NUMBER() OVER (ORDER BY salary DESC)  -- unique always: 1,2,3,4,5
RANK()        OVER (ORDER BY salary DESC)  -- gaps on ties: 1,2,2,4,5
DENSE_RANK()  OVER (ORDER BY salary DESC)  -- no gaps: 1,2,2,3,4
NTILE(4)      OVER (ORDER BY salary DESC)  -- split into N equal groups
```

## Side-by-side comparison
```
Name   Salary  ROW_NUMBER  RANK  DENSE_RANK
────────────────────────────────────────────
Alice  90000       1         1       1
Bob    80000       2         2       2
Carol  80000       3         2       2    ← same rank (tie)
Dave   70000       4         4       3    ← RANK gap (4), DENSE_RANK no gap (3)
Eve    60000       5         5       4
```

## When to use which
| Problem | Use |
|---|---|
| Dedup — 1 row per group | ROW_NUMBER |
| Top N — ties must not add rows | RANK |
| Top N — ties show same rank | DENSE_RANK |
| Percentile groups | NTILE(4/10/100) |

## RANK vs DENSE_RANK for top N
```
Tie at rank 2:
RANK:        1, 2, 2, 4   → WHERE Rank <= 3 = 3 rows  (correct)
DENSE_RANK:  1, 2, 2, 3   → WHERE Rank <= 3 = 4 rows  (extra row!)
Use RANK for top N problems.
```

## PARTITION BY
```sql
-- Rank resets per department
DENSE_RANK() OVER (PARTITION BY Department ORDER BY Salary DESC)

-- No PARTITION BY = whole table is one window
DENSE_RANK() OVER (ORDER BY Salary DESC)
```

## AVG / SUM Over Window
```sql
-- Department average on every row (single scan)
AVG(salary) OVER (PARTITION BY Department)

-- Company average on every row
AVG(salary) OVER ()

-- Running total
SUM(salary) OVER (ORDER BY salary DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

## LAG / LEAD
```sql
LAG(salary)  OVER (ORDER BY date)  -- previous row value
LEAD(salary) OVER (ORDER BY date)  -- next row value

-- Always ORDER BY date column, not value column
-- First row LAG = NULL, last row LEAD = NULL
```

## Can't mix window + plain aggregate
```sql
-- WRONG
SELECT AVG(salary) OVER (PARTITION BY dept), AVG(salary) FROM Employees

-- FIX: window in subquery, aggregate in outer
SELECT Department, DeptAvg, CompanyAvg
FROM (
    SELECT Department,
        AVG(CAST(salary AS DECIMAL(10,2))) OVER (PARTITION BY Department) AS DeptAvg,
        AVG(CAST(salary AS DECIMAL(10,2))) OVER () AS CompanyAvg
    FROM Employees
) t
GROUP BY Department, DeptAvg, CompanyAvg
```

## Performance
```
Window function = single table scan
CTE + JOIN = 2 scans
Top N per group → always try window function first
P15 real numbers: window 0.0147 vs CTE 0.0261 (44% cheaper)
```

## Quick reference
```
ROW_NUMBER → unique, no ties
RANK       → gaps on ties (Olympic style)
DENSE_RANK → no gaps on ties
NTILE(n)   → n equal buckets
LAG        → previous row
LEAD       → next row
FIRST_VALUE / LAST_VALUE → first/last in partition (LAST_VALUE needs full frame)
```
