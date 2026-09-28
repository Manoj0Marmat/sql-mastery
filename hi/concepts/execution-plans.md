# Execution Plans — Hindi in English

## Kya hai execution plan?

SQL Server batata hai: "Maine ye query is tarike se execute ki." Visual map of what happened inside.

## Key metrics — kya dekhna hai

| Metric | Kya batata hai | Kaise milega |
|---|---|---|
| SubTreeCost | Estimated total cost — quick comparison | Plan XML |
| Logical reads | Kitne pages padhe — sabse reliable | SET STATISTICS IO ON |
| Elapsed time | User ne kitna wait kiya — ground truth | SET STATISTICS TIME ON |
| CPU time | Kitna compute laga | SET STATISTICS TIME ON |

## Priority order

```
1. Elapsed time   → user ko kya feel hua (sabse important)
2. Logical reads  → consistent, reliable number
3. SubTreeCost    → quick comparison, estimated nahi actual
```

## Kaise capture karein

```sql
SET STATISTICS TIME ON
SET STATISTICS IO ON

-- apni query yahan

SET STATISTICS TIME OFF
SET STATISTICS IO OFF
```

Messages tab mein dikhega:
```
SQL Server Execution Times:
   CPU time = 0 ms, elapsed time = 1 ms.
Table 'Employees'. Scan count 1, logical reads 2
```

## Key operators — good vs bad

| Operator | Matlab | Dhyan do |
|---|---|---|
| Clustered Index **Seek** | Index use kiya → fast | Good — rakho isko |
| Clustered Index **Scan** | Poori table padhi | Index chahiye shayad |
| Nested Loops | Row-by-row join | Chhoti table pe theek |
| Hash Match | Large unsorted data join | Memory heavy |
| Sort | Mid-plan sorting | Large data pe expensive |

## QueryPlanHash — plan comparison

```
Same hash = same execution plan (chahe SQL alag ho)
CTE vs subquery → often same hash → optimizer dono ko same plan deta hai
Window function → different hash → genuinely alag plan

Same hash → readability ke liye choose karo, performance ke liye nahi
Different hash → dono benchmark karo, lower logical reads wala lo
```

## Real workflow

```
1. Query chalao → SubTreeCost + logical reads capture karo
2. Change karo (index add karo / query rewrite karo)
3. Wapas chalao → compare karo
4. Cost significantly gira? → improvement confirmed
```

## P15 real numbers

```
Window function:  SubTreeCost = 0.0147  (1 sort, 1 scan)
CTE join:         SubTreeCost = 0.0261  (2 sorts, 2 scans)
→ Window function 44% sasta "top N per group" ke liye
```

## Quick reference

```
SET STATISTICS IO ON   → logical reads
SET STATISTICS TIME ON → elapsed + CPU
SubTreeCost            → estimated cost (plan properties mein)
QueryPlanHash          → same value = same plan
Seek > Scan            → seek = index use, scan = sab kuch padha
```
