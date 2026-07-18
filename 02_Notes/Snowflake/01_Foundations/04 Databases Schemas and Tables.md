---
tags:
  - snowflake/foundations
  - database
  - schema
  - table
---

# Databases, Schemas, and Tables

## Database

A database is a logical container for schemas.

```sql
CREATE DATABASE IF NOT EXISTS SNOWFLAKE_MASTERY_DB;
USE DATABASE SNOWFLAKE_MASTERY_DB;
```

## Schema

A schema organizes related objects inside a database.

```sql
CREATE SCHEMA IF NOT EXISTS RAW;
CREATE SCHEMA IF NOT EXISTS STAGING;
CREATE SCHEMA IF NOT EXISTS MART;
CREATE SCHEMA IF NOT EXISTS AUDIT;
```

Recommended data-engineering structure:

```text
RAW      → source-aligned data
STAGING  → cleaned and standardized data
MART     → business-ready data
AUDIT    → validation, logging, and pipeline metadata
```

## Table

A table stores rows and columns.

```sql
CREATE OR REPLACE TABLE RAW.ORDERS (
    order_id NUMBER,
    customer_id NUMBER,
    order_date DATE,
    status STRING,
    amount NUMBER(10, 2),
    loaded_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);
```

Insert data:

```sql
INSERT INTO RAW.ORDERS
    (order_id, customer_id, order_date, status, amount)
VALUES
    (1, 101, '2026-06-01', 'PLACED', 1200.50),
    (2, 102, '2026-06-02', 'SHIPPED', 800.00),
    (3, 103, '2026-06-03', 'CANCELLED', 300.00);
```

Query:

```sql
SELECT *
FROM RAW.ORDERS;
```

---

# Object Naming

A fully qualified table name:

```text
DATABASE.SCHEMA.TABLE
```

Example:

```sql
SELECT *
FROM SNOWFLAKE_MASTERY_DB.RAW.ORDERS;
```

## Naming Recommendations

Use names that communicate:

- Business meaning
- Data layer
- Object type when helpful
- Environment where required

Examples:

```text
RAW.SALESFORCE_ACCOUNTS
STAGING.STG_ORDERS
MART.FACT_SALES
MART.DIM_CUSTOMER
AUDIT.PIPELINE_RUN_LOG
```

Avoid unclear names such as:

```text
TEMP1
FINAL_TABLE_NEW
TEST_DATA_2
ABC
```

---

# Data Grain

The grain states what one row represents.

Examples:

| Table | Grain |
|---|---|
| `RAW.ORDERS` | One order |
| `RAW.ORDER_EVENTS` | One order event |
| `RAW.ORDER_ITEMS` | One item within an order |
| `MART.CUSTOMER_DAILY_SALES` | One customer per date |

> [!warning]
> SQL can be syntactically correct and still produce wrong results when the grain is misunderstood.

Before creating or querying a table, ask:

```text
What does one row represent?
Which columns uniquely identify a row?
Can multiple rows exist for the same business key?
```

---

# Common Data Layers

## Raw Layer

Characteristics:

- Minimal transformation
- Source-aligned structure
- Preserves original values
- Supports replay and auditing

## Staging Layer

Characteristics:

- Standardized data types
- Clean column names
- Deduplication
- Basic validation
- Intermediate transformation logic

## Mart Layer

Characteristics:

- Business-oriented models
- Fact and dimension tables
- Aggregated or curated data
- Optimized for reporting and consumption

## Audit Layer

Characteristics:

- Load counts
- Validation results
- Error records
- Pipeline status
- Watermarks
- Reconciliation metrics

---

# Review Questions

1. What is the role of a schema?
2. Why separate raw, staging, mart, and audit objects?
3. What is table grain?
4. Why are fully qualified table names useful?
5. What should one row represent in an event-history table?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
