---
title: "When a Placeholder Becomes Data: The Story of a [NONE] Broker"
date: 2026-11-03
categories: [Engineering, Data Quality]
tags: [data quality, sql, stored procedures, api, debugging, databases]
description: "QA flagged a broker showing as [NONE]. The API and the stored procedure were fine. The placeholder was stored as real data, and here is why that matters and how to prevent it."
image: https://plus.unsplash.com/premium_photo-1733306464128-d6b80ed2f2e2?q=80&w=1170&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=NHwxMjA3fDA%3D
---

**When a Placeholder Becomes Data**

QA flagged a record where the broker came back as `[NONE]`. My first stop was the API, then the stored procedure. Both were innocent. When I looked at the table, the value was right there in the broker column, saved as the text `[NONE]`.

That changed the question. It was no longer "why is the output wrong?" It became "how did a label for nothing end up stored as something?"

**Null and "[NONE]" are not the same thing**

A null says "no value here," and every database knows how to treat it. A string reading `[NONE]` says "here is a value," and the database believes it. That difference is the whole problem.

Because `[NONE]` is a real string, it slips past everything built to catch missing data:

- `IS NULL` checks do not find it.
- `NOT NULL` constraints are satisfied by it.
- `COALESCE` and `ISNULL` leave it alone because there is nothing to replace.
- A join to the broker table fails quietly, since no broker has the code `[NONE]`.

The record looks complete and valid. It is neither.

**How a placeholder gets into the data**

*A legacy or upstream system wrote it.* Older systems often use a bracketed label to say "nothing selected." When that data is imported, the label travels along as if it were a real value.

*A dropdown default got saved.* If a form offers `[NONE]` as the first option and nothing forces a choice, users submit it without noticing.

*An import or migration filled the gap.* A script that needed something in a column may have written `[NONE]` instead of leaving it empty.

*A column default did it.* A default value on the column can quietly stamp every row where the broker was not supplied.

**Why it is hard to catch**

Nothing fails. There is no exception, no constraint violation, and no broken join error. The data satisfies every rule we wrote, because we never wrote a rule against it. It sits there until a human, usually QA, reads the screen and notices that something looks off.

**How to avoid it**

*Use null for absence.* If a broker is genuinely missing, store null. It is the one value every tool understands as "nothing here."

*Or give "unknown" a real home.* If the business needs an explicit "no broker" state, add a proper row to the broker table and reference it with a foreign key. Now it is a valid, joinable value instead of a magic string.

*Clean at the door.* Convert placeholder strings such as `[NONE]`, `N/A`, `NULL`, and empty text to a real null as data enters the system. Do this in one place so every path benefits.

*Block it with a constraint.* A check constraint that rejects known placeholder strings turns a silent problem into an immediate, visible one.

*Go looking for it.* Run a simple profile query on key columns to see which values appear most often. Placeholders show up fast:

```sql
SELECT broker, COUNT(*) AS total
FROM records
GROUP BY broker
ORDER BY total DESC;
```

*Fix what is already there.* Once you know the cause, backfill the existing rows so history is consistent with the new rule.

**The takeaway**

A placeholder is a message for a human. A database cannot read messages, only values. The moment a label for "nothing" is stored as text, the system stops treating it as missing and starts trusting it. Keep absence as null, or model it properly, and your constraints, joins, and checks can do the job they were built for.

Happy Life.

---