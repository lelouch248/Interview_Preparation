---
tags:
  - snowflake
  - revision
---

# Snowflake Quick Revision

## Architecture

```text
Snowflake = Storage + Compute + Cloud Services
```

- Storage holds centralized columnar data.
- Virtual warehouses provide independent compute.
- Cloud services handle metadata, authentication, optimization, and transactions.

## Core Triangle

```text
Database = where objects live
Warehouse = what runs queries
Role = what gives permission
```

## Warehouse Tuning

```text
One query is slow    → inspect SQL; consider scale up
Many queries queued  → consider scale out
Different workloads  → separate warehouses
```

## Table Types

| Type | Persists | Fail-safe | Use |
|---|---:|---:|---|
| Permanent | Yes | Yes | Critical production data |
| Transient | Yes | No | Rebuildable staging data |
| Temporary | Session only | No | Scratch processing |

## Access Requirements

Usually required to query a table:

```text
USAGE on warehouse
USAGE on database
USAGE on schema
SELECT on table/view
```

## Recovery

```text
Time Travel → user-accessible historical recovery
Fail-safe   → Snowflake-managed last-resort recovery
Clone       → fast logical copy
```

## Query Processing

```text
FROM
WHERE
GROUP BY
HAVING
WINDOW
QUALIFY
DISTINCT
ORDER BY
LIMIT
```

## Filter Choice

```text
WHERE   → source rows
HAVING  → aggregate groups
QUALIFY → window results
```

## Latest Record Pattern

```sql
SELECT *
FROM source_table
QUALIFY ROW_NUMBER() OVER (
    PARTITION BY business_key
    ORDER BY updated_at DESC, ingestion_id DESC
) = 1;
```

## Rank Functions

```text
ROW_NUMBER → always unique sequence
RANK       → ties with gaps
DENSE_RANK → ties without gaps
```

## Grain Rule

Before joining or aggregating:

```text
What does one row represent?
What should one output row represent?
Is the business key unique?
```

## Join Rule

```text
One-to-many  → left-side values repeat
Many-to-many → matching combinations multiply
```

## Safe Existence Logic

```sql
WHERE EXISTS (...)
WHERE NOT EXISTS (...)
```

## Null-Safe Equality

```sql
left_value IS NOT DISTINCT FROM right_value
```

## Join Explosion Fix

```text
Define output grain
→ aggregate/deduplicate inputs
→ validate uniqueness
→ join
→ validate output counts
```

## Current Next Topic

`MERGE` and production-grade incremental loading.

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Notes Index]]
