# Transactions + TRY/CATCH — Hindi in English

## Bank transfer analogy

Manoj ke account se Ravi ke account mein 10,000 transfer karna hai.

```
Step 1: Manoj ka account -10,000
Step 2: Ravi ka account  +10,000
```

Bich mein server crash. Step 1 hua, Step 2 nahi hua.
Manoj ke 10,000 gaye, Ravi ko nahi mile. **Data corrupt.**

Transaction ke saath:
```
BEGIN TRANSACTION
  Step 1: Manoj -10,000
  Step 2: Ravi  +10,000
COMMIT   ← dono succeed → save
-- Ya agar error aaya →
ROLLBACK ← dono undo → original wapas
```

## Basic syntax

```sql
BEGIN TRANSACTION
    UPDATE Accounts SET Balance = Balance - 10000 WHERE AccountID = 1
    UPDATE Accounts SET Balance = Balance + 10000 WHERE AccountID = 2
COMMIT TRANSACTION
-- Ya:
ROLLBACK TRANSACTION
```

## TRY/CATCH — error pakdo, rollback karo

```sql
BEGIN TRANSACTION

BEGIN TRY
    UPDATE Accounts SET Balance = Balance - 10000 WHERE AccountID = 1
    UPDATE Accounts SET Balance = Balance + 10000 WHERE AccountID = 2
    COMMIT TRANSACTION         -- sab theek → save karo
END TRY

BEGIN CATCH
    IF XACT_STATE() <> 0
        ROLLBACK TRANSACTION   -- kuch gadbad → sab undo

    SELECT ERROR_NUMBER() AS ErrNum, ERROR_MESSAGE() AS ErrMsg
END CATCH
```

## XACT_STATE() — transaction ki health check

```
 1  = transaction open, commit kar sakte hain
-1  = transaction open, commit nahi kar sakte (ROLLBACK must)
 0  = koi transaction nahi
```

## THROW vs RAISERROR

```sql
RAISERROR('Kuch gadbad', 16, 1)    -- purana tarika
THROW 50001, 'Kuch gadbad', 1      -- naya tarika (prefer this)
THROW                               -- CATCH mein: original error re-throw karo
```

## Common mistakes

```sql
-- MISTAKE 1: COMMIT/ROLLBACK bhool gaye → lock hold hoga forever → blocking!
BEGIN TRANSACTION
    UPDATE ...
-- Commit nahi kiya → doosre queries wait karte rahenge

-- MISTAKE 2: Nested transactions — inner COMMIT actually commit nahi karta
BEGIN TRANSACTION   -- @@TRANCOUNT = 1
    BEGIN TRANSACTION   -- @@TRANCOUNT = 2
    COMMIT              -- @@TRANCOUNT = 1 (actually nahi hua commit!)
COMMIT                  -- @@TRANCOUNT = 0 (ab hua)
```

## Quick reference

```
BEGIN TRANSACTION  → shuru karo
COMMIT             → sab save karo
ROLLBACK           → sab undo karo
SAVE TRANSACTION   → checkpoint (partial rollback ke liye)
@@TRANCOUNT        → kitne nested transactions hain
XACT_STATE()       → CATCH mein transaction ki condition check karo
THROW              → error dobara raise karo
```

## Interview one-liner

```
Transaction = all or nothing. Error aaya → ROLLBACK in CATCH.
XACT_STATE() check karo ROLLBACK se pehle.
COMMIT/ROLLBACK bhoolna = lock held forever = blocking.
```
