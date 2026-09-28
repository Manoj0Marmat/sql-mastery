# Temp Tables vs Table Variables vs CTEs

## Three options for intermediate data
```
#TempTable      → physical table in tempdb, session-scoped
@TableVariable  → memory (mostly), batch-scoped
CTE             → no storage, named subquery only
```

## Side-by-side comparison
| Feature | #TempTable | @TableVariable | CTE |
|---|---|---|---|
| Storage | tempdb (disk) | Memory (mostly) | None |
| Scope | Session / procedure | Batch only | Query only |
| Indexes | Yes — full support | Limited (PK only) | No |
| Statistics | Yes | No | No |
| Transaction aware | Yes | No | N/A |
| Reuse across queries | Yes | No | No |

## Syntax
```sql
-- CTE
WITH CTE AS (SELECT ...)
SELECT * FROM CTE

-- @TableVariable
DECLARE @Emps TABLE (EmpID INT, EmpName VARCHAR(100))
INSERT INTO @Emps SELECT EmpID, EmpName FROM Employees WHERE ...
SELECT * FROM @Emps

-- #TempTable
CREATE TABLE #Emps (EmpID INT, EmpName VARCHAR(100))
INSERT INTO #Emps SELECT EmpID, EmpName FROM Employees WHERE ...
CREATE INDEX IX_Emps_ID ON #Emps(EmpID)   -- indexes supported!
SELECT * FROM #Emps
DROP TABLE IF EXISTS #Emps
```

## When to use which
```
CTE           → simple readable subquery, one-time use, no reuse needed
@TableVariable → small result set (< ~1000 rows), short batch, no indexes needed
#TempTable    → large data, multiple queries need it, indexes needed, ETL steps
```

## Performance trap — @TableVariable
```
@TableVariable has NO statistics
Optimizer always assumes 1 row regardless of actual count
100,000 rows inserted → optimizer still plans for 1 row → bad plan

#TempTable has statistics → optimizer knows actual row count → better plan

Rule: result set > ~1000 rows → use #TempTable, not @TableVariable
```

## Quick reference
```
Just this query?              → CTE
Small data, short batch?      → @TableVariable
Large data, indexes, multi-step? → #TempTable
```
