# Temp Tables vs Table Variables vs CTEs — Hindi in English

## Roti banana analogy

**CTE** = Recipe jo sirf isi kaam ke liye yaad hai. Query khatam → bhool gaye.
```
Koi storage nahi. Query ke andar sirf reference.
```

**@TableVariable** = Chhote dabbe mein rakhi atta — isi kaam ke liye, chhoti quantity.
```
Batch khatam → dabbe gaya. Chota kaam ke liye theek.
```

**#TempTable** = Fridge mein rakha atta — zyada quantity, kal bhi use kar sakte.
```
Session khatam → fridge khali. Bada kaam ke liye.
```

## Side-by-side

| Feature | #TempTable | @TableVariable | CTE |
|---|---|---|---|
| Storage | tempdb (disk) | Memory (mostly) | Koi nahi |
| Scope | Session / procedure | Sirf batch | Sirf query |
| Indexes | Haan — full | Limited (PK only) | Nahi |
| Statistics | Haan | Nahi | Nahi |
| Transaction aware | Haan | Nahi | N/A |

## Syntax

```sql
-- CTE — ek baar use, bhool jao
WITH CTE AS (SELECT ...)
SELECT * FROM CTE

-- @TableVariable — chhota kaam
DECLARE @Emps TABLE (EmpID INT, EmpName VARCHAR(100))
INSERT INTO @Emps SELECT TOP 10 EmpID, EmpName FROM Employees
SELECT * FROM @Emps

-- #TempTable — bada kaam, index bhi lagao
CREATE TABLE #Emps (EmpID INT, EmpName VARCHAR(100))
INSERT INTO #Emps SELECT TOP 10000 EmpID, EmpName FROM Employees
CREATE INDEX IX_Emps ON #Emps(EmpID)   -- indexes support karta hai!
SELECT * FROM #Emps
DROP TABLE IF EXISTS #Emps
```

## Performance trap — @TableVariable

```
@TableVariable mein statistics nahi hoti
Optimizer hamesha assume karta hai: 1 row hai
Tum 100,000 rows insert karo → optimizer phir bhi plan banata hai 1 row ke liye
→ Galat plan → slow

#TempTable mein statistics hoti hain → optimizer sahi count jaanta hai → better plan

Rule: 1000 rows se zyada ho → #TempTable use karo, @TableVariable nahi
```

## Kab kya use karo

```
Sirf isi query ke liye?              → CTE
Chhota data, batch mein kaam?         → @TableVariable
Bada data, multiple queries, index?   → #TempTable
```

## Quick reference

```
CTE           → sirf ek query ke liye, koi storage nahi
@TableVariable → chhota (< 1000 rows), no indexes, batch scope
#TempTable    → bada data, indexes lagao, session scope
Statistics    → sirf #TempTable mein → better optimizer plans
```
