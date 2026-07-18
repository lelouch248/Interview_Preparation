---
tags:
  - snowflake
  - sql-patterns
  - revision
---

# Snowflake SQL Patterns

## Session Context

```sql
USE ROLE SYSADMIN;
USE WAREHOUSE WH_SNOWFLAKE_LEARNING;
USE DATABASE SNOWFLAKE_MASTERY_DB;
USE SCHEMA RAW;
```

## Inspect Context

```sql
SELECT
    CURRENT_USER(),
    CURRENT_ROLE(),
    CURRENT_WAREHOUSE(),
    CURRENT_DATABASE(),
    CURRENT_SCHEMA();
```

## Latest Row per Business Key

```sql
SELECT *
FROM source_table
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY business_key
    ORDER BY updated_at DESC, ingestion_id DESC
) = 1;
```

## First Row per Business Key

```sql
SELECT *
FROM source_table
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY business_key
    ORDER BY created_at, ingestion_id
) = 1;
```

## Top N per Group

```sql
SELECT *
FROM source_table
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY group_key
    ORDER BY measure DESC, business_key
) <= 3;
```

## Previous Value

```sql
LAG(status) OVER (
    PARTITION BY business_key
    ORDER BY event_timestamp, event_id
)
```

## Next Effective Timestamp

```sql
LEAD(effective_timestamp) OVER (
    PARTITION BY business_key
    ORDER BY effective_timestamp
)
```

## Change Detection

```sql
WITH compared AS (
    SELECT
        *,
        LAG(attribute_value) OVER (
            PARTITION BY business_key
            ORDER BY updated_at, ingestion_id
        ) AS previous_value
    FROM source_table
)
SELECT *
FROM compared
WHERE attribute_value IS DISTINCT FROM previous_value;
```

## Running Total

```sql
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY transaction_timestamp, transaction_id
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

## Semi Join

```sql
SELECT *
FROM left_table l
WHERE EXISTS (
    SELECT 1
    FROM right_table r
    WHERE r.business_key = l.business_key
);
```

## Anti Join

```sql
SELECT *
FROM left_table l
WHERE NOT EXISTS (
    SELECT 1
    FROM right_table r
    WHERE r.business_key = l.business_key
);
```

## Null-Safe Join

```sql
SELECT *
FROM left_table l
JOIN right_table r
    ON l.nullable_key
       IS NOT DISTINCT FROM
       r.nullable_key;
```

## Preserve Left Rows with Filtered Matches

```sql
SELECT *
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.order_status = 'COMPLETED';
```

## Aggregate Before Join

```sql
WITH payment_summary AS (
    SELECT
        order_id,
        SUM(paid_amount) AS paid_amount
    FROM payments
    WHERE payment_status = 'SUCCESS'
    GROUP BY order_id
),
item_summary AS (
    SELECT
        order_id,
        SUM(item_amount) AS item_amount
    FROM order_items
    GROUP BY order_id
)
SELECT
    o.order_id,
    p.paid_amount,
    i.item_amount
FROM orders o
LEFT JOIN payment_summary p
    ON o.order_id = p.order_id
LEFT JOIN item_summary i
    ON o.order_id = i.order_id;
```

## Full Reconciliation

```sql
SELECT
    s.business_key AS source_key,
    t.business_key AS target_key,
    CASE
        WHEN s.business_key IS NOT NULL
         AND t.business_key IS NOT NULL
            THEN 'BOTH'
        WHEN s.business_key IS NOT NULL
            THEN 'SOURCE_ONLY'
        ELSE 'TARGET_ONLY'
    END AS reconciliation_status
FROM source_keys s
FULL OUTER JOIN target_keys t
    ON s.business_key = t.business_key;
```

## ASOF JOIN

```sql
SELECT *
FROM events e
ASOF JOIN effective_values v
    MATCH_CONDITION (
        e.event_timestamp >= v.effective_timestamp
    )
    ON e.business_key = v.business_key;
```

## Duplicate-Key Check

```sql
SELECT
    business_key,
    COUNT(*) AS row_count
FROM source_table
GROUP BY business_key
HAVING COUNT(*) > 1;
```

## Cardinality Check

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT business_key) AS unique_keys,
    COUNT(*) - COUNT(DISTINCT business_key) AS excess_rows
FROM source_table;
```

## Navigation

- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Notes Index]]
- [[06_Revision/Snowflake/Snowflake Mistakes Log|Snowflake Mistakes Log]]
