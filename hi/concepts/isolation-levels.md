# Isolation Levels — Hindi in English

## Bank passbook analogy

Tum bank mein ho, passbook update ho rahi hai (transaction chal raha hai).

```
READ UNCOMMITTED = Adha-update hua balance dekh sakte ho → wrong number!
READ COMMITTED   = Sirf final update ke baad dekh sakte ho (default — safe)
REPEATABLE READ  = Jo balance tune pehle dekha, wahi dikhega baar baar (lock rakha)
SERIALIZABLE     = Poori passbook lock — koi naya entry bhi nahi aa sakta
SNAPSHOT         = Apni copy pe kaam karo — original touch nahi karo
```

## Pancch levels

### READ UNCOMMITTED — sabse loose, sabse dangerous
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
-- Ya: WITH (NOLOCK) hint
SELECT * FROM Orders WITH (NOLOCK)

-- Dirty data read karta hai (uncommitted changes)
-- Risk: transaction rollback ho gaya → tumne galat data dekh liya
-- Use: sirf approximate counts ke liye
```

### READ COMMITTED — SQL Server default
```sql
-- Default hai — set karne ki zaroorat nahi

-- Sirf committed data padhta hai → no dirty reads
-- Same query dobara chalao → alag result aa sakta hai (non-repeatable read)
-- Use: zyaataar OLTP applications
```

### REPEATABLE READ
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ

-- Jo rows tune padhe, unhe lock kar deta hai — koi update nahi kar sakta
-- New rows aa sakti hain (phantom reads)
-- Use: reports jo consistent rehni chahiye transaction ke andar
```

### SERIALIZABLE — sabse strict
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE

-- Range lock → koi bhi change nahi ho sakta
-- Maximum blocking → slowest
-- Use: financial transactions, critical operations
```

### SNAPSHOT — no blocking magic
```sql
ALTER DATABASE YourDB SET READ_COMMITTED_SNAPSHOT ON

-- Readers writers ko block nahi karte, writers readers ko block nahi karte
-- tempdb mein row versions rakhta hai
-- Use: high concurrency — reporting + OLTP dono saath
```

## Comparison

| Level | Dirty Read | Same query = same result? | Blocking |
|---|---|---|---|
| READ UNCOMMITTED | Haan (danger!) | Nahi | None |
| READ COMMITTED | Nahi | Nahi | Low |
| REPEATABLE READ | Nahi | Haan | Medium |
| SERIALIZABLE | Nahi | Haan | High |
| SNAPSHOT | Nahi | Haan | None |

## WITH (NOLOCK) — dangerous shortcut

```sql
SELECT * FROM BigTable WITH (NOLOCK)
-- = READ UNCOMMITTED for this table
-- Dirty data, ek row dobara read, rows skip — sab possible
-- Common misuse: "NOLOCK lagao, faster hoga"
-- Real risk: financial totals galat ho sakte hain
```

Kabhi use karo: approximate counts, non-critical monitoring.
Kabhi mat use karo: financial, order processing, kuch bhi accurate.

## Quick reference

```
READ UNCOMMITTED → fastest, dirtiest (NOLOCK) → accurate nahi
READ COMMITTED   → default, balanced → OLTP ke liye theek
REPEATABLE READ  → consistent reads, zyada locks
SERIALIZABLE     → cleanest, most blocking
SNAPSHOT         → no blocking, tempdb versioning → best for mixed
```
