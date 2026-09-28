# Window Frames — Hindi in English

## Frame kya hai?

Window partition ke andar — frame define karta hai ki **kaunsi rows calculation mein include hongi** current row ke liye.

## Queue analogy

500 log line mein khade hain (partition). Tum bich mein ho.

```
UNBOUNDED PRECEDING = "line ki bilkul shuruaat se"    → pehle wale sab
N PRECEDING         = "N log pehle se"                → sirf N log peeche
CURRENT ROW         = "sirf main"                     → bas meri row
N FOLLOWING         = "N log aage tak"                → N log aage
UNBOUNDED FOLLOWING = "line ke bilkul end tak"        → baad wale sab
```

## Chaar common frames

### Default frame (jab ORDER BY ho)
```sql
-- Frame specify nahi kiya + ORDER BY hai →
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- Shuruaat se current row tak — running total ke liye
```

### Whole partition frame
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
-- Poori line — har row ke liye same value
-- LAST_VALUE ke liye zaruri!
```

### Moving window frame
```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- Mujhse 2 peeche + main = 3 log → moving average
```

### Remaining rows frame
```sql
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
-- Mere se aage sab → remaining budget
```

## LAST_VALUE trap — sabse common galti

```sql
-- WRONG — "abhi tak ka last" return karta hai, "partition ka last" nahi
LAST_VALUE(col) OVER (PARTITION BY ... ORDER BY ...)
-- Default frame = UNBOUNDED PRECEDING to CURRENT ROW
-- Har row pe alag value aata hai!

-- CORRECT — poori partition dekho
LAST_VALUE(col) OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
-- Har row pe same value — partition ka last
```

**Yaad karo:** LAST_VALUE pe hamesha full frame likho. FIRST_VALUE ko nahi chahiye.

## ROWS vs RANGE

```sql
-- ROWS — har row strictly alag (recommended)
SUM(Salary) OVER (ORDER BY Salary DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- RANGE — tie rows ko same cumulative value milti hai (default behavior)
SUM(Salary) OVER (ORDER BY Salary DESC)
-- 3 rows ki salary same hai → teeno ko total of all 3 milega, not running
-- Accurate running total ke liye ROWS use karo
```

## Quick reference card

```
Running total:     ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
Moving 3-row avg:  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
LAST_VALUE fix:    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
FIRST_VALUE:       Frame nahi chahiye (default sahi kaam karta hai)
Whole partition:   No ORDER BY in OVER() ya ROWS UNBOUNDED BOTH SIDES
```

## Ek line memory trick

```
LAST_VALUE = hamesha full frame
FIRST_VALUE = koi frame nahi
Running total = PRECEDING AND CURRENT ROW
Moving avg = N PRECEDING AND CURRENT ROW
```
