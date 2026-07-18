---
tags:
  - snowflake
  - interview-answers
---

# Snowflake Interview Answers

## Explain Snowflake Architecture

> Snowflake separates storage, compute, and cloud services. Data is stored centrally in Snowflake-managed columnar storage and automatically organized into micro-partitions. Virtual warehouses provide independent compute for queries and processing. Cloud services manage metadata, authentication, optimization, transactions, and access control. This design supports independent scaling, workload isolation, concurrency, and cost control.

## What Is a Virtual Warehouse?

> A virtual warehouse is an independent compute cluster that executes SQL, DML, loading, unloading, and some Snowpark workloads. It does not permanently store table data.

## Scale Up vs Scale Out

> Scaling up increases warehouse size to give an individual workload more CPU, memory, and parallelism. Scaling out adds clusters or separates workloads to handle higher concurrency.

## Permanent vs Transient vs Temporary Tables

> Permanent tables are used for critical long-lived data and provide the strongest recovery features. Transient tables persist but have no Fail-safe, making them appropriate for rebuildable staging data. Temporary tables are visible only in the creating session and are suitable for scratch processing.

## Explain Snowflake RBAC

> Privileges are granted to roles, and roles are granted to users or other roles. The active role in a session determines what operations the user can perform. To query a table, a role normally needs warehouse usage, database and schema usage, and select on the object.

## Time Travel vs Fail-safe

> Time Travel is user-accessible historical recovery during a configured retention period. It supports historical queries, restoration, and point-in-time cloning. Fail-safe is Snowflake-managed last-resort recovery after Time Travel expires and is not a normal self-service feature.

## What Is Zero-Copy Cloning?

> Zero-copy cloning creates a fast logical copy of a database, schema, or table without immediately duplicating all unchanged underlying storage. The clone and source become logically independent as changes occur.

## What Is a Window Function?

> A window function performs an analytical calculation across related rows while preserving the individual rows. The `OVER` clause defines the partition, ordering, and optional frame.

## GROUP BY vs Window Function

> `GROUP BY` collapses input rows into one output row per group. A window function usually preserves the input rows and adds calculations based on related rows.

## WHERE vs HAVING vs QUALIFY

> `WHERE` filters source rows before grouping, `HAVING` filters grouped results, and `QUALIFY` filters results after window functions are evaluated.

## Latest Record per Business Key

> I use `ROW_NUMBER` partitioned by the business key and ordered by the update timestamp descending. I include a deterministic tie-breaker such as an ingestion sequence or event ID and filter with `QUALIFY ROW_NUMBER() = 1`.

## ROW_NUMBER vs RANK vs DENSE_RANK

> `ROW_NUMBER` always assigns a unique sequence and is appropriate when exactly one record must win. `RANK` gives tied rows the same rank and leaves gaps. `DENSE_RANK` gives tied rows the same rank without leaving gaps.

## What Is Table Grain?

> Table grain defines what one row represents. I identify the grain before aggregating or joining because the same business key may appear multiple times at a more detailed grain.

## Why Do Joins Create Duplicate-Looking Rows?

> A join returns every matching combination. If a key is not unique on one side, a one-to-many join repeats rows. If it is not unique on either side, a many-to-many join multiplies combinations.

## How Do You Prevent Inflated Totals After a Join?

> I define the required final grain, aggregate or deduplicate each input to that grain, verify key uniqueness, perform the join, and validate the output row count and measures.

## ON vs WHERE in a LEFT JOIN

> The `ON` clause controls which right-side rows match while preserving all left-side rows. A right-table condition in `WHERE` is applied after the join and can remove unmatched null-padded rows, effectively changing the result to inner-join behavior.

## When Do You Use EXISTS?

> I use `EXISTS` when the requirement is to determine whether at least one matching row exists and I do not need right-side columns. It avoids multiplying left-side rows.

## What Is an Anti Join?

> An anti join returns left-side rows that have no matching right-side row. I normally express it using `NOT EXISTS` because it communicates intent clearly and behaves safely with nullable data.

## What Is a Join Explosion?

> A join explosion occurs when a join outputs far more rows than expected, commonly because of an incomplete condition or a many-to-many relationship. I diagnose it by checking input grain, key uniqueness, row counts after each join, and Snowflake Query Profile.

## What Is ASOF JOIN?

> An `ASOF JOIN` matches each left-side time-series row to one closest qualifying right-side record using an inequality timestamp condition. It is useful for effective prices, exchange rates, mappings, or classifications active at a transaction time.

## Navigation

- [[03_Mock Interviews/Snowflake/Snowflake Question Bank|Snowflake Question Bank]]
- [[06_Revision/Snowflake/Snowflake Quick Revision|Quick Revision]]
