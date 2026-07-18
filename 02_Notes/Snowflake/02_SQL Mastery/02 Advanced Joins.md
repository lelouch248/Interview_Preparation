---
tags:
  - snowflake/sql
  - joins
  - grain
  - reconciliation
---

# Advanced Joins

## Core Principle

> A join can be syntactically correct and still produce incorrect business results.

Before joining, determine:

```text
1. What does one row represent in the left table?
2. What does one row represent in the right table?
3. Is the join key unique on either side?
4. What should one output row represent?
5. Can the join multiply rows?
```

---

# Practice Table Grains

| Table | Grain |
|---|---|
| `CUSTOMERS` | One customer |
| `ORDERS` | One order |
| `PAYMENTS` | One payment attempt |
| `ORDER_ITEMS` | One item inside an order |

---

# Join Cardinality

## One-to-One

Each key appears at most once on each side.

## One-to-Many

Example:

```text
One customer → many orders
```

The customer row repeats for every matching order.

## Many-to-Many

Example:

```text
Many payments → one order ID ← many order items
```

Joining payments to items on `order_id` produces every matching payment-item combination.

---

# INNER JOIN

Returns matching combinations from both sides.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_amount
FROM CUSTOMERS c
INNER JOIN ORDERS o
    ON c.customer_id = o.customer_id;
```

Unmatched rows on either side are excluded.

---

# LEFT JOIN

Preserves every row from the left side.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_amount
FROM CUSTOMERS c
LEFT JOIN ORDERS o
    ON c.customer_id = o.customer_id;
```

When no right-side match exists, right-side columns become `NULL`.

Uses:

- Customers without orders
- Expected files without arrivals
- Configurations without executions
- Tables without grants
- Mappings without implementation

---

# ON vs WHERE in a LEFT JOIN

Incorrect when all customers must be preserved:

```sql
SELECT
    c.customer_id,
    o.order_id
FROM CUSTOMERS c
LEFT JOIN ORDERS o
    ON c.customer_id = o.customer_id
WHERE o.order_status = 'COMPLETED';
```

The `WHERE` clause removes null-padded unmatched rows.

Correct:

```sql
SELECT
    c.customer_id,
    o.order_id
FROM CUSTOMERS c
LEFT JOIN ORDERS o
    ON c.customer_id = o.customer_id
   AND o.order_status = 'COMPLETED';
```

Rule:

```text
ON    → decides right-side matches
WHERE → decides which final rows survive
```

---

# RIGHT JOIN

Preserves the right side.

```sql
SELECT
    c.customer_name,
    o.order_id
FROM CUSTOMERS c
RIGHT JOIN ORDERS o
    ON c.customer_id = o.customer_id;
```

For readability, it can often be rewritten as a left join by reversing table order.

---

# FULL OUTER JOIN

Preserves:

- Matches
- Left-only rows
- Right-only rows

```sql
SELECT
    c.customer_id AS master_customer_id,
    o.customer_id AS order_customer_id
FROM CUSTOMERS c
FULL OUTER JOIN (
    SELECT DISTINCT customer_id
    FROM ORDERS
) o
    ON c.customer_id = o.customer_id;
```

Useful for reconciliation.

---

# Reconciliation Classification

```sql
SELECT
    c.customer_id AS master_customer_id,
    o.customer_id AS order_customer_id,
    CASE
        WHEN c.customer_id IS NOT NULL
         AND o.customer_id IS NOT NULL
            THEN 'BOTH'
        WHEN c.customer_id IS NOT NULL
         AND o.customer_id IS NULL
            THEN 'MASTER_ONLY'
        WHEN c.customer_id IS NULL
         AND o.customer_id IS NOT NULL
            THEN 'ORDERS_ONLY'
    END AS reconciliation_status
FROM CUSTOMERS c
FULL OUTER JOIN (
    SELECT DISTINCT customer_id
    FROM ORDERS
) o
    ON c.customer_id = o.customer_id;
```

Why deduplicate orders first?

```text
The reconciliation is at customer grain, not order grain.
```

---

# USING vs ON

Concise syntax:

```sql
SELECT *
FROM CUSTOMERS
JOIN ORDERS
USING (customer_id);
```

Explicit syntax:

```sql
SELECT *
FROM CUSTOMERS c
JOIN ORDERS o
    ON c.customer_id = o.customer_id;
```

Use `ON` when:

- Column names differ
- Multiple conditions exist
- Transformations are required
- Explicit references improve readability

---

# Avoid NATURAL JOIN

`NATURAL JOIN` automatically joins on identically named columns.

Risk:

```text
If a new same-named column is added later, the join logic silently changes.
```

Prefer explicit `ON` or controlled `USING`.

---

