# CROSS APPLY / OUTER APPLY — Hindi in English

## Chai wala analogy

Class mein 30 students hain. Har student ka **TOP 1 favourite chai** puchna hai.

**JOIN se problem:** JOIN poori list banata hai, TOP 1 per student nahi kar sakta.

**APPLY ka kaam:** Har student ke paas jao, TOP 1 lo, wapas aao.
```
Student 1 → chai shop → TOP 1 lo → wapas
Student 2 → chai shop → TOP 1 lo → wapas
Har row ke liye alag subquery chalti hai — outer row ka data use kar sakti hai
```

## CROSS vs OUTER — ek line mein

```
CROSS APPLY = Sirf wo students jo chai peete hain (no match = bahar)   → INNER JOIN jaisa
OUTER APPLY = Saare students — jo nahi peete, unke liye NULL aayega    → LEFT JOIN jaisa
```

## Syntax

```sql
-- CROSS APPLY — no match = row excluded
SELECT E.EmpName, T.TopSkill
FROM Employees E
CROSS APPLY (
    SELECT TOP 1 SkillName AS TopSkill
    FROM Skills S
    WHERE S.EmpID = E.EmpID        -- outer row ka data use kar raha hai!
    ORDER BY Rating DESC
) T

-- OUTER APPLY — no match = NULL aayega
SELECT E.EmpName, T.TopSkill
FROM Employees E
OUTER APPLY (
    SELECT TOP 1 SkillName AS TopSkill
    FROM Skills S
    WHERE S.EmpID = E.EmpID
    ORDER BY Rating DESC
) T
```

## JOIN se kya problem thi?

```sql
-- WRONG — JOIN mein outer row reference nahi hota
JOIN (SELECT TOP 1 SkillName FROM Skills WHERE EmpID = E.EmpID) S  -- ERROR!

-- CORRECT — APPLY handles per-row correlated subquery
CROSS APPLY (SELECT TOP 1 ... WHERE EmpID = E.EmpID ...) T  -- WORKS
```

## Real use — DBA scripts mein daily

```sql
-- Expensive queries dhundho — har DBA script mein milega
SELECT TOP 10 qs.total_elapsed_time, qt.text AS QueryText
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
-- sys.dm_exec_sql_text = table-valued function, har row ke liye call hoti hai
```

## Quick reference

| | CROSS APPLY | OUTER APPLY |
|---|---|---|
| No match | Row bahar | Row andar (NULL) |
| Equivalent | INNER JOIN | LEFT JOIN |
| Use when | Match guaranteed | Match optional |

```
Kab use karo: TOP N per row, TVF call per row, correlated subquery
```
