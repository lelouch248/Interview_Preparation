---
tags:
  - snowflake/foundations
  - time-travel
  - failsafe
  - cloning
---

# Time Travel, Fail-safe, and Zero-Copy Cloning

## Time Travel

Time Travel allows access to historical object states during the configured retention period.

It supports:

- Querying previous data
- Recovering deleted or changed rows
- Restoring dropped objects
- Creating point-in-time clones
- Investigating data issues

---

# Query Historical Data

Create sample data:

```sql
CREATE OR REPLACE TABLE RAW.TIME_TRAVEL_TEST (
    id NUMBER,
    name STRING
);

INSERT INTO RAW.TIME_TRAVEL_TEST VALUES
    (1, 'Karthik'),
    (2, 'Snowflake');
```

Delete a row:

```sql
DELETE FROM RAW.TIME_TRAVEL_TEST
WHERE id = 2;
```

Query an earlier state using an offset:

```sql
SELECT *
FROM RAW.TIME_TRAVEL_TEST
AT (OFFSET => -300);
```

The offset is expressed in seconds. `-300` means approximately five minutes before the query time.

---

# Statement-Based Time Travel

Retrieve a query ID from history or the current session:

```sql
SELECT LAST_QUERY_ID();
```

Query the table state before a statement:

```sql
SELECT *
FROM RAW.TIME_TRAVEL_TEST
BEFORE (STATEMENT => '<query_id>');
```

This is useful when recovering data changed by a known statement.

---

# Recover Deleted Rows

```sql
INSERT INTO RAW.TIME_TRAVEL_TEST
SELECT *
FROM RAW.TIME_TRAVEL_TEST
AT (OFFSET => -300)
WHERE id = 2;
```

> [!important]
> Use a carefully selected timestamp or statement ID in production. A fixed offset may include unintended changes.

---

# UNDROP

Restore a dropped table:

```sql
DROP TABLE RAW.TIME_TRAVEL_TEST;

UNDROP TABLE RAW.TIME_TRAVEL_TEST;
```

Time Travel also supports recovery operations for certain dropped schemas and databases.

---

# Retention

Retention is controlled through:

```sql
DATA_RETENTION_TIME_IN_DAYS
```

Example:

```sql
CREATE TABLE CRITICAL_ORDERS (
    order_id NUMBER,
    amount NUMBER
)
DATA_RETENTION_TIME_IN_DAYS = 7;
```

Alter retention:

```sql
ALTER TABLE CRITICAL_ORDERS
SET DATA_RETENTION_TIME_IN_DAYS = 7;
```

Retention limits depend on object type and Snowflake edition.

---

# Fail-safe

Fail-safe is Snowflake-managed recovery protection after Time Travel ends.

## Difference from Time Travel

| Feature | Time Travel | Fail-safe |
|---|---|---|
| User-accessible | Yes | No direct self-service |
| Purpose | Routine recovery and historical access | Last-resort disaster recovery |
| Query historical data | Yes | No |
| Clone historical state | Yes | No |
| Normal operational tool | Yes | No |

## Interview Answer

> Time Travel is a user-accessible feature for querying, cloning, and restoring historical data during the retention period. Fail-safe is Snowflake-managed last-resort recovery after Time Travel expires and is not intended for normal user queries.

---

# Zero-Copy Cloning

A clone creates a logical copy of a table, schema, or database without immediately duplicating all underlying data.

## Clone a Table

```sql
CREATE OR REPLACE TABLE RAW.ORDERS_CLONE
CLONE RAW.ORDERS;
```

## Clone a Schema

```sql
CREATE SCHEMA DEV_RAW
CLONE RAW;
```

## Point-in-Time Clone

```sql
CREATE TABLE RAW.ORDERS_BEFORE_CHANGE
CLONE RAW.ORDERS
AT (OFFSET => -3600);
```

This creates a clone representing the table state approximately one hour earlier.

---

# Clone Behaviour

At creation time:

```text
Original and clone initially reference the same unchanged storage.
```

When either object changes:

```text
Snowflake tracks changed storage separately.
```

This enables fast cloning while preserving logical independence.

## Test Clone Independence

```sql
DELETE FROM RAW.ORDERS_CLONE
WHERE status = 'CANCELLED';

SELECT * FROM RAW.ORDERS_CLONE;
SELECT * FROM RAW.ORDERS;
```

Changing the clone does not change the original table.

---

# Real-World Uses

- Development environments
- Testing risky transformations
- Production issue investigation
- Point-in-time debugging
- CI/CD validation datasets
- Creating isolated copies for experiments

> [!warning]
> Cloning is not a complete backup strategy. Retention, object lifecycle, permissions, and downstream dependencies still matter.

---

# Review Questions

1. What can Time Travel do?
2. What is the difference between `AT` and `BEFORE`?
3. Who uses Fail-safe?
4. Does changing a clone change the original?
5. Why is zero-copy cloning useful for development?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