# CROSS JOIN

Produces every possible left-right combination.

```sql
SELECT
    c.customer_id,
    d.calendar_date
FROM CUSTOMERS c
CROSS JOIN DATE_DIMENSION d;
```

If there are:

```text
100 customers × 365 dates = 36,500 rows
```

Valid uses:

- Complete reporting grids
- Scenario generation
- Test data
- Date expansion

Danger:

```sql
SELECT *
FROM CUSTOMERS c
JOIN ORDERS o;
```

A missing join condition can produce a Cartesian product.

---

# Semi Join with EXISTS

Question:

```text
Which customers have at least one completed order?
```

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM CUSTOMERS c
WHERE EXISTS (
    SELECT 1
    FROM ORDERS o
    WHERE o.customer_id = c.customer_id
      AND o.order_status = 'COMPLETED'
);
```

Use `EXISTS` when:

- Only left-side columns are required
- The requirement is about existence
- Right-side duplicates should not multiply output rows

---

# Anti Join with NOT EXISTS

Question:

```text
Which customers have no orders?
```

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM CUSTOMERS c
WHERE NOT EXISTS (
    SELECT 1
    FROM ORDERS o
    WHERE o.customer_id = c.customer_id
);
```

Alternative:

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM CUSTOMERS c
LEFT JOIN ORDERS o
    ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

`NOT EXISTS` often communicates intent more clearly.

---

# NOT IN and NULL

Avoid relying on:

```sql
WHERE customer_id NOT IN (
    SELECT customer_id
    FROM ORDERS
)
```

If the subquery contains `NULL`, the result can be surprising.

Prefer:

```sql
WHERE NOT EXISTS (...)
```

for null-safe anti-join logic.

---

# Nullable Join Keys

Regular equality does not match two null values.

```sql
NULL = NULL
```

does not evaluate to true.

Snowflake null-safe equality:

```sql
left_column IS NOT DISTINCT FROM right_column
```

Example:

```sql
SELECT
    c.customer_id,
    s.segment_description
FROM CUSTOMERS c
LEFT JOIN SEGMENTS s
    ON c.customer_segment
       IS NOT DISTINCT FROM
       s.segment_code;
```

> [!warning]
> Treating two nulls as equal must match the business meaning. Null can represent unknown, missing, or not applicable.

---

# Join Explosion

A join explosion occurs when a join outputs far more rows than expected.

Example:

```text
Order 1001
2 payments
2 items
```

Join result:

```text
2 × 2 = 4 rows
```

```sql
SELECT
    p.order_id,
    p.payment_id,
    i.item_id
FROM PAYMENTS p
JOIN ORDER_ITEMS i
    ON p.order_id = i.order_id;
```

This is not caused by ordinary duplicate rows. It is caused by a many-to-many relationship at the join key.

---

# Aggregate Before Joining

Incorrect pattern:

```sql
SELECT
    p.order_id,
    SUM(p.paid_amount),
    SUM(i.item_amount)
FROM PAYMENTS p
JOIN ORDER_ITEMS i
    ON p.order_id = i.order_id
GROUP BY p.order_id;
```

Measures are multiplied.

Correct:

```sql
WITH payment_summary AS (
    SELECT
        order_id,
        SUM(
            CASE
                WHEN payment_status = 'SUCCESS'
                THEN paid_amount
                ELSE 0
            END
        ) AS successful_paid_amount
    FROM PAYMENTS
    GROUP BY order_id
),
item_summary AS (
    SELECT
        order_id,
        SUM(item_amount) AS total_item_amount
    FROM ORDER_ITEMS
    GROUP BY order_id
)
SELECT
    o.order_id,
    o.order_amount,
    p.successful_paid_amount,
    i.total_item_amount
FROM ORDERS o
LEFT JOIN payment_summary p
    ON o.order_id = p.order_id
LEFT JOIN item_summary i
    ON o.order_id = i.order_id;
```

Every prepared dataset is one row per order.

---

# Grain Alignment Rule

```text
1. Define the desired final grain.
2. Identify each input grain.
3. Validate key uniqueness.
4. Deduplicate or aggregate inputs.
5. Join compatible grains.
6. Validate output cardinality.
```

For an order-level report:

```text
ORDERS          → one row per order
PAYMENT_SUMMARY → one row per order
ITEM_SUMMARY    → one row per order
```

---

# Diagnose Duplicate Keys

```sql
SELECT
    order_id,
    COUNT(*) AS row_count
FROM PAYMENTS
GROUP BY order_id
HAVING COUNT(*) > 1;
```

General check:

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT business_key) AS unique_keys,
    COUNT(*) - COUNT(DISTINCT business_key) AS excess_rows
