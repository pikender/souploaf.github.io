---
layout: post
title: Power up the Stats Pages 
---

>
  What you measure is what you get

Like every successful business app, our's also shows important metrics
relevant to business for daily operations and adjusting to strategies.

As already well-known, its mostly aggregation of data over a column.

As already guessed, the referenced tables must be centre to Business as
shown by Metric interest and liable to be holding huge data.

Over time, the system changes to accomodate new requirements and hype
and in the chase, such pages are left unattended.

As page hit of stats page is very less so slow response is also well
accepted <doing lot of data crunching> but it hits bad when DB is
competed for different queries.

Best strategy is to make the DB free ASAP.

I know, lot of theory and nothing substantial still.

Here, it is.

We looked at stats page and found its taking lot of time just showing
some graphs.

Time taken was surprising.

Like every other developer, we turned to indexes in the queried tables
and found that most of indexes are stale now as data is getting
aggregated on different columns than indexed for.

**SAMPLE SQL**

```sql
SELECT `subscription_lists`.*
FROM `subscription_lists`
WHERE (subscription = 1 && (created_at >= "2012-02-01 00:00:00" AND
created_at <= "2012-10-05 23:59:59"))
ORDER BY created_at ASC 
```

**INDEXES**

```sql
SHOW INDEXES FROM subscription_lists;
```

**Indexes Found**

- Primary Index
- Index on `status` found

As query is operating on `status` and `created_at`, it was
straightforward to add a compound index on **`[status, created_at]`**

Check the index used by checking the query plan using **EXPLAIN**

```sql
EXPLAIN
SELECT `subscription_lists`.*
FROM `subscription_lists`
WHERE (subscription = 1 && (created_at >= "2012-02-01 00:00:00" AND
created_at <= "2012-10-05 23:59:59"))
ORDER BY created_at ASC 
```

In the output, look for *possible_keys*, *key*, *rows* column to verify
whether new index is used or not

It should list index in *possible_keys* and also in *key* and rows
scanned should also reduce drastically

**_QUIZ_**

Will the index be used for below query ?

*Note:* Now subscription = 1 has changed to subscription != 1

```sql
SELECT `subscription_lists`.*
FROM `subscription_lists`
WHERE (subscription != 1 && (created_at >= "2012-02-01 00:00:00" AND
created_at <= "2012-10-05 23:59:59"))
ORDER BY created_at ASC 
```

Run *Explain*

In the output, *key* column would not be listing the index

*Tip:* Use *`IN`* queries for *NEGATION* CASES
