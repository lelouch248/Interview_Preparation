---
tags:
  - snowflake/foundations
  - interview-questions
---

# Phase 1 Interview Questions

## 1. Explain Snowflake architecture.

> Snowflake has three major layers: storage, compute, and cloud services. Data is stored centrally in Snowflake-managed columnar storage. Virtual warehouses provide independent compute for queries and data processing. Cloud services manage authentication, metadata, optimization, transactions, and access control. This separation supports independent scaling, workload isolation, and cost control.

## 2. What is a virtual warehouse?

> A virtual warehouse is an independent compute cluster. It provides CPU, memory, and temporary storage for SQL, DML, loading, unloading, and some Snowpark operations. It does not permanently store table data.

## 3. What is the difference between a database and a warehouse?

> A database is a logical container for schemas and data objects. A warehouse is compute used to execute queries and processing. Creating a database does not provide compute.

## 4. What is compute-storage separation?

> Snowflake stores data separately from compute. Multiple independent warehouses can query the same central data without sharing compute resources. This allows workload isolation and independent scaling.

## 5. What are micro-partitions?

> Snowflake automatically organizes table data into micro-partitions. It stores metadata about each micro-partition and uses that metadata to prune irrelevant data during query execution.

## 6. Permanent vs transient vs temporary tables?

> Permanent tables are used for critical long-lived data and include the strongest recovery features. Transient tables persist but do not include Fail-safe, making them suitable for rebuildable staging data. Temporary tables exist only for the creating session.

## 7. How does RBAC work?

> Privileges are granted to roles, and roles are granted to users or other roles. The active role in a session determines what actions the user can perform.

## 8. What privileges are needed to query a table?

> The role typically needs usage on the warehouse, database, and schema, plus select on the table or view.

## 9. What is ownership?

> Ownership is the highest object-level privilege. One role owns each object and can normally manage that object and its grants.

## 10. What is Time Travel?

> Time Travel allows users to query, clone, or restore historical object states within the retention period. It is useful for recovery and investigation.

## 11. What is Fail-safe?

> Fail-safe is Snowflake-managed last-resort recovery after Time Travel ends. It is not intended for normal self-service querying or restoration.

## 12. What is zero-copy cloning?

> Zero-copy cloning creates a fast logical copy of a table, schema, or database without immediately duplicating all unchanged underlying storage.

## 13. Scale up vs scale out?

> Scaling up increases warehouse size for an individual workload. Scaling out adds clusters or separates workloads to support greater concurrency.

## 14. Why use separate warehouses?

> Separate warehouses isolate workloads such as ETL, BI, analytics, and development while allowing all of them to access the same stored data.

## 15. Why use fully qualified names?

> Fully qualified names reduce dependence on session context and prevent accidental queries against the wrong database or schema.

---

# Two-Minute Foundation Answer

> Snowflake separates storage and compute. Data is stored centrally in Snowflake-managed columnar storage and automatically organized into micro-partitions. Virtual warehouses provide independent compute for SQL and data-processing workloads, while cloud services handle authentication, metadata, optimization, access control, and transactions. Objects are organized into databases and schemas, and access is managed through role-based privileges. Snowflake also provides recovery and development features such as Time Travel, Fail-safe, and zero-copy cloning.


---

# SQL Mastery Questions

## Window Functions

1. What is a window function?
2. How does a window function differ from `GROUP BY`?
3. What is the difference between `ROW_NUMBER`, `RANK`, and `DENSE_RANK`?
4. How do you select the latest row for every business key?
5. Why is deterministic ordering important?
6. What is a window frame?
7. What is the difference between `WHERE`, `HAVING`, and `QUALIFY`?
8. How do `LAG` and `LEAD` work?
9. How would you detect a change in a status column?
10. Why can aggregating an event-history table create incorrect totals?

## Advanced Joins

1. What is table grain?
2. What is join cardinality?
3. Why can a join create duplicate-looking rows?
4. What is a join explosion?
5. How do you prevent a many-to-many join from inflating totals?
6. What is the difference between a condition in `ON` and a condition in `WHERE` for a left join?
7. When should `EXISTS` be used instead of a regular join?
8. What is an anti join?
9. Why is `NOT EXISTS` often safer than `NOT IN`?
10. How does Snowflake handle `NULL = NULL`?
11. When should `IS NOT DISTINCT FROM` be used?
12. What is an `ASOF JOIN`?
13. How do you diagnose unexpected row multiplication?
14. Why should `DISTINCT` not be used to blindly fix a join?
15. What should be checked before joining two production datasets?

## Mock Interview Readiness

- [ ] Explain Snowflake architecture in two minutes
- [ ] Explain warehouse sizing and workload isolation
- [ ] Design basic RBAC
- [ ] Recover deleted data with Time Travel
- [ ] Write latest-record logic
- [ ] Debug a many-to-many join
- [ ] Explain `ASOF JOIN`
- [ ] Explain SQL grain and cardinality

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[06_Revision/Snowflake/Snowflake Interview Answers|Interview Answers]]
- [[99_Templates/Snowflake Mock Interview Template|Mock Interview Template]]
