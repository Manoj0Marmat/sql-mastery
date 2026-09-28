# Dynamic SQL — sp_executesql

## What is Dynamic SQL?
SQL built as a string at runtime and executed. Used when table names, column names, or conditions are unknown at compile time.

## EXEC vs sp_executesql
```sql
-- EXEC — simple but SQL injection risk
DECLARE @sql NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE Dept = ''' + @dept + ''''
EXEC(@sql)   -- DANGEROUS — never use with user input

-- sp_executesql — parameterized, safe, plan cached
DECLARE @sql NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE Dept = @Dept'
EXEC sp_executesql @sql, N'@Dept VARCHAR(50)', @Dept = @dept
```

**Always use sp_executesql. Never concatenate user input.**

## SQL Injection example
```sql
-- User types: ' OR 1=1 --
-- Concatenation builds: SELECT * FROM Employees WHERE Dept = '' OR 1=1 --'
-- Returns ALL rows → security breach

-- sp_executesql: parameter treated as data, not code → safe
```

## Common patterns
```sql
-- Dynamic table name (must use QUOTENAME)
DECLARE @table NVARCHAR(128) = 'Employees_2024'
EXEC sp_executesql N'SELECT * FROM ' + QUOTENAME(@table)

-- Dynamic ORDER BY
DECLARE @col NVARCHAR(50) = 'Salary', @dir NVARCHAR(4) = 'DESC'
EXEC sp_executesql N'SELECT * FROM Employees ORDER BY ' + QUOTENAME(@col) + ' ' + @dir

-- Optional WHERE conditions
DECLARE @sql NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE 1=1'
IF @dept   IS NOT NULL SET @sql += ' AND Department = @Dept'
IF @minSal IS NOT NULL SET @sql += ' AND Salary >= @MinSal'
EXEC sp_executesql @sql, N'@Dept VARCHAR(50), @MinSal INT', @Dept = @dept, @MinSal = @minSal

-- Output parameter
DECLARE @count INT
EXEC sp_executesql N'SELECT @Count = COUNT(1) FROM Employees',
    N'@Count INT OUTPUT', @Count = @count OUTPUT
```

## QUOTENAME() — required for object names
```sql
QUOTENAME('Employees')          → [Employees]
QUOTENAME('My Table')           → [My Table]
QUOTENAME('; DROP TABLE Emp--') → [; DROP TABLE Emp--]  -- injection prevented
```

## Performance
```
sp_executesql + parameters → plan cached (same as stored procedure)
EXEC() with literals       → new plan per unique SQL text → cache bloat
```

## Quick reference
```
sp_executesql @sql, N'@p TYPE', @p = val   → always use this
EXEC(@sql)                                  → avoid (injection risk)
QUOTENAME(@objectName)                      → table/column names only
1=1 trick                                   → dynamic WHERE conditions
OUTPUT param                                → return values from dynamic SQL
```
