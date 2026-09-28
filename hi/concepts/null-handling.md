# NULL Handling — Hindi in English

## NULL kya hai?

NULL = "Pata nahi". Zero nahi, empty string nahi, false nahi. **Unknown.**

## Absent student analogy

Register mein Ravi ka marks column blank hai. Teacher ne marks diye hi nahi.

```
Blank (NULL) ≠ Zero marks
Blank (NULL) ≠ "Failed"
Blank (NULL) = "Marks entered hi nahi hue — pata nahi"
```

```sql
NULL = NULL   → FALSE  (pata nahi = pata nahi = still pata nahi)
NULL + 5      → NULL   (pata nahi + 5 = still pata nahi)
```

## Teen functions

### ISNULL — do values, SQL Server
```sql
ISNULL(Salary, 0)
-- Salary NULL hai → 0 return karo
-- Salary 50000 hai → 50000 return karo
```

### COALESCE — multiple values, standard
```sql
COALESCE(NickName, FirstName, 'Unknown')
-- Pehla non-NULL value return karo
-- NickName NULL, FirstName 'Ravi' → 'Ravi'
-- Dono NULL → 'Unknown'
```

### NULLIF — equal hain → NULL karo
```sql
NULLIF(Units, 0)
-- Units = 0 → NULL return karo (divide by zero se bacho!)
Revenue / NULLIF(Units, 0)   -- safe division
```

## NULL comparison — common galti

```sql
-- WRONG — hamesha 0 rows aayenge
WHERE Salary = NULL
WHERE Salary <> NULL

-- CORRECT
WHERE Salary IS NULL
WHERE Salary IS NOT NULL
```

**Yaad karo:** NULL ke saath = mat karo. Hamesha IS NULL / IS NOT NULL.

## NULL in aggregates

```sql
COUNT(*)       -- saari rows gino (NULL bhi)
COUNT(Salary)  -- sirf non-NULL gino

SUM, AVG, MIN, MAX  -- sab NULL ignore karte hain automatically
```

## NOT IN trap — sabse dangerous

```sql
-- Agar Departments mein koi bhi NULL hai → ZERO rows!
WHERE DeptID NOT IN (SELECT DeptID FROM Departments)

-- Safe:
WHERE NOT EXISTS (SELECT 1 FROM Departments D WHERE D.DeptID = E.DeptID)
```

## Quick reference

```
ISNULL(col, default)     → NULL replace karo (2 values, SQL Server)
COALESCE(v1, v2, v3)     → pehla non-NULL lo (multiple values)
NULLIF(v1, v2)           → equal hain → NULL (divide by zero guard)
IS NULL / IS NOT NULL    → NULL check karo (= NULL kabhi nahi!)
COUNT(*) vs COUNT(col)   → col NULL wale nahi ginega
NOT IN + NULL = 0 rows   → NOT EXISTS use karo
```
