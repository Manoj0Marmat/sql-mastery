# Implicit Conversions — Hindi in English

## Address mismatch analogy

Column mein address English mein hai: "12, MG Road, Bangalore"
Tum search kar rahe ho Hindi mein: "12, एमजी रोड, बेंगलुरु"

Database ko pehle **saare addresses translate** karne honge phir compare hoga.
Har row pe extra kaam. **Index kisi kaam ka nahi.**

## Kya hota hai exactly?

```sql
-- Column: EmployeeCode VARCHAR(20), index exist karta hai
WHERE EmployeeCode = 12345      -- INT literal
-- SQL Server: "Ye VARCHAR hai, INT se compare karna hai"
-- Solution: poore column ko INT mein convert karo
-- Result: INDEX USELESS → full table scan!

-- FIX: quotes lagao
WHERE EmployeeCode = '12345'    -- VARCHAR literal → index seek → fast!
```

## Common scenarios

```sql
-- VARCHAR column + INT literal → bad
WHERE EmpCode = 12345          -- BAD
WHERE EmpCode = '12345'        -- GOOD (quotes!)

-- NVARCHAR column + VARCHAR literal → bad
WHERE CustomerName = 'Ravi'    -- BAD (N prefix missing)
WHERE CustomerName = N'Ravi'   -- GOOD (N prefix = NVARCHAR literal)

-- Function on column → bad
WHERE CAST(OrderDate AS DATE) = '2024-01-15'  -- BAD (function on column!)
WHERE OrderDate = '2024-01-15'                 -- GOOD (direct compare)
```

## Real production impact

```
SLCProject — billion-row table
WHERE EmployeeCode = 12345    → full scan  → 50 seconds
WHERE EmployeeCode = '12345'  → index seek → 0.01 seconds

Ek chhoti si quote missing = 5000x slower!
```

## Kaise detect karein

```
Execution plan mein dekho: CONVERT_IMPLICIT warning (yellow triangle)
Message: "Type conversion in expression may affect CardinalityEstimate"
```

```sql
-- DMVs se dhundho
SELECT TOP 20 qs.total_logical_reads, qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) qp
WHERE CAST(qp.query_plan AS NVARCHAR(MAX)) LIKE '%CONVERT_IMPLICIT%'
ORDER BY qs.total_logical_reads DESC
```

## Fix checklist

```
1. VARCHAR column → 'value' (quotes!)
2. NVARCHAR column → N'value' (N prefix!)
3. WHERE mein function mat lagao indexed column pe
4. Execution plan check karo CONVERT_IMPLICIT ke liye
```

## Quick reference

```
VARCHAR col  = 'value'   → correct (quotes!)
NVARCHAR col = N'value'  → correct (N prefix!)
WHERE function(col)      → kabhi nahi indexed column pe
CONVERT_IMPLICIT         → execution plan mein dhundho
```

## Interview one-liner

```
Implicit conversion = column values silently convert hote hain → index useless → full scan.
Fix: literal ka data type column ke data type se match karo.
Detect: CONVERT_IMPLICIT warning in execution plan.
```
