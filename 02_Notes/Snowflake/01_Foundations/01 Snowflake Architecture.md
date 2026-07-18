---
tags:
  - snowflake/foundations
  - architecture
aliases:
  - Snowflake Architecture
---

# Snowflake Architecture

## Core Mental Model

```text
Snowflake = Storage + Compute + Cloud Services
```

Snowflake separates persistent data storage from the compute resources used to process that data.

## Architecture Layers

### 1. Storage Layer

The storage layer holds table data in Snowflake-managed cloud storage.

Snowflake automatically manages:

- Columnar storage
- Compression
- Physical file organization
- Micro-partitions
- Metadata about stored data
- Statistics used for pruning

Users do not manually manage traditional database files or indexes for standard Snowflake tables.

### 2. Compute Layer

The compute layer consists of **virtual warehouses**.

A virtual warehouse provides:

- CPU
- Memory
- Temporary local storage
- Parallel query execution

Warehouses execute:

- SQL queries
- DML operations
- Data loading and unloading
- Some Snowpark workloads

> [!important]
> A warehouse does not permanently store table data. It is compute only.

### 3. Cloud Services Layer

The cloud services layer coordinates the Snowflake platform.

It handles:

- Authentication
- Authorization
- Query parsing
- Query optimization
- Metadata management
- Transaction coordination
- Query routing
- Result-cache checks
- Account-level services

---

# Query Execution Flow

When a query is submitted:

```text
1. The user submits SQL.
2. Cloud services validates identity and privileges.
3. Snowflake parses and optimizes the query.
4. A virtual warehouse executes the query.
5. The warehouse reads required micro-partitions from storage.
6. The result is returned.
7. Query history and metadata are recorded.
```

Example:

```sql
SELECT
    customer_id,
    SUM(amount)
FROM orders
GROUP BY customer_id;
```

The cloud services layer plans the query, while the warehouse performs the computation.

---

# Compute-Storage Separation

Multiple warehouses can query the same central data using separate compute resources.

Example:

```text
WH_ETL       → pipeline workloads
WH_BI        → dashboards
WH_ADHOC     → analyst queries
WH_DEV       → development work
```

All four warehouses can access the same tables, but their compute resources remain isolated.

## Benefits

- Independent scaling
- Workload isolation
- Better concurrency
- Flexible cost control
- Separate compute for teams and applications

## Interview Answer

> Snowflake has three primary layers: storage, compute, and cloud services. Data is stored centrally in Snowflake-managed columnar storage. Virtual warehouses provide independent compute for queries and data processing. The cloud services layer manages metadata, authentication, optimization, transactions, and access control. This separation enables independent scaling, workload isolation, and cost control.

---

# Micro-Partitions

Snowflake automatically divides table data into immutable storage units called **micro-partitions**.

Snowflake stores metadata for micro-partitions, such as:

- Minimum and maximum column values
- Distinct-value information
- Row counts
- Storage location
- Other optimization statistics

## Partition Pruning

Suppose a table contains several years of order data:

```sql
SELECT *
FROM orders
WHERE order_date = '2026-06-01';
```

If Snowflake knows that certain micro-partitions contain only earlier dates, it can skip those partitions.

This is called **micro-partition pruning**.

## Key Interview Point

> Snowflake automatically organizes data into micro-partitions and uses metadata to avoid scanning irrelevant partitions. Users normally do not create partitions manually.

---

# Traditional Database vs Snowflake

| Area | Traditional database | Snowflake |
|---|---|---|
| Storage and compute | Often tightly coupled | Separated |
| Scaling | Hardware/server-oriented | Warehouse resizing |
| Workload isolation | Can require separate systems | Separate warehouses |
| Partition management | Often manual | Automatic micro-partitions |
| Infrastructure | User-managed or partially managed | Snowflake-managed |
| Concurrency | Shared system pressure | Independent warehouses |

---

# Common Misunderstandings

## “The warehouse stores my data”

Incorrect.

```text
Warehouse = compute
Storage layer = persistent data
```

## “A bigger warehouse always fixes a slow query”

Incorrect.

A query may be slow because of:

- Excessive scanning
- Poor pruning
- Join explosion
- Bad filtering
- Poor data grain
- Excessive concurrency
- Inefficient SQL

Warehouse size is only one factor.

---

# Review Questions

1. What are Snowflake's three architecture layers?
2. What does the cloud services layer do?
3. Where does table data live?
4. What is a virtual warehouse?
5. Why can two teams query the same table without sharing compute?
6. What is partition pruning?
7. Why does Snowflake not normally require manually created partitions?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
