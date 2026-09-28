# Transactions + TRY/CATCH

## What is a Transaction?
Group of SQL statements that execute as ONE unit — all succeed or all rollback.

```
ACID:
  Atomicity   = All or nothing
  Consistency = Data valid before and after
  Isolation   = Transactions don't see each other's partial work
  Durability  = Committed data survives crashes
```

## Basic syntax
```sql
BEGIN TRANSACTION
    UPDATE Accounts SET Balance = Balance - 10000 WHERE AccountID = 1
    UPDATE Accounts SET Balance = Balance + 10000 WHERE AccountID = 2
COMMIT TRANSACTION
-- Or on error:
ROLLBACK TRANSACTION
```

## TRY/CATCH
```sql
BEGIN TRANSACTION
BEGIN TRY
    UPDATE Accounts SET Balance = Balance - 10000 WHERE AccountID = 1
    UPDATE Accounts SET Balance = Balance + 10000 WHERE AccountID = 2
    COMMIT TRANSACTION
END TRY
BEGIN CATCH
    IF XACT_STATE() <> 0
        ROLLBACK TRANSACTION
    SELECT ERROR_NUMBER() AS ErrNum, ERROR_MESSAGE() AS ErrMsg, ERROR_LINE() AS ErrLine
END CATCH
```

## XACT_STATE() — check transaction status in CATCH
```
 1  = transaction open, committable
-1  = transaction open, uncommittable (must rollback)
 0  = no active transaction
```

## THROW vs RAISERROR
```sql
RAISERROR('Error message', 16, 1)    -- old way

THROW 50001, 'Error message', 1      -- new way (SQL 2012+), prefer this

THROW    -- no params in CATCH = re-throws original error
```

## Savepoints
```sql
BEGIN TRANSACTION
    INSERT INTO Log VALUES ('Step 1')
    SAVE TRANSACTION SavePoint1

    UPDATE BigTable SET ...
    IF @@ERROR <> 0
        ROLLBACK TRANSACTION SavePoint1   -- rollback to savepoint only

COMMIT TRANSACTION
```

## Common mistakes
```sql
-- Forgotten COMMIT/ROLLBACK → lock held forever → blocking
BEGIN TRANSACTION
    UPDATE ...
-- no COMMIT = lock held until session closes!

-- Nested transactions — inner COMMIT doesn't commit
BEGIN TRANSACTION   -- @@TRANCOUNT = 1
    BEGIN TRANSACTION   -- @@TRANCOUNT = 2
    COMMIT              -- @@TRANCOUNT = 1 (NOT committed yet)
COMMIT                  -- @@TRANCOUNT = 0 (now committed)
```

## Quick reference
```
BEGIN TRANSACTION    → start
COMMIT               → save all
ROLLBACK             → undo all
SAVE TRANSACTION     → savepoint
@@TRANCOUNT          → nested transaction count
XACT_STATE()         → is transaction committable?
ERROR_MESSAGE()      → in CATCH block
THROW                → re-raise error
```
