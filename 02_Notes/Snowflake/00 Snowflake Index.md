---
tags:
  - snowflake
  - index
  - data-engineering
aliases:
  - Snowflake Notes
  - Snowflake Home
---

# Snowflake Notes Index ❄️

> [!summary]
> Main knowledge index for all detailed Snowflake notes studied so far.

## Foundation Notes

1. [[02_Notes/Snowflake/01_Foundations/01 Snowflake Architecture|Snowflake Architecture]]
2. [[02_Notes/Snowflake/01_Foundations/02 Object Hierarchy and Context|Object Hierarchy and Context]]
3. [[02_Notes/Snowflake/01_Foundations/03 Virtual Warehouses|Virtual Warehouses]]
4. [[02_Notes/Snowflake/01_Foundations/04 Databases Schemas and Tables|Databases, Schemas, and Tables]]
5. [[02_Notes/Snowflake/01_Foundations/05 Table Types|Table Types]]
6. [[02_Notes/Snowflake/01_Foundations/06 RBAC Grants and Ownership|RBAC, Grants, and Ownership]]
7. [[02_Notes/Snowflake/01_Foundations/07 Time Travel Failsafe and Cloning|Time Travel, Fail-safe, and Cloning]]

## SQL Mastery Notes

1. [[02_Notes/Snowflake/02_SQL Mastery/01 Query Processing Window Functions and QUALIFY|Query Processing, Window Functions, and QUALIFY]]
2. [[02_Notes/Snowflake/02_SQL Mastery/02 Advanced Joins|Advanced Joins]]

## Practice and Application

- [[04_Real Life Problems/Snowflake/Snowflake SQL Practice|Snowflake SQL Practice]]
- [[04_Real Life Problems/Snowflake/Join Explosion Investigation|Join Explosion Investigation]]
- [[03_Mock Interviews/Snowflake/Snowflake Question Bank|Snowflake Question Bank]]

## Revision

- [[06_Revision/Snowflake/Snowflake Quick Revision|Quick Revision]]
- [[06_Revision/Snowflake/Snowflake Interview Answers|Interview Answers]]
- [[06_Revision/Snowflake/Snowflake SQL Patterns|Reusable SQL Patterns]]
- [[06_Revision/Snowflake/Snowflake Mistakes Log|Mistakes Log]]

## Core Mental Models

### Snowflake Triangle

```text
Database = where objects live
Warehouse = what runs queries
Role = what gives permission
```

### Grain-First SQL

```text
1. What does one row represent?
2. Which columns identify a row?
3. Is the join key unique?
4. What should one output row represent?
5. Could this operation multiply rows?
```

### Filtering Stages

```text
WHERE   → source rows
HAVING  → grouped rows
QUALIFY → window-function results
```

## Current Position

Completed:

- Foundations
- Window functions and `QUALIFY`
- Advanced joins

Next:

- `MERGE` and production-grade incremental loading

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
