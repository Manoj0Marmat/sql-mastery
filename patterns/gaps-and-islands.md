# Gaps and Islands Pattern

## The Core Trick
```
Consecutive dates minus consecutive row numbers = constant value per streak

LoginDate    RowNum    Date - RowNum
2026-01-01     1       2025-12-31   ← streak A
2026-01-02     2       2025-12-31   ← streak A
2026-01-03     3       2025-12-31   ← streak A
2026-01-05     4       2026-01-01   ← streak B (gap breaks it)
2026-01-06     5       2026-01-01   ← streak B
2026-01-10     6       2026-01-04   ← streak C

The "gap" on Jan 4 shifts all row numbers by 1 → different constant
```

## The Pattern
```sql
WITH Numbered AS (
    SELECT DateColumn,
        DATEADD(DAY, -ROW_NUMBER() OVER (ORDER BY DateColumn), DateColumn) AS GroupKey
    FROM TableName
)
SELECT
    MIN(DateColumn) AS StreakStart,
    MAX(DateColumn) AS StreakEnd,
    COUNT(1) AS Length
FROM Numbered
GROUP BY GroupKey
ORDER BY StreakStart
```

## Per-User Streaks (add PARTITION BY)
```sql
WITH Numbered AS (
    SELECT UserID, LoginDate,
        DATEADD(DAY,
            -ROW_NUMBER() OVER (PARTITION BY UserID ORDER BY LoginDate),
            LoginDate) AS GroupKey
    FROM LoginDays
)
SELECT UserID,
    MIN(LoginDate) AS StreakStart,
    MAX(LoginDate) AS StreakEnd,
    COUNT(1) AS Days
FROM Numbered
GROUP BY UserID, GroupKey
ORDER BY UserID, StreakStart
```

## Real World Uses
- Login streak tracking (games, apps)
- Consecutive days a server was down
- Consecutive months without a payment
- Date ranges where a status didn't change
- Finding missing IDs in a numeric sequence
- Consecutive profitable days in financial data
