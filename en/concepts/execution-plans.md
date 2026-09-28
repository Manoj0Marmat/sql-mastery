# Reading Execution Plans

## Key Metrics
| Metric | What it tells you | How to get it |
|---|---|---|
| StatementSubTreeCost | Estimated total cost — quick comparison | Execution plan XML |
| Logical reads | Pages read from buffer/disk — most reliable | SET STATISTICS IO ON |
| Elapsed time | Wall clock — what user actually waits | SET STATISTICS TIME ON |
| CPU time | Compute cost | SET STATISTICS TIME ON |
| Scan count | How many times table/index was accessed | SET STATISTICS IO ON |

## Priority Order
```
1. Elapsed time   → ground truth (what user feels)
2. Logical reads  → most reliable, consistent across runs
3. SubTreeCost    → quick comparison, estimated not actual
```

## How to Capture
```sql
SET STATISTICS TIME ON
SET STATISTICS IO ON

-- your query here

SET STATISTICS TIME OFF
SET STATISTICS IO OFF
```

Output in Messages tab:
```
SQL Server Execution Times:
   CPU time = 0 ms, elapsed time = 1 ms.
Table 'Employees'. Scan count 1, logical reads 2
```

## Key Operators
| Operator | Meaning | Watch for |
|---|---|---|
| Clustered Index Seek | Finds rows directly via index | Good — keep it |
| Clustered Index Scan | Reads entire table | Might need an index |
| Nested Loops | Row-by-row join | Fine on small tables |
| Hash Match | Join on large unsorted data | Memory heavy |
| Sort | Orders data mid-plan | Expensive on large data |
| Table Spool | Caches in tempdb | Recursive CTEs do this |
| Filter | Post-scan filter | Push into seek if possible |

## QueryPlanHash
```
Same hash    = same execution plan, different SQL text
Different hash = genuinely different plan shape

CTE vs subquery      → often same hash → optimizer rewrites identically
Window function      → different hash → different plan shape
```

## Real Workflow
```
1. Run query → capture SubTreeCost + logical reads
2. Make change (add index / rewrite query)
3. Run again → compare numbers
4. Cost dropped significantly? → confirmed improvement
```

## P15 Real Numbers
```
Window function:  SubTreeCost = 0.0147  (1 sort, 1 scan)
CTE join:         SubTreeCost = 0.0261  (2 sorts, 2 scans)
→ Window function 44% cheaper for "top N per group"
```

## Quick reference
```
SET STATISTICS IO ON   → logical reads
SET STATISTICS TIME ON → elapsed + CPU
SubTreeCost            → estimated cost (execution plan properties)
QueryPlanHash          → same value = same plan
Seek > Scan            → seek uses index, scan reads everything
```
