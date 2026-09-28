# Aggregates

## Execution Order
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

## Functions
```sql
COUNT(*)           -- counts all rows including NULLs
COUNT(column)      -- skips NULLs
COUNT(1)           -- same as COUNT(*), style preference
SUM(column)
AVG(column)        -- INT column → truncates decimals
MIN(column)
MAX(column)
```

## DECIMAL Precision
```sql
-- INT / INT = INT (truncates .33)
AVG(salary)  -- bad for decimals

-- Fix: cast inside AVG
CAST(AVG(CAST(salary AS DECIMAL(10,2))) AS DECIMAL(10,2))
--        ↑ preserve during division    ↑ control display

-- Money: always use DECIMAL(18,2)
```

## WHERE vs HAVING
```sql
WHERE   -- filters ROWS before grouping
HAVING  -- filters GROUPS after grouping

-- WRONG: aggregate not allowed in WHERE
WHERE COUNT(1) > 3

-- CORRECT
HAVING COUNT(1) > 3
```

## CASE inside Aggregates
```sql
-- Count only rows matching condition
SUM(CASE WHEN salary >= 85000 THEN 1 ELSE 0 END) AS HighEarners
-- Always ELSE 0 not ELSE NULL → avoids NULL in output
```

## CASE Versions
```sql
-- Searched CASE (conditions differ per WHEN)
CASE
    WHEN salary >= 85000 THEN 'High'
    WHEN salary >= 65000 THEN 'Mid'  -- CASE stops at first match, no need to repeat upper bound
    ELSE 'Low'
END

-- Simple CASE (equality only)
CASE Department
    WHEN 'Engineering' THEN 'Tech'
    WHEN 'HR'          THEN 'People'
    ELSE 'Other'
END
```

## Alias Rules
```sql
-- Alias NOT available in HAVING (resolves same level as SELECT)
HAVING NoOfEmp > 3       -- ERROR: Msg 207
HAVING COUNT(1) > 3      -- correct

-- Alias available in ORDER BY (last to resolve)
ORDER BY NoOfEmp DESC    -- works
```

## Performance
```
CAST in SELECT/HAVING → safe, no index impact
CAST on column in WHERE/JOIN → kills index seek → full scan
COUNT(EmpID) not COUNT(1) when NULLs possible from outer JOINs
```

## Quick reference
```
GROUP BY  → group rows
HAVING    → filter groups (after GROUP BY)
WHERE     → filter rows (before GROUP BY)
COUNT(*)  → all rows | COUNT(col) → non-NULL only
AVG(INT)  → truncates → cast to DECIMAL first
```
