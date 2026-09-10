# Dedup Pattern

## The Pattern
```sql
WITH Ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY col1, col2, col3   -- columns that define a "duplicate"
            ORDER BY id ASC                 -- which duplicate to KEEP (lowest id = original)
        ) AS rn
    FROM TableName
)
-- To preview:
SELECT * FROM Ranked WHERE rn > 1   -- these will be deleted

-- To delete:
DELETE FROM Ranked WHERE rn > 1
```

## Reading It
```
PARTITION BY EmpName, Department, Salary
→ "treat rows with same name+dept+salary as a group"

ORDER BY EmpID ASC
→ "within that group, row 1 = lowest EmpID = the one we keep"

WHERE rn > 1
→ "delete everything that's not the first row in its group"
```

## When to Use
- Duplicate rows in a table
- "Keep one row per customer, latest record"
- "Deduplicate imports"
- Preview before delete — always check `WHERE rn > 1` first

## Common Mistakes
```sql
-- WRONG: GROUP BY doesn't guarantee which row is kept
SELECT EmpName, Department, Salary FROM Employees
GROUP BY EmpName, Department, Salary

-- CORRECT: ROW_NUMBER guarantees specific row is kept
WITH Ranked AS (...)
SELECT * FROM Ranked WHERE rn = 1

-- WRONG: deleting from base table directly (risky — no preview)
DELETE FROM Employees WHERE EmpID IN (SELECT EmpID FROM ...)

-- CORRECT: CTE delete — readable, safe, previewable
WITH Ranked AS (...) DELETE FROM Ranked WHERE rn > 1
```
