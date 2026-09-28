# Stored Procedures — Hindi in English

## Dabba system analogy

**Ad-hoc SQL** = Har din khud khana banao. Ingredients lao, recipe yaad karo, banao.

**Stored Procedure** = Dabba system. Recipe ek baar likhi, dabba banaya.
Ab sirf "dabba bhejo" bolo — poori SQL ka kaam ek line se.

```sql
EXEC GetEmployeesByDept 'IT'
-- Bas itna. Andar kya chal raha hai, DB jaanta hai.
```

## Basic syntax

```sql
CREATE PROCEDURE GetEmployeesByDept
    @Department VARCHAR(50)
AS
BEGIN
    SELECT EmpID, EmpName, Salary
    FROM Employees
    WHERE Department = @Department
    ORDER BY Salary DESC
END

-- Chalao
EXEC GetEmployeesByDept 'IT'
EXEC GetEmployeesByDept @Department = 'HR'   -- named param — clear rehta hai

-- Modify karo
ALTER PROCEDURE GetEmployeesByDept ...

-- Hatao
DROP PROCEDURE IF EXISTS GetEmployeesByDept
```

## Parameters — default values + OUTPUT

```sql
CREATE PROCEDURE GetSummary
    @Department VARCHAR(50),
    @MinSalary  INT = 0,          -- optional, default = 0
    @EmpCount   INT OUTPUT        -- wapas value bhejo
AS
BEGIN
    SELECT * FROM Employees
    WHERE Department = @Department AND Salary >= @MinSalary

    SET @EmpCount = @@ROWCOUNT    -- kitni rows aayi
END

-- Use karo
DECLARE @count INT
EXEC GetSummary @Department = 'IT', @MinSalary = 50000, @EmpCount = @count OUTPUT
SELECT @count   -- result aaya
```

## RETURN — sirf INT status code

```sql
-- RETURN sirf integer return karta hai
RETURN 1    -- found
RETURN 0    -- not found

-- Data return karna ho → OUTPUT ya result set use karo
```

## Scalar function trap — row-by-row = SLOW

```sql
-- SCALAR FUNCTION — har row ke liye alag execute hoti hai
CREATE FUNCTION dbo.GetDeptName(@DeptID INT) RETURNS VARCHAR(50)
AS BEGIN RETURN (SELECT DeptName FROM Departments WHERE DeptID = @DeptID) END

SELECT EmpName, dbo.GetDeptName(DeptID) FROM Employees
-- 10,000 rows hain → 10,000 baar function call! SLOW!
```

```sql
-- INLINE TVF — set-based, ek baar — FAST
CREATE FUNCTION dbo.GetTopEarners(@Dept VARCHAR(50)) RETURNS TABLE
AS RETURN (SELECT TOP 5 EmpName, Salary FROM Employees WHERE Department = @Dept ORDER BY Salary DESC)

SELECT D.DeptName, T.EmpName
FROM Departments D
CROSS APPLY dbo.GetTopEarners(D.DeptName) T   -- set-based, ek hi baar
```

**Rule: Scalar function pe large table mein kabhi mat karo. Inline TVF use karo.**

## Quick reference

```
CREATE PROCEDURE name @param TYPE AS BEGIN ... END
EXEC name @param = value
OUTPUT param      → data wapas bhejo
RETURN            → sirf INT status code
WITH RECOMPILE    → fresh plan (parameter sniffing fix)
Scalar function   → row-by-row → avoid on large tables
Inline TVF        → set-based → CROSS APPLY se use karo
```
