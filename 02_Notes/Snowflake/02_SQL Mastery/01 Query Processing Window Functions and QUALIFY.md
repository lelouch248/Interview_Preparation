---
tags:
  - snowflake/sql
  - window-functions
  - qualify
  - cte
---

# Query Processing, Window Functions, and QUALIFY

## Practice Dataset

```sql
CREATE OR REPLACE TABLE RAW.ORDER_EVENTS (
    event_id NUMBER,
    order_id NUMBER,
    customer_id NUMBER,
    event_timestamp TIMESTAMP_NTZ,
    order_status STRING,
    amount NUMBER(10, 2)
);

INSERT INTO RAW.ORDER_EVENTS VALUES
    (1, 1001, 101, '2026-06-01 09:00:00', 'PLACED',    1200),
    (2, 1001, 101, '2026-06-01 11:00:00', 'CONFIRMED', 1200),
    (3, 1001, 101, '2026-06-02 10:00:00', 'SHIPPED',   1200),
    (4, 1002, 102, '2026-06-01 10:00:00', 'PLACED',     800),
    (5, 1002, 102, '2026-06-01 14:00:00', 'CANCELLED',  800),
    (6, 1003, 101, '2026-06-03 08:00:00', 'PLACED',     500),
    (7, 1003, 101, '2026-06-03 12:00:00', 'CONFIRMED',  500),
    (8, 1004, 103, '2026-06-04 09:00:00', 'PLACED',    1500),
    (9, 1004, 103, '2026-06-04 16:00:00', 'SHIPPED',   1500),
    (10, 1005, 102, '2026-06-05 10:00:00', 'PLACED',    800);
```

Grain:

```text
One row represents one event in an order's lifecycle.
```

---

# Logical Query-Processing Order

A useful logical order is:

```text
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. WINDOW
6. QUALIFY
7. DISTINCT
8. ORDER BY
9. LIMIT
```

This explains why a window-function alias cannot normally be used in `WHERE`.

Incorrect:

```sql
SELECT
    *,
    ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp DESC
    ) AS row_num
FROM ORDER_EVENTS
WHERE row_num = 1;
```

Correct:

```sql
SELECT
    *,
    ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp DESC
    ) AS row_num
FROM ORDER_EVENTS
QUALIFY row_num = 1;
```

---

# WHERE vs HAVING vs QUALIFY

| Clause | Filters | Stage |
|---|---|---|
| `WHERE` | Source rows | Before grouping |
| `HAVING` | Aggregated groups | After `GROUP BY` |
| `QUALIFY` | Window results | After window calculation |

## WHERE

```sql
SELECT *
FROM ORDER_EVENTS
WHERE order_status = 'SHIPPED';
```

## HAVING

```sql
SELECT
    customer_id,
    SUM(amount) AS event_amount
FROM ORDER_EVENTS
GROUP BY customer_id
HAVING SUM(amount) > 2000;
```

## QUALIFY

```sql
SELECT *
FROM ORDER_EVENTS
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY order_id
    ORDER BY event_timestamp DESC
) = 1;
```

Memory aid:

```text
WHERE   → raw rows
HAVING  → grouped rows
QUALIFY → analytical rows
```

---

# Window Function Mental Model

A window function performs a calculation across related rows while preserving the individual rows.

## `GROUP BY`

```sql
SELECT
    customer_id,
    SUM(amount)
FROM ORDER_EVENTS
GROUP BY customer_id;
```

Result grain:

```text
One row per customer
```

## Window Function

```sql
SELECT
    event_id,
    customer_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
    ) AS customer_total
FROM ORDER_EVENTS;
```

Result grain remains:

```text
One row per event
```

Core difference:

```text
GROUP BY collapses rows.
Window functions usually preserve rows.
```

---

# Window Function Anatomy

```sql
ROW_NUMBER() OVER (
    PARTITION BY order_id
    ORDER BY event_timestamp DESC
)
```

Components:

```text
ROW_NUMBER()              → calculation
PARTITION BY order_id     → grouping boundary
ORDER BY timestamp DESC   → sequence within the partition
```

Generic pattern:

```sql
FUNCTION_NAME(expression) OVER (
    PARTITION BY grouping_columns
    ORDER BY sequencing_columns
    ROWS BETWEEN frame_start AND frame_end
)
```

---

# ROW_NUMBER

`ROW_NUMBER` assigns a unique sequence inside each partition.

```sql
SELECT
    order_id,
    event_timestamp,
    order_status,
    ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp
    ) AS event_sequence
FROM ORDER_EVENTS;
```

Use it for:

- Latest record per key
- First record per key
- Deduplication
- Top N per group
- Selecting exactly one winner

---

# Latest Record per Business Key

```sql
SELECT *
FROM ORDER_EVENTS
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY order_id
    ORDER BY
        event_timestamp DESC,
        event_id DESC
) = 1;
```

## Why the tie-breaker matters

Weak ordering:

```sql
ORDER BY event_timestamp DESC
```

If two rows share the same timestamp, the winning row may not be deterministic.

Better:

```sql
ORDER BY
    event_timestamp DESC,
    event_id DESC
```

Possible tie-breakers:

- Event ID
- Ingestion timestamp
- Batch ID
- Source sequence
- File row number
- Update sequence

> [!important]
> A production deduplication rule must make the winning row explainable.

---

# ROW_NUMBER vs RANK vs DENSE_RANK

Suppose sales are:

