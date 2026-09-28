# CTEs (Common Table Expressions)

## Basic Syntax
```sql
WITH CTE_Name AS (
    SELECT ...
)
SELECT * FROM CTE_Name
```

## Chained CTEs
```sql
WITH CTE1 AS (
    SELECT ...
),
CTE2 AS (
    SELECT * FROM CTE1 WHERE ...   -- can reference CTE1
),
CTE3 AS (
    SELECT * FROM CTE2 JOIN CTE1 ON ...   -- can reference both
)
SELECT * FROM CTE3
```

## Recursive CTE
```sql
WITH RecursiveCTE AS (
    -- Anchor: runs ONCE, finds starting rows
    SELECT EmpID, EmpName, ManagerID, 1 AS Level,
           CAST(NULL AS VARCHAR(50)) AS ManagerName
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    -- Recursive member: runs repeatedly, references itself
    SELECT e.EmpID, e.EmpName, e.ManagerID,
           r.Level + 1,
           r.EmpName AS ManagerName
    FROM Employees e
    INNER JOIN RecursiveCTE r ON e.ManagerID = r.EmpID
)
SELECT * FROM RecursiveCTE
```

## How Recursion Works
```
Iteration 0 (Anchor):  finds root rows (ManagerID IS NULL)
Iteration 1:           finds rows whose ManagerID = anchor result
Iteration 2:           finds rows whose ManagerID = iteration 1 result
...continues until no more matches...
Final result = UNION ALL of every iteration
```

## When to Use What
| Situation | Use |
|---|---|
| Parent-child / org hierarchy | Recursive CTE |
| Reuse same subquery 2+ times | Regular CTE |
| Complex multi-step calculations | Chained CTEs |
| Simple, used once | Inline subquery |

## Performance Rules
```sql
-- Default max recursion = 100 levels
OPTION (MAXRECURSION 500)   -- override if hierarchy deeper
OPTION (MAXRECURSION 0)     -- unlimited (dangerous with cycles)

-- Guard against infinite loops
WHERE r.Level < 50

-- Index the recursive JOIN column
CREATE INDEX IX_Employees_ManagerID ON Employees(ManagerID)
```

## NULL Type Fix
```sql
-- WRONG: NULL has no type → Msg 240
NULL AS ManagerName

-- CORRECT: match type explicitly
CAST(NULL AS VARCHAR(50)) AS ManagerName
```

## CTE vs Subquery Performance
```
CTE and correlated subquery → often identical plans (same QueryPlanHash)
Optimizer rewrites both identically
Choose CTE for readability, not speed
```

## Quick reference
```
WITH name AS (...)         → define CTE
Chained: WITH a AS (...), b AS (...) SELECT FROM b
Recursive: anchor UNION ALL recursive member
MAXRECURSION hint          → control depth
CAST(NULL AS type)         → fix type mismatch in anchor
```
