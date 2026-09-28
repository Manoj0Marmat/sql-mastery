# Index Maintenance — Hindi in English

## Library books analogy

Library mein books order mein rakhi hain (index). Roz naye books aate hain, kuch nikalte hain.

**Kuch mahino baad:** Books ki jagah idhar-udhar ho gayi. Ek subject ki books
3 shelves pe spread. Dhundhna slow.

**Solution:**
- Thodi badi mess → books wahan hi rearrange karo (REORGANIZE)
- Bahut zyada mess → saari books nikalo, order mein wapas rakho (REBUILD)

## Pehle fragmentation check karo

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
< 10%   → kuch mat karo
10-30%  → REORGANIZE (gentle rearrange)
> 30%   → REBUILD (poora rebuild)
< 1000 pages → skip (chhota index, fragmentation matter nahi karta)
```

## REORGANIZE — gentle, always online

```sql
ALTER INDEX IX_Employees_Dept ON Employees REORGANIZE

-- Koi blocking nahi (always online)
-- Leaf-level pages rearrange karta hai
-- Statistics UPDATE nahi karta → baad mein UPDATE STATISTICS chalao
-- Use: 10-30% fragmentation
```

## REBUILD — thorough

```sql
-- Offline (blocks — business hours mein avoid karo)
ALTER INDEX IX_Employees_Dept ON Employees REBUILD

-- Online (koi blocking nahi — prefer this)
ALTER INDEX IX_Employees_Dept ON Employees REBUILD WITH (ONLINE = ON)

-- Table ke saare indexes rebuild
ALTER INDEX ALL ON Employees REBUILD WITH (ONLINE = ON)

-- Statistics automatically update hoti hain
-- Use: > 30% fragmentation
-- ONLINE = ON → Enterprise Edition chahiye
```

## UPDATE STATISTICS

```sql
UPDATE STATISTICS Employees IX_Employees_Dept   -- specific index
UPDATE STATISTICS Employees                     -- table ke saare stats
EXEC sp_updatestats                             -- database ke saare stats
```

## REBUILD vs UPDATE STATISTICS

```
REBUILD      → statistics automatic update (full scan)
REORGANIZE   → statistics update NAHI hoti → baad mein UPDATE STATISTICS chalao
UPDATE STATS → sirf statistics, rebuild nahi (faster, thoda kam accurate)
```

## DBCC commands

```sql
DBCC SHOW_STATISTICS('Employees', 'IX_Employees_Dept')  -- histogram, density dekho
DBCC CHECKDB('YourDatabase')   -- weekly integrity check
```

## Maintenance schedule

```
Har hafte:
  Saturday raat → REBUILD > 30%
  Sunday raat   → REORGANIZE 10-30%
  REORGANIZE ke baad → UPDATE STATISTICS

Har mahine:
  DBCC CHECKDB

Roz:
  Heavy-write tables pe UPDATE STATISTICS
```

## Quick reference

```
sys.dm_db_index_physical_stats → fragmentation check karo
REORGANIZE  → 10-30%, gentle, online, stats update nahi
REBUILD     → >30%, thorough, stats update, ONLINE=ON for no blocking
UPDATE STATS → REORGANIZE ke baad run karo
DBCC CHECKDB → weekly
```
