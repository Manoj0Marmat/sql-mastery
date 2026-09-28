# Window Frames

## What is a Frame?

Within a window partition, a frame defines which rows are included in each calculation.

```sql
FUNCTION() OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS BETWEEN <start> AND <end>   -- the frame
)
```

## Frame Boundary Keywords
```
UNBOUNDED PRECEDING  → from very first row of partition
N PRECEDING          → N rows back from current row
CURRENT ROW          → this row only
N FOLLOWING          → N rows ahead of current row
UNBOUNDED FOLLOWING  → to very last row of partition
```

## Four Common Frames

### Default Frame (when ORDER BY present)
```sql
-- No frame specified + ORDER BY present =
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- Accumulates from start to current row
-- Use: running totals
```

### Whole Partition Frame
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
-- Entire partition included for every row
-- Use: LAST_VALUE, partition-wide aggregates
```

### Moving Window Frame
```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- Current row + 2 rows before = 3-row window
-- Use: moving averages
```

### Remaining Rows Frame
```sql
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
-- Current row to end of partition
-- Use: remaining budget calculations
```

## LAST_VALUE Trap — most common mistake
```sql
-- WRONG — returns "last so far" not "last in partition"
LAST_VALUE(col) OVER (PARTITION BY ... ORDER BY ...)
-- Default frame = UNBOUNDED PRECEDING to CURRENT ROW
-- Changes every row!

-- CORRECT — full frame required
LAST_VALUE(col) OVER (
    PARTITION BY ...
    ORDER BY ...
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
-- Same value on every row in partition
```

## ROWS vs RANGE
```sql
-- ROWS: each row strictly separate (recommended)
SUM(salary) OVER (ORDER BY salary DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- RANGE: tie rows get same cumulative value (default behavior)
SUM(salary) OVER (ORDER BY salary DESC)
-- If 3 rows have same salary → all get the total of all 3, not running
-- Use ROWS for accurate running totals
```

## Quick Reference Card
```
Running total:    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
Moving 3-row avg: ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
LAST_VALUE fix:   ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
FIRST_VALUE:      No frame needed (default works correctly)
Whole partition:  No ORDER BY in OVER() (or ROWS UNBOUNDED PRECEDING AND FOLLOWING)
```
