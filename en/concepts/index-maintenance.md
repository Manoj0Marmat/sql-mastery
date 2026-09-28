# Index Maintenance

## Why indexes need maintenance?
INSERT/UPDATE/DELETE fragment indexes over time → SQL Server reads more pages → slower queries.

## Check fragmentation first
```sql
SELECT
    i.name                           AS IndexName,
    ips.avg_fragmentation_in_percent AS Fragmentation,
    ips.page_count                   AS PageCount
FROM sys.dm_db_index_physical_stats(
    DB_ID(), OBJECT_ID('Employees'), NULL, NULL, 'LIMITED'
) ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC
```

## Decision rule
```
< 10%   → nothing
10-30%  → REORGANIZE
> 30%   → REBUILD
< 1000 pages → skip (small index, fragmentation doesn't matter)
```

## REORGANIZE — online, gentle
```sql
ALTER INDEX IX_Employees_Dept ON Employees REORGANIZE

-- Always online (no blocking)
-- Reorders leaf-level pages in place
-- Does NOT update statistics → run UPDATE STATISTICS after
-- Use: 10-30% fragmentation
```

## REBUILD — thorough
```sql
ALTER INDEX IX_Employees_Dept ON Employees REBUILD                  -- offline (blocks!)
ALTER INDEX IX_Employees_Dept ON Employees REBUILD WITH (ONLINE=ON) -- online (Enterprise)
ALTER INDEX ALL ON Employees REBUILD WITH (ONLINE=ON)               -- all indexes on table

-- Updates statistics automatically
-- Use: > 30% fragmentation
-- ONLINE=ON requires Enterprise Edition
```

## UPDATE STATISTICS
```sql
UPDATE STATISTICS Employees IX_Employees_Dept   -- specific index
UPDATE STATISTICS Employees                     -- all stats on table
EXEC sp_updatestats                             -- all stats in database
```

## REBUILD vs UPDATE STATISTICS
```
REBUILD      → updates statistics automatically (full scan)
REORGANIZE   → does NOT update statistics → run UPDATE STATISTICS separately
UPDATE STATS → stats only, no index rebuild (faster, less accurate)
```

## DBCC commands
```sql
DBCC SHOW_STATISTICS('Employees', 'IX_Employees_Dept')  -- inspect histogram, density
DBCC CHECKDB('YourDatabase')                             -- weekly integrity check
DBCC CHECKTABLE('Employees')                             -- single table check
```

## Maintenance schedule
```
Weekly:  REBUILD > 30%  →  Saturday night
         REORGANIZE 10-30%  →  Sunday night
         UPDATE STATISTICS after REORGANIZE
Monthly: DBCC CHECKDB
Daily:   UPDATE STATISTICS on heavy-write tables
```

## Quick reference
```
sys.dm_db_index_physical_stats  → check fragmentation
REORGANIZE    → 10-30%, always online, no stats update
REBUILD       → >30%, stats updated, ONLINE=ON for no blocking
UPDATE STATS  → after REORGANIZE
DBCC CHECKDB  → weekly integrity
```
