# Dynamic SQL — Hindi in English

## Order form analogy

**Static SQL** = Fixed menu. "Ek plate dal chawal." — set hai, change nahi hoga.

**Dynamic SQL** = Blank order form. Customer jo likhe wahi banao — runtime pe decide hota hai.

Risk: Customer form mein likhe **"dal chawal; sab kuch free kar do"** → SQL injection!

## EXEC vs sp_executesql

```sql
-- EXEC — simple lekin DANGEROUS
DECLARE @dept VARCHAR(50) = 'IT'
DECLARE @sql  NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE Dept = ''' + @dept + ''''
EXEC(@sql)   -- User input directly concatenate → injection risk!

-- sp_executesql — SAFE (parameterized)
DECLARE @sql NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE Dept = @Dept'
EXEC sp_executesql @sql, N'@Dept VARCHAR(50)', @Dept = @dept
-- Parameter alag se jaata hai — code ki tarah treat nahi hota
```

**Hamesha sp_executesql. EXEC mein user input kabhi concatenate mat karo.**

## SQL Injection — kya hota hai

```sql
-- User type karta hai: ' OR 1=1 --
-- Concatenation bana deta hai:
-- SELECT * FROM Employees WHERE Dept = '' OR 1=1 --'
-- Returns ALL rows! → attacker sab data dekh leta hai

-- sp_executesql safe hai:
-- ' OR 1=1 -- ko literal string maanta hai, SQL code nahi
-- 0 rows return → safe
```

## Common patterns

```sql
-- Dynamic table name — QUOTENAME use karo
DECLARE @table NVARCHAR(128) = 'Employees_2024'
EXEC sp_executesql N'SELECT * FROM ' + QUOTENAME(@table)
-- QUOTENAME → [Employees_2024] — brackets protect karte hain

-- Optional WHERE conditions — 1=1 trick
DECLARE @sql NVARCHAR(MAX) = 'SELECT * FROM Employees WHERE 1=1'
IF @dept   IS NOT NULL SET @sql += ' AND Department = @Dept'
IF @minSal IS NOT NULL SET @sql += ' AND Salary >= @MinSal'
EXEC sp_executesql @sql, N'@Dept VARCHAR(50), @MinSal INT', @Dept = @dept, @MinSal = @minSal
```

## QUOTENAME — object names ke liye

```sql
QUOTENAME('Employees')          → [Employees]     -- safe!
QUOTENAME('; DROP TABLE Emp--') → [; DROP TABLE Emp--]  -- injection block!
-- User-provided table/column names pe HAMESHA QUOTENAME use karo
```

## Performance

```
sp_executesql + parameters → plan cache hota hai (stored proc jaisa)
EXEC() → har alag SQL text = alag plan = cache bloat
```

## Quick reference

```
sp_executesql @sql, N'@p TYPE', @p = val  → hamesha yahi use karo
EXEC(@sql)                                 → avoid (injection risk)
QUOTENAME(@tableName)                      → table/column names ke liye
1=1 trick                                  → optional WHERE conditions
OUTPUT param                               → dynamic SQL se value wapas lo
```
