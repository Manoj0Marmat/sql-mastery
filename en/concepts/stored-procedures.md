# Stored Procedures

## What is a Stored Procedure?
Pre-compiled T-SQL stored in database. Execute by name. Plan cached.

```
Benefits:
  ✓ Plan cached — compiled once, reused
  ✓ Reduced network traffic — name not full SQL
  ✓ Security — EXEC permission without table access
  ✓ Encapsulation — logic in DB not application
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

EXEC GetEmployeesByDept 'IT'
EXEC GetEmployeesByDept @Department = 'IT'   -- named params (clearer)

ALTER PROCEDURE GetEmployeesByDept ...        -- modify
DROP PROCEDURE IF EXISTS GetEmployeesByDept   -- drop
```

## Parameters — default values + OUTPUT
```sql
CREATE PROCEDURE GetSummary
    @Department VARCHAR(50),
    @MinSalary  INT = 0,          -- optional, default = 0
    @EmpCount   INT OUTPUT        -- output parameter
AS
BEGIN
    SELECT * FROM Employees
    WHERE Department = @Department AND Salary >= @MinSalary

    SET @EmpCount = @@ROWCOUNT
END

-- Execute with output
DECLARE @count INT
EXEC GetSummary @Department = 'IT', @MinSalary = 50000, @EmpCount = @count OUTPUT
SELECT @count
```

## RETURN — integer status only
```sql
CREATE PROCEDURE CheckDept @Dept VARCHAR(50)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Departments WHERE DeptName = @Dept)
        RETURN 1    -- found
    RETURN 0        -- not found
END

DECLARE @result INT
EXEC @result = CheckDept 'IT'   -- capture return value
```

## WITH RECOMPILE
```sql
CREATE PROCEDURE GetOrders @CustomerID INT
WITH RECOMPILE    -- fresh plan every execution
AS ...

-- Or per-execution:
EXEC GetOrders 123 WITH RECOMPILE
-- Use when: parameter sniffing causes bad plans
```

## Scalar function vs Inline TVF — performance trap
```sql
-- SCALAR FUNCTION — row-by-row = SLOW
CREATE FUNCTION dbo.GetDeptName(@DeptID INT) RETURNS VARCHAR(50)
AS BEGIN RETURN (SELECT DeptName FROM Departments WHERE DeptID = @DeptID) END

SELECT EmpName, dbo.GetDeptName(DeptID) FROM Employees   -- N calls for N rows!

-- INLINE TVF — set-based = FAST
CREATE FUNCTION dbo.GetTopEarners(@Dept VARCHAR(50)) RETURNS TABLE
AS RETURN (SELECT TOP 5 EmpName, Salary FROM Employees WHERE Department = @Dept ORDER BY Salary DESC)

SELECT D.DeptName, T.EmpName
FROM Departments D
CROSS APPLY dbo.GetTopEarners(D.DeptName) T    -- set-based, one execution
```

## Quick reference
```
CREATE PROCEDURE name @param TYPE AS BEGIN ... END
EXEC name @param = value
ALTER / DROP PROCEDURE IF EXISTS
OUTPUT param         → return data back to caller
RETURN               → INT status code only
WITH RECOMPILE       → fresh plan (parameter sniffing fix)
Scalar function      → avoid on large tables (row-by-row)
Inline TVF           → prefer (set-based, CROSS APPLY)
```
