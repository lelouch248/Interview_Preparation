---
tags:
  - snowflake/foundations
  - rbac
  - security
  - grants
---

# RBAC, Grants, and Ownership

## Core Mental Model

```text
Privileges are granted to roles.
Roles are granted to users.
The active role determines what the session can do.
```

Avoid direct grants to individual users when a role-based design can be used.

---

# Main Security Objects

| Object | Purpose |
|---|---|
| User | Human or service identity |
| Role | Collection of privileges |
| Privilege | Permission to perform an action |
| Ownership | Highest object-level control |
| Role hierarchy | Privilege inheritance between roles |

---

# Privileges Commonly Needed to Query a Table

```text
USAGE on warehouse
USAGE on database
USAGE on schema
SELECT on table or view
```

Example:

```sql
GRANT USAGE
ON WAREHOUSE WH_SNOWFLAKE_LEARNING
TO ROLE DEVELOPER_ROLE;

GRANT USAGE
ON DATABASE SNOWFLAKE_MASTERY_DB
TO ROLE DEVELOPER_ROLE;

GRANT USAGE
ON SCHEMA SNOWFLAKE_MASTERY_DB.RAW
TO ROLE DEVELOPER_ROLE;

GRANT SELECT
ON TABLE SNOWFLAKE_MASTERY_DB.RAW.ORDERS
TO ROLE DEVELOPER_ROLE;
```

---

# Create a Role and User

```sql
USE ROLE USERADMIN;

CREATE ROLE IF NOT EXISTS DEVELOPER_ROLE;

CREATE USER KARTHIK_USER
    PASSWORD = 'ReplaceWithSecurePassword'
    DEFAULT_ROLE = DEVELOPER_ROLE
    MUST_CHANGE_PASSWORD = TRUE;

GRANT ROLE DEVELOPER_ROLE
TO USER KARTHIK_USER;
```

> [!warning]
> Use proper secret-management practices in real environments. Do not store real passwords in notes or repositories.

---

# Existing and Future Grants

## Existing Tables

```sql
GRANT SELECT
ON ALL TABLES IN SCHEMA SNOWFLAKE_MASTERY_DB.RAW
TO ROLE DEVELOPER_ROLE;
```

## Future Tables

```sql
GRANT SELECT
ON FUTURE TABLES IN SCHEMA SNOWFLAKE_MASTERY_DB.RAW
TO ROLE DEVELOPER_ROLE;
```

Both may be required:

```text
ALL TABLES    → objects that already exist
FUTURE TABLES → objects created later
```

---

# Common Privileges

| Privilege | Meaning |
|---|---|
| `USAGE` | Use a warehouse, database, or schema |
| `SELECT` | Read table or view data |
| `INSERT` | Add rows |
| `UPDATE` | Modify rows |
| `DELETE` | Remove rows |
| `CREATE TABLE` | Create tables in a schema |
| `MONITOR` | View certain usage or execution information |
| `OWNERSHIP` | Full control over the object |

---

# Ownership

Each securable object is owned by one role.

By default, the role that creates the object owns it.

```sql
USE ROLE SYSADMIN;

CREATE TABLE RAW.TEST_TABLE (
    id NUMBER
);
```

Check ownership and grants:

```sql
SHOW GRANTS ON TABLE RAW.TEST_TABLE;
```

Transfer ownership:

```sql
GRANT OWNERSHIP
ON TABLE RAW.TEST_TABLE
TO ROLE DATA_ENGINEER_ROLE
REVOKE CURRENT GRANTS;
```

> [!warning]
> Ownership transfer can affect existing grants. Review the transfer behavior carefully in production.

---

# Role Hierarchy

Roles can be granted to other roles.

Example:

```text
ACCOUNTADMIN
    ↑
SECURITYADMIN
    ↑
SYSADMIN
    ↑
DATA_ENGINEER_ROLE
    ↑
DEVELOPER_ROLE
```

A higher role can inherit privileges from roles granted to it.

A clean role hierarchy supports:

- Least privilege
- Reusable access patterns
- Easier auditing
- Environment separation
- Team-based access

---

# Least Privilege

Grant only the access required for the role's duties.

Example:

```text
ANALYST_ROLE
- USAGE on BI warehouse
- USAGE on curated database and schema
- SELECT on reporting views
- No table modification privileges
```

```text
ETL_ROLE
- USAGE on ETL warehouse
- Read access to raw data
- Write access to staging and mart schemas
- Task and procedure execution privileges as needed
```

---

# Interview Answer

> Snowflake commonly uses role-based access control. Privileges are granted to roles, and roles are granted to users or other roles. A user activates a role in a session, and that role determines the available actions. To query a table, the role typically needs warehouse usage, database and schema usage, and table select privileges.

---

# Review Questions

1. Why is `SELECT` on a table alone often insufficient?
2. What is the difference between existing-object and future-object grants?
3. What role owns an object by default?
4. Why should direct user grants be avoided?
5. What does least privilege mean?

---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Index]]
- [[01_Topics/Snowflake/Snowflake Mastery Track|Snowflake Mastery Track]]
