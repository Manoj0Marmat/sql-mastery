# UNION vs UNION ALL

## What both do
Stack rows from two SELECT statements vertically.

```
UNION     = stack + remove duplicates (runs DISTINCT internally) → slower
UNION ALL = stack all rows, keep duplicates                      → faster
```

## Syntax
```sql
SELECT EmpName FROM Employees_India
UNION ALL                              -- prefer this
SELECT EmpName FROM Employees_US

SELECT CustomerID FROM CRM_Customers
UNION                                  -- only when duplicates must be removed
SELECT CustomerID FROM ERP_Customers
```

## Rules — both must follow
```
1. Same number of columns in each SELECT
2. Compatible data types per column position
3. Column names come from first SELECT
```

## When to use which
```
UNION ALL → always prefer — faster, no sort, no dedup
            Use when duplicates are impossible OR don't matter

UNION     → only when duplicates definitely must be removed
```

## Related — EXCEPT and INTERSECT
```sql
-- EXCEPT — rows in first set, not in second
SELECT CustomerID FROM AllCustomers
EXCEPT
SELECT CustomerID FROM ActiveCustomers    -- inactive customers

-- INTERSECT — rows in both sets
SELECT CustomerID FROM CRM_Customers
INTERSECT
SELECT CustomerID FROM ERP_Customers     -- customers in both systems
```

## Quick reference
```
UNION ALL  = A + B (keep all, fast)
UNION      = A + B - duplicates (slow, sort+dedup)
EXCEPT     = A - B (remove B from A)
INTERSECT  = A ∩ B (common only)
```
