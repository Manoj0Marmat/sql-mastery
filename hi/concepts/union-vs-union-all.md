# UNION vs UNION ALL — Hindi in English

## Class attendance analogy

Do sections: Section A aur Section B. Teacher dono ki attendance ek list mein chahiye.

```
Section A: Alice, Bob, Carol, Dave
Section B: Carol, Dave, Eve, Frank   ← Carol aur Dave dono mein hain
```

**UNION ALL — seedha chipkao:**
```
Alice, Bob, Carol, Dave, Carol, Dave, Eve, Frank
                   ^^^^^^^^^^^^  duplicates rakhe — fast, koi extra kaam nahi
```

**UNION — chipkao + duplicates hatao:**
```
Alice, Bob, Carol, Dave, Eve, Frank
Sort kiya → duplicates dhundhe → hataye → SLOW
```

## Simple rule

```
UNION ALL = Seedha dono lists chipkao (duplicates keep) → FAST
UNION     = Chipkao + duplicates nikalo (sort + dedup)  → SLOW

Hamesha UNION ALL prefer karo jab duplicates impossible ya okay hain.
```

## Syntax

```sql
SELECT EmpName FROM Employees_India
UNION ALL                               -- prefer this!
SELECT EmpName FROM Employees_US

SELECT CustomerID FROM CRM_Customers
UNION                                   -- sirf tab jab duplicates MUST remove karne hain
SELECT CustomerID FROM ERP_Customers
```

## Rules — dono ke liye

```
Same number of columns
Compatible data types per column
Column names → pehle SELECT se aate hain
```

## EXCEPT aur INTERSECT — related

```sql
-- EXCEPT — pehli list mein hai, doosri mein nahi
SELECT CustomerID FROM AllCustomers
EXCEPT
SELECT CustomerID FROM ActiveCustomers  -- inactive customers

-- INTERSECT — dono lists mein hai
SELECT CustomerID FROM CRM
INTERSECT
SELECT CustomerID FROM ERP              -- dono mein common customers
```

## Memory trick

```
UNION ALL  = A + B (saab rakho, fast)
UNION      = A + B - duplicates (slow, sort laga)
EXCEPT     = A - B (B walon ko nikalo)
INTERSECT  = A ∩ B (sirf common)
```

## Quick reference

| | UNION | UNION ALL |
|---|---|---|
| Duplicates | Hata deta hai | Rakhta hai |
| Speed | Slow (sort+dedup) | Fast |
| Use when | Must remove duplicates | Always prefer this |
