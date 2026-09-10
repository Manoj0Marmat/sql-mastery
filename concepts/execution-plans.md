# Reading Execution Plans

## Key Metrics

| Metric | What it tells you | How to get it |
|---|---|---|
| StatementSubTreeCost | Estimated total cost — good for quick comparison | Execution plan XML |
| Logical reads | Pages read from buffer/disk — most reliable cost signal | SET STATISTICS IO ON |
| Elapsed time | Wall clock — what the user actually waits | SET STATISTICS TIME ON |
| CPU time | Compute cost | SET STATISTICS TIME ON |
| Scan count | How many times table/index was read | SET STATISTICS IO ON |

## Priority Order
```
1. Elapsed time     ← ground truth (what user feels)
2. Logical reads    ← most reliable, consistent across runs
3. SubTreeCost      ← quick comparison, estimated not actual
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

| Operator | Meaning | Watch out for |
|---|---|---|
| Clustered Index Seek | Efficient — finds rows directly via index | Good, keep it |
| Clustered Index Scan | Reads entire table | Might need an index |
| Nested Loops | Row-by-row join | Fine on small tables |
| Hash Match | Join on large unsorted data | Memory heavy |
| Sort | Orders data mid-plan | Expensive on large data |
| Table Spool | Caches intermediate result in tempdb | Recursive CTEs do this |
| Filter | Post-scan filter | Ideally push this into the seek |

## QueryPlanHash
```
Same hash = same execution plan, different SQL text
CTE vs correlated subquery → often same QueryPlanHash
  → optimizer rewrites both to identical plan
  → choosing between them is about readability, not performance

Window function → different hash → genuinely different plan shape
```

## Real Workflow
```
1. Run query → capture SubTreeCost + logical reads
2. Make change (add index / rewrite)
3. Run again → compare numbers
4. Cost dropped 80%? → confirmed improvement
```

## P15 Real Example
```
Window function:  SubTreeCost = 0.0147  (1 sort, 1 scan)
CTE join:        SubTreeCost = 0.0261  (2 sorts, 2 scans)
Correlated sub:  SubTreeCost = 0.0261  (same plan as CTE — identical QueryPlanHash)

Winner: Window function — 44% cheaper
Lesson: when doing "top N per group", window function wins
```

## Decision: When Plans Are Same vs Different
```
Same QueryPlanHash  → rewrite for readability only, not speed
Different hash      → benchmark both, pick lower logical reads
```
