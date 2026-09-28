# Isolation Levels

## What are Isolation Levels?
Control what one transaction can see of another transaction's uncommitted changes.

```
Higher isolation = cleaner reads + more blocking
Lower isolation  = more concurrency + dirty/phantom reads possible
```

## Five levels

### READ UNCOMMITTED
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
SELECT * FROM Orders WITH (NOLOCK)   -- hint equivalent

-- Reads uncommitted (dirty) data
-- Risk: reads data that gets rolled back → wrong results
-- Use: approximate counts only
```

### READ COMMITTED — SQL Server default
```sql
-- Default — no need to set explicitly

-- Reads only committed data → no dirty reads
-- Non-repeatable reads possible (same query twice = different results)
-- Use: most OLTP applications
```

### REPEATABLE READ
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ

-- Locks rows you've read → no updates by others during transaction
-- Phantom reads still possible (new rows can be inserted)
-- Use: reports that must be consistent within transaction
```

### SERIALIZABLE
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE

-- Locks range → no dirty, non-repeatable, or phantom reads
-- Maximum blocking → slowest
-- Use: financial transactions, audit-critical operations
```

### SNAPSHOT
```sql
ALTER DATABASE YourDB SET READ_COMMITTED_SNAPSHOT ON   -- database level
-- Or per session:
SET TRANSACTION ISOLATION LEVEL SNAPSHOT

-- Readers don't block writers, writers don't block readers
-- Uses row versions in tempdb
-- Use: high concurrency mixed OLTP + reporting
```

## Comparison table
| Level | Dirty Read | Non-Repeatable | Phantom Read | Blocking |
|---|---|---|---|---|
| READ UNCOMMITTED | Yes | Yes | Yes | None |
| READ COMMITTED | No | Yes | Yes | Low |
| REPEATABLE READ | No | No | Yes | Medium |
| SERIALIZABLE | No | No | No | High |
| SNAPSHOT | No | No | No | None (versioning) |

## WITH (NOLOCK) — the dangerous shortcut
```sql
SELECT * FROM BigTable WITH (NOLOCK)
-- = READ UNCOMMITTED for this table
-- Can read dirty data, read same row twice, skip rows entirely
-- Common misuse: "just add NOLOCK to make it faster"
-- Risk: wrong totals in financial reports
```

Use NOLOCK only for: approximate counts, non-critical monitoring.
Never for: financial data, order processing, anything accuracy-critical.

## Quick reference
```
READ UNCOMMITTED → fastest, dirtiest (NOLOCK)
READ COMMITTED   → default, balanced
REPEATABLE READ  → consistent reads, more locks
SERIALIZABLE     → cleanest, most blocking
SNAPSHOT         → no blocking, tempdb versioning, best for mixed workloads
```
