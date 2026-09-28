# Window Functions — Hindi in English

## Syntax

```sql
FUNCTION() OVER (
    PARTITION BY column   -- optional: har group ke liye reset
    ORDER BY column       -- calculation ka order
    ROWS BETWEEN ...      -- optional: frame
)
```

## Key rule

```
ORDER BY inside OVER()  → calculation ka order control karta hai
ORDER BY end mein       → display order control karta hai
Dono INDEPENDENT hain — ek doosre ko affect nahi karte
```

## Ranking functions — Race analogy

5 log ek race mein hain, Bob aur Carol ki timing same (tie):

```
Name   Time
───────────
Alice  10s   ← sabse fast
Bob    12s   ← tie
Carol  12s   ← tie
Dave   15s
Eve    18s   ← sabse slow
```

**ROW_NUMBER — "Queue number do, tie nahi maante"**
```
Alice  1
Bob    2   ← forcefully alag
Carol  3   ← forcefully alag
Dave   4
Eve    5
```
> Use: dedup (sirf ek row chahiye per group)

**RANK — "Olympic style — tie pe same rank, gap aata hai"**
```
Alice  1
Bob    2   ← dono 2nd
Carol  2   ← dono 2nd (tie!)
Dave   4   ← 3 SKIP (GAP!)
Eve    5
```
> Use: actual position matter karta ho, top N problems

**DENSE_RANK — "Tie pe same rank, gap NAHI aata"**
```
Alice  1
Bob    2   ← dono 2nd
Carol  2   ← dono 2nd (tie!)
Dave   3   ← 3 skip nahi hua (NO GAP)
Eve    4
```
> Use: continuous ranking chahiye

**Side by side:**
```
Name   Time  ROW_NUMBER  RANK  DENSE_RANK
──────────────────────────────────────────
Alice  10s       1         1       1
Bob    12s       2         2       2
Carol  12s       3         2       2   ← same rank
Dave   15s       4         4       3   ← RANK gap (4), DENSE_RANK no gap (3)
Eve    18s       5         5       4
```

## Top N problems mein RANK vs DENSE_RANK

```
Electronics mein tie ho (Phone aur Watch dono 82000):

RANK:       Laptop=1, Phone=2, Watch=2, Tablet=4
            WHERE Rank <= 3 → 3 rows (CORRECT top 3)

DENSE_RANK: Laptop=1, Phone=2, Watch=2, Tablet=3
            WHERE Rank <= 3 → 4 rows (Tablet bhi aa gaya!)

Isliye top N problems mein RANK safer hai ties ke saath.
```

## LAG / LEAD — pichli ya agli row

```sql
LAG(Salary)  OVER (ORDER BY EffectiveDate)  -- pichli row ki value
LEAD(Salary) OVER (ORDER BY EffectiveDate)  -- agli row ki value

-- HAMESHA ORDER BY date column karo, value column nahi!
-- Pehli row ka LAG = NULL, aakhri row ka LEAD = NULL
```

## NTILE — buckets mein baato

```sql
NTILE(4) OVER (ORDER BY TotalSpend DESC)
-- Top 25% → bucket 1 (Platinum)
-- Next 25% → bucket 2 (Gold)
-- ...
-- Uneven rows → top buckets mein extra rows milti hain
```

## Performance

```
Window function = single table scan
CTE + JOIN = 2 scans
P15 real numbers: window 0.0147 vs CTE 0.0261 (44% sasta!)
```

## Quick reference

```
ROW_NUMBER  → unique, tie nahi
RANK        → gap on tie (Olympic)
DENSE_RANK  → no gap on tie
NTILE(n)    → n equal buckets
LAG         → pichli row
LEAD        → agli row
FIRST_VALUE → partition ki pehli value
LAST_VALUE  → partition ki aakhri value (full frame chahiye!)
```
