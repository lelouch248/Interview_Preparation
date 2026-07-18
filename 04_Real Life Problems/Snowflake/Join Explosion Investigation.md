---
tags:
  - snowflake
  - real-life-problem
  - joins
  - debugging
---

# Join Explosion Investigation

## Problem

An order-level report returns inflated payment and item totals.

## Source Grains

| Table | Grain |
|---|---|
| `ORDERS` | One row per order |
| `PAYMENTS` | One row per payment attempt |
| `ORDER_ITEMS` | One row per order item |

## Problematic Query

```sql
SELECT
    p.order_id,
    SUM(p.paid_amount) AS total_paid,
    SUM(i.item_amount) AS total_item_amount
FROM PAYMENTS p
JOIN ORDER_ITEMS i
    ON p.order_id = i.order_id
GROUP BY p.order_id;
```

## Why It Fails

For one order:

```text
2 payment rows
2 item rows
```

The join generates:

```text
2 × 2 = 4 rows
```

Each payment is repeated for every item, and each item is repeated for every payment.

## Incorrect Fix

```sql
SUM(DISTINCT p.paid_amount)
```

This is unsafe because two legitimate payment rows may contain the same amount.

## Correct Design

Aggregate each dataset to the final report grain before joining.

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

## Debugging Checklist

1. Identify the grain of every table.
2. Check duplicate key counts.
3. Measure row counts after every join.
4. Compare total rows with distinct business keys.
5. Inspect Query Profile.
6. Look for join operators with large output growth.
7. Aggregate or deduplicate before joining.
8. Validate financial totals against source systems.

## Lessons

- A shared key does not imply compatible grain.
- A syntactically correct join can be logically wrong.
- Do not use `DISTINCT` to hide unexplained duplication.
- Final report grain must be designed before the join.
- Measures should be aggregated at their natural grain.

## Navigation

- [[02_Notes/Snowflake/02_SQL Mastery/02 Advanced Joins|Advanced Joins]]
- [[06_Revision/Snowflake/Snowflake Mistakes Log|Snowflake Mistakes Log]]
