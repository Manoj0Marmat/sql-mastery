# CROSS APPLY / OUTER APPLY

## What is APPLY?
Runs a subquery or table-valued function **for each row** of the outer table.
The subquery can reference the outer row's columns — regular JOIN cannot do this.

```
CROSS APPLY  = like INNER JOIN — no match = row excluded
OUTER APPLY  = like LEFT JOIN  — no match = row included (NULLs)
```

## Why APPLY exists — JOIN limitation
```sql
-- WRONG — cannot reference outer row inside JOIN subquery
SELECT E.EmpName, S.SkillName
FROM Employees E
JOIN (SELECT TOP 1 SkillName FROM Skills WHERE EmpID = E.EmpID) S  -- ERROR

-- CORRECT — APPLY handles per-row correlated subquery
SELECT E.EmpName, T.SkillName
FROM Employees E
CROSS APPLY (
    SELECT TOP 1 SkillName FROM Skills
    WHERE EmpID = E.EmpID
    ORDER BY Rating DESC
) T
```

## Syntax
```sql
-- CROSS APPLY — employees with no skills excluded
SELECT E.EmpName, T.TopSkill
FROM Employees E
CROSS APPLY (
    SELECT TOP 1 SkillName AS TopSkill
    FROM Skills S
    WHERE S.EmpID = E.EmpID
    ORDER BY Rating DESC
) T

-- OUTER APPLY — employees with no skills included (NULL)
SELECT E.EmpName, T.TopSkill
FROM Employees E
OUTER APPLY (
    SELECT TOP 1 SkillName AS TopSkill
    FROM Skills S
    WHERE S.EmpID = E.EmpID
    ORDER BY Rating DESC
) T
```

## Real production use — DMV scripts
```sql
-- Every DBA monitoring script uses CROSS APPLY
SELECT TOP 10
    qs.total_elapsed_time,
    qt.text AS QueryText
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_elapsed_time DESC
-- sys.dm_exec_sql_text = table-valued function called per row
```

## When to use
```
✓ TOP N per row (top 3 orders per customer)
✓ Table-valued function per row (DMV queries)
✓ Correlated subquery returning multiple columns
```

## Quick reference
| | CROSS APPLY | OUTER APPLY |
|---|---|---|
| No match | Row excluded | Row included (NULL) |
| Equivalent | INNER JOIN | LEFT JOIN |
| Use when | Match guaranteed | Match optional |
