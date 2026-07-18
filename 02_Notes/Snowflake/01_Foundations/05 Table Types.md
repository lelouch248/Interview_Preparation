---
tags:
  - snowflake/foundations
  - table-types
---

# Permanent, Transient, and Temporary Tables

Snowflake supports three common table types:

```text
Permanent
Transient
Temporary
```

The correct choice depends on persistence, recovery requirements, and whether the data can be rebuilt.

---

# Permanent Tables

Permanent tables are the default table type.

```sql
CREATE TABLE CUSTOMER_DIM (
    customer_id NUMBER,
    customer_name STRING
);
```

Use permanent tables for:

- Critical production data
- Fact tables
- Dimension tables
- Long-lived curated datasets
- Data requiring stronger recovery protection

## Characteristics

- Persist across sessions
- Visible to authorized sessions
- Support Time Travel
- Include Fail-safe protection

---

# Transient Tables

Transient tables persist across sessions but do not include Fail-safe.

```sql
CREATE TRANSIENT TABLE STAGING.STG_ORDERS (
    order_id NUMBER,
    customer_id NUMBER,
    order_date DATE,
    status STRING,
    amount NUMBER(10, 2)
);
```

Use transient tables for:

- Rebuildable staging tables
- Intermediate ETL outputs
- Data replicated from a recoverable source
- High-churn data that does not require Fail-safe

## Characteristics

- Persist across sessions
- Visible to authorized sessions
- Limited Time Travel retention
- No Fail-safe

---

# Temporary Tables

Temporary tables exist only inside the session that created them.

```sql
CREATE TEMPORARY TABLE TEMP_HIGH_VALUE_ORDERS AS
SELECT *
FROM RAW.ORDERS
WHERE amount >= 500;
```

Use temporary tables for:

- Scratch work
- Intermediate calculations
- Session-specific transformations
- Testing
- One-time analysis

## Characteristics

- Session-scoped
- Not visible to other sessions
- Removed when the session ends
- No Fail-safe

---

# Comparison

| Feature | Permanent | Transient | Temporary |
|---|---:|---:|---:|
| Persists after session | Yes | Yes | No |
| Visible to other sessions | Yes | Yes | No |
| Time Travel | Yes | Limited | Limited/session-scoped |
| Fail-safe | Yes | No | No |
| Best use | Critical data | Rebuildable data | Scratch processing |

---

# Decision Guide

Use a **permanent table** when:

```text
Losing the data would create meaningful business risk.
```

Use a **transient table** when:

```text
The data must persist but can be rebuilt from another source.
```

Use a **temporary table** when:

```text
The data is needed only during the current session.
```

---

# Common Mistakes

## Using transient tables for irreplaceable data

No Fail-safe means reduced recovery protection.

## Leaving temporary tables open in long sessions

Temporary tables can continue consuming storage while the session remains active.

## Assuming temporary means shared staging

Temporary tables are not visible to other sessions.

---

# Interview Answer

> Permanent tables are used for critical long-lived data and include the strongest recovery features. Transient tables persist but have no Fail-safe, so they are useful for rebuildable staging or intermediate data. Temporary tables are session-scoped and suitable for scratch or intermediate processing.

---

# Review Questions

1. Which table type should hold a production fact table?
2. Which table type is suitable for rebuildable staging data?
3. Can another session query your temporary table?
4. Which table types do not include Fail-safe?
5. Why is a temporary table not a replacement for a shared staging table?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
