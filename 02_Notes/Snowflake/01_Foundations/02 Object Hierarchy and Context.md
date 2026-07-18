---
tags:
  - snowflake/foundations
  - object-hierarchy
  - session-context
---

# Snowflake Object Hierarchy and Session Context

## Object Hierarchy

```text
Organization
└── Account
    ├── Users
    ├── Roles
    ├── Warehouses
    └── Databases
        └── Schemas
            ├── Tables
            ├── Views
            ├── Stages
            ├── File Formats
            ├── Streams
            ├── Tasks
            ├── Procedures
            └── Functions
```

## Important Objects

| Object | Purpose |
|---|---|
| Organization | Top-level grouping for Snowflake accounts |
| Account | Snowflake environment containing users, roles, warehouses, and databases |
| Warehouse | Compute cluster |
| Database | Logical container for schemas |
| Schema | Namespace and container for database objects |
| Table | Stores data |
| View | Saved query definition |
| Role | Bundle of privileges |
| User | Human or service identity |

---

# Three Things Needed to Query a Table

```text
1. Correct role
2. Active warehouse
3. Correct database/schema/table context
```

A useful mental model:

```text
Database = where objects live
Warehouse = what runs queries
Role = what gives permission
```

---

# Current Session Context

Use these commands to inspect the current session:

```sql
SELECT CURRENT_USER();
SELECT CURRENT_ROLE();
SELECT CURRENT_WAREHOUSE();
SELECT CURRENT_DATABASE();
SELECT CURRENT_SCHEMA();
```

## Set Context

```sql
USE ROLE SYSADMIN;
USE WAREHOUSE WH_SNOWFLAKE_LEARNING;
USE DATABASE SNOWFLAKE_MASTERY_DB;
USE SCHEMA RAW;
```

## Show Objects

```sql
SHOW WAREHOUSES;
SHOW DATABASES;
SHOW SCHEMAS;
SHOW TABLES;
SHOW ROLES;
```

## Describe Objects

```sql
DESC WAREHOUSE WH_SNOWFLAKE_LEARNING;
DESC DATABASE SNOWFLAKE_MASTERY_DB;
DESC SCHEMA RAW;
DESC TABLE ORDERS;
```

---

# Fully Qualified Object Names

A fully qualified table name contains:

```text
DATABASE.SCHEMA.TABLE
```

Example:

```sql
SELECT *
FROM SNOWFLAKE_MASTERY_DB.RAW.ORDERS;
```

This is safer in production code because it reduces dependency on session context.

## Partially Qualified Names

```sql
SELECT * FROM RAW.ORDERS;
```

This depends on the active database.

```sql
SELECT * FROM ORDERS;
```

This depends on both the active database and schema.

---

# Common Context Errors

## No active warehouse

Symptoms:

- Queries requiring compute fail
- Metadata-only commands may still work

Fix:

```sql
USE WAREHOUSE WH_SNOWFLAKE_LEARNING;
```

## Wrong role

Symptoms:

- Object exists but access is denied
- Warehouse cannot be used
- Table cannot be queried

Fix:

```sql
USE ROLE appropriate_role;
```

## Wrong database or schema

Symptoms:

- Object not found
- Wrong table queried
- Development and production objects confused

Fix:

```sql
USE DATABASE SNOWFLAKE_MASTERY_DB;
USE SCHEMA RAW;
```

Or use fully qualified names.

---

# Interview Answer

> Snowflake objects follow a hierarchy: accounts contain databases and warehouses, databases contain schemas, and schemas contain tables, views, stages, tasks, and other objects. A query normally requires an active role with the necessary privileges, an active warehouse for compute, and the correct object context.

---

# Review Questions

1. What is the difference between a warehouse and a database?
2. Where do tables live?
3. What does `CURRENT_ROLE()` return?
4. When should fully qualified object names be used?
5. Why can an object exist but still be inaccessible?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
