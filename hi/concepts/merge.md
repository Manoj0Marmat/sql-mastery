# MERGE — Hindi in English

## Kya hai MERGE?

Ek hi statement mein INSERT + UPDATE + DELETE — match condition ke basis pe.

## Ration Card Office analogy

Officer ke paas purani list hai (TARGET). Government ne nayi list bheji (SOURCE).

Officer ek-ek naam check karta hai:
```
"Ye naam dono lists mein hai?"       → MATCHED           → details update karo
"Nayi list mein hai, purani mein nahi?" → NOT MATCHED BY TARGET → add karo
"Purani list mein hai, nayi mein nahi?" → NOT MATCHED BY SOURCE → hatao / inactive karo
```

## TARGET vs SOURCE — yaad karo

```
TARGET = jo badlega (main table — production data)
SOURCE = jo compare karega (naya feed) — KABHI NAHI badlega

Naya data aata hai SOURCE se, changes hote hain TARGET mein.
```

## Skeleton — 4 lines yaad karo

```
MERGE  Target            ← kahan jaana hai
USING  Source            ← kahan se aa raha hai
ON     T.ID = S.ID       ← kaise pehchano (unique key!)
WHEN   ... THEN ...;     ← kya karo (semicolon MUST at end!)
```

## Full syntax

```sql
MERGE #EmpTarget AS T
USING #EmpStaging AS S
ON T.EmpID = S.EmpID

WHEN MATCHED THEN
    UPDATE SET T.Department = S.Department,
               T.Salary     = S.Salary

WHEN NOT MATCHED BY TARGET THEN
    INSERT (EmpID, EmpName, Department, Salary)
    VALUES (S.EmpID, S.EmpName, S.Department, S.Salary)

WHEN NOT MATCHED BY SOURCE THEN
    UPDATE SET T.IsActive = 0;    -- soft delete — production mein DELETE nahi karte
```

## Allowed actions — trap!

```
NOT MATCHED BY TARGET → sirf INSERT (row exist hi nahi target mein, update kaise?!)
NOT MATCHED BY SOURCE → UPDATE ya DELETE dono allowed
MATCHED               → UPDATE ya DELETE
```

## OUTPUT — har action log karo

```sql
OUTPUT $action, inserted.EmpID, inserted.EmpName
INTO #AuditLog
-- $action automatic return karta hai: 'INSERT', 'UPDATE', 'DELETE'
```

## 3 questions memory trick

```
Dono mein hai?                → MATCHED           → UPDATE
Source mein, target nahi?     → NOT MATCHED BY TARGET → INSERT
Target mein, source nahi?     → NOT MATCHED BY SOURCE → DELETE/soft-delete
```

## Performance warning

```
MERGE + high concurrency = deadlock risk (Microsoft ne officially acknowledge kiya)
Billion-row tables pe avoid karo
Nightly ETL / batch jobs pe theek hai
```

## Quick reference

```
MERGE Target AS T / USING Source AS S / ON key = key
MATCHED → UPDATE
NOT MATCHED BY TARGET → INSERT only
NOT MATCHED BY SOURCE → UPDATE / DELETE
OUTPUT $action → audit log
; → semicolon end mein mandatory
```