| Customer | Sales |
|---|---:|
| Anu | 5000 |
| Bharat | 4000 |
| Charan | 4000 |
| Divya | 3000 |

Result:

| Customer | `ROW_NUMBER` | `RANK` | `DENSE_RANK` |
|---|---:|---:|---:|
| Anu | 1 | 1 | 1 |
| Bharat | 2 | 2 | 2 |
| Charan | 3 | 2 | 2 |
| Divya | 4 | 4 | 3 |

Use:

| Requirement | Function |
|---|---|
| Exactly one winner | `ROW_NUMBER` |
| Competition rank with gaps | `RANK` |
| Rank levels without gaps | `DENSE_RANK` |

---

# Top N per Group

Find the two largest current orders per customer:

```sql
WITH latest_orders AS (
    SELECT *
    FROM ORDER_EVENTS
    QUALIFY ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp DESC, event_id DESC
    ) = 1
)
SELECT
    customer_id,
    order_id,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY amount DESC, order_id
    ) AS customer_order_rank
FROM latest_orders
QUALIFY customer_order_rank <= 2;
```

---

# LAG

`LAG` accesses a previous row according to the window ordering.

```sql
SELECT
    order_id,
    event_timestamp,
    order_status,
    LAG(order_status) OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp, event_id
    ) AS previous_status
FROM ORDER_EVENTS;
```

Uses:

- Previous status
- Previous amount
- Previous update time
- Change detection
- Period-over-period comparison

---

# Change Detection

```sql
WITH status_comparison AS (
    SELECT
        *,
        LAG(order_status) OVER (
            PARTITION BY order_id
            ORDER BY event_timestamp, event_id
        ) AS previous_status
    FROM ORDER_EVENTS
)
SELECT *
FROM status_comparison
WHERE order_status IS DISTINCT FROM previous_status;
```

`IS DISTINCT FROM` is null-safe.

Unlike:

```sql
order_status <> previous_status
```

it handles cases where either side is `NULL`.

---

# LEAD

`LEAD` accesses a following row.

```sql
SELECT
    order_id,
    order_status,
    event_timestamp AS valid_from,
    LEAD(event_timestamp) OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp, event_id
    ) AS valid_until
FROM ORDER_EVENTS;
```

Uses:

- Next event
- Next effective timestamp
- Building validity intervals
- SCD Type 2 logic
- Session boundaries

---

# Running Totals

```sql
SELECT
    transaction_id,
    customer_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY transaction_date, transaction_id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM CUSTOMER_TRANSACTIONS;
```

## Window Frame

```text
UNBOUNDED PRECEDING → first row in the partition
CURRENT ROW         → current row
2 PRECEDING         → two rows before current row
UNBOUNDED FOLLOWING → final row in the partition
```

Moving three-row average:

```sql
AVG(amount) OVER (
    PARTITION BY customer_id
    ORDER BY transaction_date, transaction_id
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

> [!tip]
> Write frames explicitly when the calculation depends on row boundaries.

---

# Common Table Expressions

A CTE is a named query block created with `WITH`.

```sql
WITH latest_orders AS (
    SELECT *
    FROM ORDER_EVENTS
    QUALIFY ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp DESC, event_id DESC
    ) = 1
),
valid_orders AS (
    SELECT *
    FROM latest_orders
    WHERE order_status <> 'CANCELLED'
),
customer_summary AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(amount) AS total_order_value
    FROM valid_orders
    GROUP BY customer_id
)
SELECT *
FROM customer_summary
WHERE total_order_value >= 1000;
```

Good CTE names:

```text
latest_orders
deduplicated_source
valid_orders
changed_records
customer_summary
```

Poor CTE names:

```text
cte1
cte2
temp
abc
final2
```

---

# Critical Grain Example

Incorrect:

```sql
SELECT
    customer_id,
    SUM(amount)
FROM ORDER_EVENTS
GROUP BY customer_id;
```

Why wrong?

The event table repeats the order amount for every status event.

Correct:

```sql
WITH latest_orders AS (
    SELECT *
    FROM ORDER_EVENTS
    QUALIFY ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY event_timestamp DESC, event_id DESC
    ) = 1
)
SELECT
    customer_id,
    SUM(amount) AS current_order_value
FROM latest_orders
WHERE order_status <> 'CANCELLED'
GROUP BY customer_id;
```

> [!important]
> Understand the table grain before aggregating.

---

# Interview Answers

## Window function

> A window function performs an analytical calculation across related rows while preserving the individual rows. The `OVER` clause defines the partition, ordering, and optionally the frame.

## GROUP BY vs window function

> `GROUP BY` collapses rows into one row per group. A window function generally keeps the source rows and adds calculations across related rows.

## Latest record per ID

> I use `ROW_NUMBER` partitioned by the business key and ordered by the update timestamp descending. I add a deterministic tie-breaker and filter with `QUALIFY`.

## WHERE vs HAVING vs QUALIFY

> `WHERE` filters source rows, `HAVING` filters grouped results, and `QUALIFY` filters window-function results.

---

# Mastery Checklist

- [ ] Explain logical query order
- [ ] Use `QUALIFY`
- [ ] Select latest record per key
- [ ] Use deterministic ordering
- [ ] Compare `ROW_NUMBER`, `RANK`, and `DENSE_RANK`
- [ ] Use `LAG`
- [ ] Use `LEAD`
- [ ] Build running totals
- [ ] Explain window frames
- [ ] Explain grain before aggregation

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
