# MERGE

## What is MERGE?

Single statement that performs INSERT + UPDATE + DELETE based on match condition.
Used for: ETL sync, upsert operations, master data management.

```
TARGET = table that receives changes (main table)
SOURCE = new data to compare against (feed/staging) — never modified
```

## Syntax
```sql
MERGE TargetTable AS T
USING SourceTable AS S
ON T.KeyCol = S.KeyCol

WHEN MATCHED THEN
    UPDATE SET T.Col1 = S.Col1, T.Col2 = S.Col2

WHEN NOT MATCHED BY TARGET THEN
    INSERT (Col1, Col2, Col3)
    VALUES (S.Col1, S.Col2, S.Col3)

WHEN NOT MATCHED BY SOURCE THEN
    DELETE;    -- semicolon mandatory at end
```

## Three WHEN conditions
```
MATCHED             → row in both TARGET and SOURCE    → UPDATE
NOT MATCHED BY TARGET → row in SOURCE, not in TARGET   → INSERT
NOT MATCHED BY SOURCE → row in TARGET, not in SOURCE   → DELETE or soft delete
```

## Allowed actions per condition
```
WHEN MATCHED              → UPDATE or DELETE
WHEN NOT MATCHED BY TARGET → INSERT only (row doesn't exist in target!)
WHEN NOT MATCHED BY SOURCE → UPDATE or DELETE
```

## OUTPUT clause — log every action
```sql
MERGE ...
...
OUTPUT $action, inserted.EmpID, inserted.EmpName
INTO #AuditLog;

-- $action         = 'INSERT', 'UPDATE', or 'DELETE' (automatic)
-- inserted.col    = new values (after change)
-- deleted.col     = old values (before change)
```

## Production pattern — soft delete
```sql
-- Never DELETE in production — mark inactive instead
WHEN NOT MATCHED BY SOURCE THEN
    UPDATE SET T.IsActive = 0, T.LastUpdated = GETDATE()
```

## Performance warning
```
MERGE has documented race condition issues under high concurrency.
Known bug: SQL Server 2008–2019 (officially acknowledged by Microsoft).

USE:   batch jobs, nightly ETL, low concurrency, small-medium tables
AVOID: high concurrency, billion-row tables, mission-critical real-time sync

Safer alternative for large tables:
  UPDATE with INNER JOIN  → update existing rows
  INSERT with NOT EXISTS  → insert new rows
  Separate statements = better lock granularity
```

## Quick reference
```
MERGE Target AS T          → who gets changed
USING Source AS S          → read-only reference
ON T.key = S.key           → matching condition
WHEN MATCHED               → UPDATE
WHEN NOT MATCHED BY TARGET → INSERT
WHEN NOT MATCHED BY SOURCE → DELETE / soft-delete
OUTPUT $action             → audit log
;                          → mandatory semicolon at end
```
