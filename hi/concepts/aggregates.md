# Aggregates — Hindi in English

## Execution Order — Yaad karo ek baar

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

Desi trick: **"FW GH SO"** — From Where Group Having Select Order

## Functions
```sql
COUNT(*)     -- saari rows gino, NULL bhi
COUNT(col)   -- sirf non-NULL gino
SUM(col)     -- jodo sab
AVG(col)     -- average nikalo (INT pe dhyan do!)
MIN / MAX    -- sabse chhota / sabse bada
```

## INT trap — Desi style

**Problem:** Salary INT hai. Average nikalo.
```sql
AVG(Salary)  -- 75000 + 85000 = 160000 / 2 = 80000.00? Nahi!
             -- INT / INT = INT → 80000 (decimal gaya!)
```

Ek aadmi ke paas 1 roti, doosre ke paas 2 roti. Average = 1.5 roti.
But INT mein = 1 roti. Adhi roti "gum" ho gayi.

**Fix:**
```sql
CAST(AVG(CAST(Salary AS DECIMAL(10,2))) AS DECIMAL(10,2))
```

## WHERE vs HAVING — Chai ki dukaan

Chai ki dukaan pe records hain:
```
WHERE   = Customers ko filter karo LINE mein lagane se PEHLE
          (50 se kam umar wale andar nahi aane denge)

HAVING  = Groups banao, PHIR groups filter karo
          (sirf wo tables rakho jahan 3 se zyada orders aaye hain)
```

```sql
-- WRONG — aggregate WHERE mein nahi hota
WHERE COUNT(1) > 3    -- ERROR

-- CORRECT
HAVING COUNT(1) > 3   -- groups filter karo
```

## CASE inside aggregate — Ration system

```sql
-- Gino: kitne log "High" earner hain
SUM(CASE WHEN Salary >= 85000 THEN 1 ELSE 0 END) AS HighEarners
-- Har row ke liye: agar condition match → 1, nahi → 0
-- SUM = kitne 1 hain = count of high earners
-- ELSE 0 use karo, ELSE NULL nahi → warna NULL output aayega
```

## Alias rules
```sql
-- HAVING mein alias kaam nahi karta (SELECT se pehle resolve hota hai)
HAVING NoOfEmp > 3       -- ERROR: column not found
HAVING COUNT(1) > 3      -- CORRECT

-- ORDER BY mein alias kaam karta hai (last mein resolve hota hai)
ORDER BY NoOfEmp DESC    -- WORKS
```

## Quick reference
```
GROUP BY  → groups banao
HAVING    → groups filter karo (GROUP BY ke baad)
WHERE     → rows filter karo (GROUP BY se pehle)
COUNT(*)  → sab gino | COUNT(col) → sirf non-NULL
AVG(INT)  → decimal gum → pehle DECIMAL mein CAST karo
```
