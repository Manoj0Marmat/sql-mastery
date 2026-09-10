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
    -- first calculation
),
CTE2 AS (
    -- can reference CTE1
    SELECT * FROM CTE1 WHERE ...
),
CTE3 AS (
    -- can reference CTE1 and CTE2
    SELECT * FROM CTE2 JOIN CTE1 ON ...
)
SELECT * FROM CTE3
```

## Recursive CTE
```sql
WITH RecursiveCTE AS (
    -- PART 1: Anchor (runs ONCE — finds starting rows)
    SELECT EmpID, EmpName, ManagerID, 1 AS Level,
           CAST(NULL AS VARCHAR(50)) AS ManagerName
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    -- PART 2: Recursive member (runs repeatedly, references itself)
    SELECT e.EmpID, e.EmpName, e.ManagerID,
           r.Level + 1,
           r.EmpName AS ManagerName
    FROM Employees e
    INNER JOIN RecursiveCTE r ON e.ManagerID = r.EmpID
)
SELECT * FROM RecursiveCTE
```

## How Recursion Works Internally
```
Iteration 0 (Anchor):  finds root rows (no parent)
Iteration 1:           finds rows whose parent = anchor result
Iteration 2:           finds rows whose parent = iteration 1 result
...continues until no more matches...
Final result = UNION ALL of every iteration
```

## When to Use What

| Situation | Best Choice |
|---|---|
| Parent-child / org hierarchy | Recursive CTE |
| Reuse same subquery 2+ times | Regular CTE |
| Complex multi-step calculations | Chained CTEs |
| "Top N per group" filter | Window function + subquery |
| Simple, used only once | Inline subquery |

## Performance Rules for Recursive CTE
```sql
-- Default max recursion = 100 levels
-- Override if hierarchy is deeper:
OPTION (MAXRECURSION 500)
OPTION (MAXRECURSION 0)  -- unlimited (dangerous with cycles)

-- Guard against infinite loops:
WHERE r.Level < 50

-- Index the JOIN column — recursive member runs N times per level
CREATE INDEX IX_Employees_ManagerID ON Employees(ManagerID)

-- Select only needed columns — recursive spool lives in tempdb
```

## NULL Type Mismatch Fix
```sql
-- WRONG: NULL has no type — recursive member has VARCHAR(50), anchor has NULL
NULL AS ManagerName  -- Msg 240

-- CORRECT: match the type explicitly
CAST(NULL AS VARCHAR(50)) AS ManagerName
```

## CTE vs Subquery Performance
```sql
-- CTE and correlated subquery often produce IDENTICAL plans
-- Same QueryPlanHash = optimizer rewrites both identically
-- Picking CTE over subquery is about readability, not speed
```