FROM your_table;
```

Composite-key check:

```sql
SELECT
    key_1,
    key_2,
    COUNT(*) AS row_count
FROM your_table
GROUP BY key_1, key_2
HAVING COUNT(*) > 1;
```

---

# Measure Row Counts Incrementally

```sql
SELECT COUNT(*) FROM ORDERS;
```

```sql
SELECT COUNT(*)
FROM ORDERS o
JOIN PAYMENTS p
    ON o.order_id = p.order_id;
```

```sql
SELECT
    COUNT(*) AS joined_rows,
    COUNT(DISTINCT o.order_id) AS distinct_orders
FROM ORDERS o
JOIN PAYMENTS p
    ON o.order_id = p.order_id
JOIN ORDER_ITEMS i
    ON o.order_id = i.order_id;
```

Do not add `DISTINCT` merely to hide unexplained multiplication.

---

# Latest Record Before Joining

```sql
WITH latest_payment AS (
    SELECT *
    FROM PAYMENTS
    QUALIFY ROW_NUMBER() OVER (
        PARTITION BY order_id
        ORDER BY
            payment_timestamp DESC,
            payment_id DESC
    ) = 1
)
SELECT
    o.order_id,
    o.order_amount,
    p.payment_status,
    p.paid_amount
FROM ORDERS o
LEFT JOIN latest_payment p
    ON o.order_id = p.order_id;
```

The right side changes from:

```text
One row per payment attempt
```

to:

```text
One row per order
```

before joining.

---

# Self Join

```sql
SELECT
    customer.customer_name,
    referrer.customer_name AS referrer_name
FROM CUSTOMERS customer
LEFT JOIN CUSTOMERS referrer
    ON customer.referrer_customer_id = referrer.customer_id;
```

Uses:

- Employee-manager relationships
- Parent-child categories
- Referral networks
- Object dependencies

---

# Range Join

```sql
SELECT
    o.order_id,
    o.order_amount,
    d.band_name
FROM ORDERS o
LEFT JOIN DISCOUNT_BANDS d
    ON o.order_amount >= d.minimum_amount
   AND o.order_amount <= d.maximum_amount;
```

> [!warning]
> Overlapping ranges can cause one record to match multiple bands.

---

# ASOF JOIN

Use `ASOF JOIN` to match a left-side time-series record with the closest qualifying right-side record.

Example:

```sql
SELECT
    s.sale_id,
    s.product_id,
    s.sale_timestamp,
    p.effective_timestamp,
    p.price
FROM SALES_EVENTS s
ASOF JOIN PRODUCT_PRICES p
    MATCH_CONDITION (
        s.sale_timestamp >= p.effective_timestamp
    )
    ON s.product_id = p.product_id;
```

Use cases:

- Effective price at sale time
- Exchange rate at transaction time
- Customer classification active at order time
- Sensor state before an alert
- Mapping version effective during processing

---

# Query Profile

For problematic joins:

```text
Snowsight
→ Query History
→ Query Profile
```

Inspect:

- Join input rows
- Join output rows
- Cartesian joins
- Time spent in join operators
- Local or remote spilling
- Sudden cardinality growth

Programmatic operator inspection:

```sql
SET problematic_query_id = LAST_QUERY_ID();

SELECT *
FROM TABLE(
    GET_QUERY_OPERATOR_STATS($problematic_query_id)
);
```

---

# Interview Answers

## Why do joins create duplicate rows?

> Duplicate-looking output occurs when the join key is not unique on one or both sides. A one-to-many join repeats the one-side row, and a many-to-many join produces every matching combination.

## How do you prevent inflated totals?

> I define the desired output grain, aggregate or deduplicate each input to that grain, validate key uniqueness, and then perform the join.

## ON vs WHERE in a left join

> `ON` determines which right-side rows match while preserving the left side. A right-table filter in `WHERE` can remove unmatched null-padded rows and accidentally create inner-join behavior.

## EXISTS vs JOIN

> I use `EXISTS` when I only need to know whether a matching row exists and do not need right-side columns.

## What is an ASOF JOIN?

> An ASOF JOIN matches each left-side time-series row with one closest qualifying right-side record based on an inequality timestamp condition.

---

# Production Join Checklist

- [ ] Left-table grain identified
- [ ] Right-table grain identified
- [ ] Desired output grain identified
- [ ] Join key uniqueness checked
- [ ] Cardinality classified
- [ ] Null-key behavior decided
- [ ] Unmatched-row preservation decided
- [ ] `ON` and `WHERE` filters reviewed
- [ ] Measure multiplication checked
- [ ] Inputs aggregated or deduplicated when needed
- [ ] Output row count validated
- [ ] Query Profile inspected when cardinality is unexpected

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
