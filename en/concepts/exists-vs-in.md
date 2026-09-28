# EXISTS vs IN

## What both do
Check if rows exist in a subquery. Same result — different execution.

```sql
-- IN
SELECT * FROM Employees E
WHERE E.DepartmentID IN (SELECT DepartmentID FROM Departments)

-- EXISTS
SELECT * FROM Employees E
WHERE EXISTS (SELECT 1 FROM Departments D WHERE D.DepartmentID = E.DepartmentID)
```

## Key difference
```
IN      → subquery executes fully, loads entire result set into memory
EXISTS  → stops at first match (short circuit) → faster on large subqueries
```

## When each wins
```
EXISTS faster: large subquery result set, NULL values possible
IN faster:     small static list — IN (1, 2, 3)
```

## NULL trap — critical
```sql
-- NOT IN with NULL → returns ZERO rows (silent bug)
SELECT * FROM Employees
WHERE DeptID NOT IN (SELECT DeptID FROM Departments)
-- If ANY row in Departments has NULL DeptID → result = 0 rows
-- NULL comparison = UNKNOWN → all rows filtered out

-- NOT EXISTS with NULL → correct result
SELECT * FROM Employees E
WHERE NOT EXISTS (SELECT 1 FROM Departments D WHERE D.DeptID = E.DeptID)
```

**Rule: Always use NOT EXISTS instead of NOT IN when subquery column can have NULLs.**

## EXISTS syntax
```sql
EXISTS (SELECT 1 FROM ...)   -- SELECT 1 = convention, means "just check existence"
EXISTS (SELECT * FROM ...)   -- also works, SELECT 1 is cleaner
```

## Quick reference
```
Large subquery           → EXISTS
Small static list        → IN (1,2,3)
NOT check + possible NULL → NOT EXISTS (always)
Existence only           → EXISTS
```
