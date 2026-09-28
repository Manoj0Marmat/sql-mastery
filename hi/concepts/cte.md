# CTEs — Hindi in English

## CTE kya hai?

WITH keyword se banaya hua temporary named result — query ke andar reuse kar sako.

```sql
WITH CTE_Name AS (
    SELECT ...
)
SELECT * FROM CTE_Name
```

## Ek example se samjho

```sql
-- Bina CTE ke — subquery repeat hoti hai
SELECT EmpName FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees)

-- CTE se — ek baar define, baar baar use
WITH AvgSal AS (
    SELECT AVG(CAST(Salary AS DECIMAL(10,2))) AS Avg FROM Employees
)
SELECT EmpName FROM Employees, AvgSal
WHERE Salary > AvgSal.Avg
```

## Chained CTEs — step by step

```sql
WITH
CTE1 AS (SELECT ...),           -- pehla step
CTE2 AS (SELECT * FROM CTE1),   -- CTE1 use kar sakta hai
CTE3 AS (SELECT * FROM CTE2 JOIN CTE1 ON ...)  -- dono use kar sakta hai
SELECT * FROM CTE3
```

## Recursive CTE — Org hierarchy

**Analogy:** Company ka family tree banana hai. CEO → Managers → Employees.

```sql
WITH OrgTree AS (
    -- Anchor: root se shuru karo (CEO — koi manager nahi)
    SELECT EmpID, EmpName, ManagerID, 1 AS Level,
           CAST(NULL AS VARCHAR(50)) AS ManagerName   -- CAST zaruri! NULL ka type nahi hota
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    -- Recursive: har level niche utro
    SELECT e.EmpID, e.EmpName, e.ManagerID,
           r.Level + 1,
           r.EmpName AS ManagerName
    FROM Employees e
    INNER JOIN OrgTree r ON e.ManagerID = r.EmpID   -- apne aap se join!
)
SELECT * FROM OrgTree
```

## Recursion kaise kaam karta hai

```
Iteration 0: CEO milta hai (ManagerID IS NULL)
Iteration 1: CEO ke neeche wale milte hain
Iteration 2: Unke neeche wale milte hain
...jab tak koi match nahi milta...
Final = sab iterations ka UNION ALL
```

## Important rules

```sql
-- Default max recursion = 100 — agar hierarchy deeper ho
OPTION (MAXRECURSION 500)

-- Infinite loop se bacho
WHERE r.Level < 50

-- Recursive JOIN column pe index lagao — recursive member N baar chalta hai
CREATE INDEX IX_Employees_ManagerID ON Employees(ManagerID)

-- NULL type fix — warna Msg 240
CAST(NULL AS VARCHAR(50)) AS ManagerName  -- type specify karo
```

## CTE vs Subquery — performance

```
CTE aur subquery often same plan banate hain (same QueryPlanHash)
Optimizer dono ko same tarike se rewrite karta hai
CTE choose karo readability ke liye, performance ke liye nahi
```

## Kab kya use karo

| Situation | Best |
|---|---|
| Org hierarchy, parent-child | Recursive CTE |
| Same subquery 2+ baar | Regular CTE |
| Multi-step calculation | Chained CTEs |
| Simple, ek baar use | Inline subquery |

## Quick reference

```
WITH name AS (...)       → CTE define karo
Chained: WITH a, b, c   → comma se separate, alag-alag define
Recursive: anchor UNION ALL recursive member
MAXRECURSION hint        → depth control
CAST(NULL AS type)       → anchor mein type mismatch fix
```
