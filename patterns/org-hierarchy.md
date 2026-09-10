# Org Hierarchy Pattern (Recursive CTE)

## The Pattern
```sql
WITH Hierarchy AS (
    -- Anchor: root nodes (no parent)
    SELECT id, name, parent_id, 1 AS Level,
           CAST(NULL AS VARCHAR(50)) AS ParentName
    FROM YourTable
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive: children of current level
    SELECT t.id, t.name, t.parent_id,
           h.Level + 1,
           h.name AS ParentName
    FROM YourTable t
    INNER JOIN Hierarchy h ON t.parent_id = h.id
)
SELECT * FROM Hierarchy
ORDER BY Level, name
OPTION (MAXRECURSION 100)
```

## How It Runs
```
Pass 1 (Anchor):  → CEO
Pass 2:           → CTO, CFO           (parent = CEO)
Pass 3:           → Dev Lead, Accountant  (parent = CTO or CFO)
Pass 4:           → Dev A, Dev B       (parent = Dev Lead)
Pass 5:           → no matches → STOP
```

## Common Errors

### NULL type mismatch (Msg 240)
```sql
-- WRONG
NULL AS ParentName  -- anchor has NULL, recursive has VARCHAR(50) → type conflict

-- CORRECT
CAST(NULL AS VARCHAR(50)) AS ParentName
```

### Infinite loop (cycles in data)
```sql
-- Guard: stop if depth exceeds expected max
WHERE h.Level < 50

-- Or raise MAXRECURSION
OPTION (MAXRECURSION 500)
OPTION (MAXRECURSION 0)  -- unlimited, only if data guaranteed cycle-free
```

## Performance Rules
1. Index the join column: `CREATE INDEX IX ON Table(parent_id)`
2. Select minimum columns — recursive spool lives in tempdb
3. Set MAXRECURSION if hierarchy deeper than 100

## When to Use Alternatives
| Depth | Rows | Alternative |
|---|---|---|
| < 10 levels | Any | Recursive CTE (simplest) |
| > 20 levels | Millions | HierarchyID column type |
| Frequent subtree queries | Large | Nested Sets model |
