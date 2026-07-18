---
tags:
  - snowflake/foundations
  - virtual-warehouse
  - compute
---

# Virtual Warehouses

## Definition

A virtual warehouse is an independent Snowflake compute cluster.

It provides resources to execute:

- SQL queries
- `INSERT`, `UPDATE`, `DELETE`, and `MERGE`
- Data loading and unloading
- Transformations
- Some Snowpark operations

> [!important]
> Warehouses consume credits while running.

---

# Create a Warehouse

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_SNOWFLAKE_LEARNING
WITH
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE;
```

## Property Explanation

### `WAREHOUSE_SIZE`

Controls the compute size.

Typical sizes:

```text
XSMALL
SMALL
MEDIUM
LARGE
XLARGE
2XLARGE
3XLARGE
...
```

Larger warehouses provide more compute and parallelism but consume more credits per running period.

### `AUTO_SUSPEND`

```sql
AUTO_SUSPEND = 60
```

Suspends the warehouse after 60 seconds without activity.

### `AUTO_RESUME`

```sql
AUTO_RESUME = TRUE
```

Automatically starts the warehouse when a query requires compute.

### `INITIALLY_SUSPENDED`

```sql
INITIALLY_SUSPENDED = TRUE
```

Creates the warehouse without immediately starting it.

---

# Warehouse Commands

```sql
USE WAREHOUSE WH_SNOWFLAKE_LEARNING;

ALTER WAREHOUSE WH_SNOWFLAKE_LEARNING RESUME;
ALTER WAREHOUSE WH_SNOWFLAKE_LEARNING SUSPEND;

SHOW WAREHOUSES LIKE 'WH_SNOWFLAKE_LEARNING';
DESC WAREHOUSE WH_SNOWFLAKE_LEARNING;
```

Resize:

```sql
ALTER WAREHOUSE WH_SNOWFLAKE_LEARNING
SET WAREHOUSE_SIZE = 'SMALL';
```

---

# Scale Up vs Scale Out

## Scale Up

Increase warehouse size.

```sql
ALTER WAREHOUSE WH_ETL
SET WAREHOUSE_SIZE = 'LARGE';
```

Use scale-up when:

- A single query needs more compute
- More memory or parallelism is required
- A large transformation spills or runs slowly

## Scale Out

Use multiple clusters or separate warehouses.

Use scale-out when:

- Many queries are queued
- Concurrency is high
- BI users compete for compute
- Workloads should be isolated

## Interview Rule

```text
Single query is slow       → inspect SQL and consider scaling up
Many queries are queued    → consider scaling out
Different workload types   → consider separate warehouses
```

---

# Workload Isolation

Example architecture:

```text
WH_ETL       → ingestion and transformations
WH_BI        → dashboards
WH_DATA_SCI  → analytical workloads
WH_DEV       → development
```

The same tables can be queried through different warehouses.

Benefits:

- Better reliability
- Predictable performance
- Cost attribution
- Independent scaling
- Reduced workload interference

---

# Cost Awareness

Warehouse cost depends on:

- Warehouse size
- Running duration
- Number of active clusters
- Workload frequency
- Auto-suspend configuration

## Cost-Safe Development Defaults

```sql
WAREHOUSE_SIZE = 'XSMALL'
AUTO_SUSPEND = 60
AUTO_RESUME = TRUE
INITIALLY_SUSPENDED = TRUE
```

---

# Troubleshooting a Slow Query

Do not immediately resize the warehouse.

Check:

1. How much data is scanned?
2. Is partition pruning happening?
3. Is a join multiplying rows?
4. Are unnecessary columns selected?
5. Are filters applied early?
6. Is the warehouse queued?
7. Is data spilling to local or remote storage?
8. Is the warehouse undersized for this operation?

---

# Interview Answers

## What is a virtual warehouse?

> A virtual warehouse is an independent compute cluster that executes queries and data-processing operations. It does not permanently store Snowflake table data.

## Why use separate warehouses?

> Separate warehouses isolate workloads. ETL jobs, BI dashboards, analysts, and development users can use independent compute while querying the same central data.

## Scale up vs scale out

> Scaling up increases the warehouse size to improve resources for individual queries. Scaling out adds clusters or separates workloads to handle greater concurrency.

---

# Review Questions

1. Does suspending a warehouse remove table data?
2. What does auto-resume do?
3. When should a warehouse be resized?
4. What problem does a multi-cluster warehouse solve?
5. Why should development warehouses use auto-suspend?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
