# Window Functions

## Syntax
```sql
FUNCTION() OVER (
    PARTITION BY column   -- optional: reset calculation per group
    ORDER BY column       -- sort order for calculation
    ROWS BETWEEN ...      -- optional: frame specification
)
```

## Key Rule
```
ORDER BY inside OVER()  → controls the calculation
ORDER BY at end         → controls display order
These are INDEPENDENT of each other
```

## Ranking Functions
```sql
ROW_NUMBER() OVER (ORDER BY salary DESC)  -- always unique: 1,2,3,4,5...
RANK()        OVER (ORDER BY salary DESC)  -- gaps on ties: 1,2,3,4,4,6...
DENSE_RANK()  OVER (ORDER BY salary DESC)  -- no gaps: 1,2,3,4,4,5...
```

## When to Use Which
| Function | Use when |
|---|---|
| ROW_NUMBER | Pagination, dedup — need unique ID per row |
| RANK | Competition style — show gap after ties |
| DENSE_RANK | Salary levels — no gaps wanted |

## PARTITION BY
```sql
-- Rank resets per department
DENSE_RANK() OVER (PARTITION BY Department ORDER BY Salary DESC)

-- No PARTITION BY = whole table is one window
DENSE_RANK() OVER (ORDER BY Salary DESC)
```

## AVG / SUM Over Window
```sql
-- Department average shown on every row (single table scan)
AVG(salary) OVER (PARTITION BY Department)

-- Company-wide average shown on every row
AVG(salary) OVER ()

-- Running total
SUM(salary) OVER (ORDER BY salary DESC)
```

## ROWS vs RANGE
```sql
-- RANGE (default): ties get same cumulative value
SUM(salary) OVER (ORDER BY salary DESC)

-- ROWS: each row strictly separate — correct running total
SUM(salary) OVER (
    ORDER BY salary DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

## Frame Options
```
UNBOUNDED PRECEDING  → from very first row in the window
N PRECEDING          → N rows back from current row
CURRENT ROW          → this row
N FOLLOWING          → N rows ahead
UNBOUNDED FOLLOWING  → to very last row in the window
```

## Common Patterns
```sql
-- Running total
SUM(salary) OVER (
    ORDER BY salary DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)

-- 3-row moving average
AVG(CAST(salary AS DECIMAL(10,2))) OVER (
    ORDER BY salary DESC
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)

-- Previous row value
LAG(salary) OVER (ORDER BY salary DESC)

-- Next row value
LEAD(salary) OVER (ORDER BY salary DESC)
```

## Can't Mix Window + Plain Aggregate
```sql
-- WRONG: SQL Server doesn't know row count when mixing
SELECT AVG(salary) OVER (PARTITION BY dept), AVG(salary)
FROM Employees

-- FIX: window functions in subquery, aggregate in outer query
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
- Window function = single table scan
- CTE + JOIN approach = 2 scans
- "Top N per group" → always try window function first
- Real number: P15 — window cost 0.0147 vs CTE cost 0.0261 (44% cheaper)
