---
tags:
  - snowflake
  - mistakes
  - revision
---

# Snowflake Mistakes Log

Use this file whenever a query fails, produces unexpected rows, or contains a conceptual mistake.

---

## Mistake 001 — Filtering a LEFT JOIN in WHERE

### Incorrect

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_status = 'COMPLETED';
```

### Why It Fails

The `WHERE` condition removes unmatched rows where `o.order_status` is `NULL`.

### Correct

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.order_status = 'COMPLETED';
```

### Lesson

`ON` controls matching. `WHERE` controls final row survival.

---

## Mistake 002 — Aggregating Event History as Current Data

### Incorrect

```sql
SELECT
    customer_id,
    SUM(amount)
FROM order_events
GROUP BY customer_id;
```

### Why It Fails

The order amount repeats for every status event.

### Correct Approach

Select one current row per order before aggregating.

### Lesson

Understand grain before aggregation.

---

## Mistake 003 — Using RANK for Deduplication

### Problem

Tied values can both receive rank 1.

### Correct

Use `ROW_NUMBER` with a deterministic tie-breaker when one record must win.

---

## Mistake 004 — Joining Raw Many-Side Tables

### Problem

Payments and order items both contain multiple rows per order.

### Result

Measures multiply after the join.

### Correct

Aggregate both tables to one row per order before joining.

---

## Mistake 005 — Blind DISTINCT

### Problem

`DISTINCT` can hide an unexplained data-model or join problem.

### Rule

Understand why rows repeat before removing them.

---

# New Mistake Template

## Mistake XXX — Title

### Context

### Incorrect SQL

```sql

```

### Actual Result

### Expected Result

### Root Cause

### Correct SQL

```sql

```

### Lesson

### Related Note

- [[]]

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[99_Templates/Snowflake Real Life Problem Template|Real Life Problem Template]]
