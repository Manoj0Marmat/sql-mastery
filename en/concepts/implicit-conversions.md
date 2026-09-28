# Implicit Conversions — Silent Index Killer

## What is an Implicit Conversion?
SQL Server silently converts a column's data type to match your literal — causing full table scan even when an index exists.

## Most common scenarios
```sql
-- VARCHAR column, INT literal → index unusable
-- Column: EmployeeCode VARCHAR(20), index exists
WHERE EmployeeCode = 12345      -- BAD  → full scan
WHERE EmployeeCode = '12345'    -- GOOD → index seek

-- NVARCHAR column, VARCHAR literal → conversion
WHERE CustomerName = 'Ravi'     -- BAD  → NVARCHAR converts to VARCHAR
WHERE CustomerName = N'Ravi'    -- GOOD → N prefix = NVARCHAR literal

-- Function on column → index unusable
WHERE CAST(OrderDate AS DATE) = '2024-01-15'   -- BAD → index unusable
WHERE OrderDate = '2024-01-15'                  -- GOOD → direct compare
```

## How to detect
```sql
-- In execution plan: look for CONVERT_IMPLICIT warning (yellow triangle)
-- Shows: "Type conversion in expression may affect CardinalityEstimate"

-- Via DMV
SELECT TOP 20 qs.total_logical_reads, qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) qp
WHERE CAST(qp.query_plan AS NVARCHAR(MAX)) LIKE '%CONVERT_IMPLICIT%'
ORDER BY qs.total_logical_reads DESC
```

## Data type precedence — who converts to what?
```
INT > VARCHAR → VARCHAR column converts to INT (bad — column-level conversion)
NVARCHAR > VARCHAR → VARCHAR literal promotes to NVARCHAR (bad — column-level)
```

## Fix checklist
```
1. Match literal type to column type (quotes for VARCHAR, N'' for NVARCHAR)
2. Never apply functions to indexed columns in WHERE
3. Fix bad column types if possible (VARCHAR date → DATE)
4. Check CONVERT_IMPLICIT in execution plans after changes
```

## Real impact
```
SLCProject — billion-row table
WHERE EmployeeCode = 12345    → full scan  → 50 seconds
WHERE EmployeeCode = '12345'  → index seek → 0.01 seconds
One missing quote = 5000x slower
```

## Quick reference
```
VARCHAR = 'value'   → correct (quotes!)
NVARCHAR = N'value' → correct (N prefix!)
= function(col)     → never in WHERE on indexed column
CONVERT_IMPLICIT    → find in execution plan warnings
```
