---
tags:
  - snowflake/foundations
  - labs
---

# Phase 1 Hands-On Labs

Complete these labs in a Snowflake worksheet.

---

# Lab 1 — Inspect Context

```sql
SELECT CURRENT_USER();
SELECT CURRENT_ROLE();
SELECT CURRENT_WAREHOUSE();
SELECT CURRENT_DATABASE();
SELECT CURRENT_SCHEMA();

SHOW WAREHOUSES;
SHOW DATABASES;
SHOW ROLES;
```

Questions:

- What is the active role?
- Which warehouse is selected?
- Which database and schema are active?
- Which commands work without a running warehouse?

---

# Lab 2 — Create Learning Objects

```sql
USE ROLE SYSADMIN;

CREATE DATABASE IF NOT EXISTS SNOWFLAKE_MASTERY_DB;

CREATE SCHEMA IF NOT EXISTS SNOWFLAKE_MASTERY_DB.RAW;
CREATE SCHEMA IF NOT EXISTS SNOWFLAKE_MASTERY_DB.STAGING;
CREATE SCHEMA IF NOT EXISTS SNOWFLAKE_MASTERY_DB.MART;
CREATE SCHEMA IF NOT EXISTS SNOWFLAKE_MASTERY_DB.AUDIT;
```

---

# Lab 3 — Create the Learning Warehouse

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_SNOWFLAKE_LEARNING
WITH
    WAREHOUSE_SIZE = 'XSMALL'
    AUTO_SUSPEND = 60
    AUTO_RESUME = TRUE
    INITIALLY_SUSPENDED = TRUE;

USE WAREHOUSE WH_SNOWFLAKE_LEARNING;
```

Inspect:

```sql
SHOW WAREHOUSES LIKE 'WH_SNOWFLAKE_LEARNING';
DESC WAREHOUSE WH_SNOWFLAKE_LEARNING;
```

---

# Lab 4 — Create and Query a Permanent Table

```sql
USE DATABASE SNOWFLAKE_MASTERY_DB;
USE SCHEMA RAW;

CREATE OR REPLACE TABLE ORDERS (
    order_id NUMBER,
    customer_id NUMBER,
    order_date DATE,
    status STRING,
    amount NUMBER(10, 2),
    loaded_at TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);

INSERT INTO ORDERS
    (order_id, customer_id, order_date, status, amount)
VALUES
    (1, 101, '2026-06-01', 'PLACED', 1200.50),
    (2, 102, '2026-06-02', 'SHIPPED', 800.00),
    (3, 103, '2026-06-03', 'PLACED', 500.00),
    (4, 101, '2026-06-04', 'CANCELLED', 200.00);

SELECT *
FROM ORDERS;
```

---

# Lab 5 — Temporary Table

```sql
CREATE TEMPORARY TABLE TEMP_HIGH_VALUE_ORDERS AS
SELECT *
FROM ORDERS
WHERE amount >= 500;

SELECT *
FROM TEMP_HIGH_VALUE_ORDERS;
```

Test:

1. Open another session.
2. Attempt to query the temporary table.
3. Explain the result.

---

# Lab 6 — Transient Table

```sql
CREATE OR REPLACE TRANSIENT TABLE STAGING.STG_ORDERS AS
SELECT
    order_id,
    customer_id,
    order_date,
    UPPER(status) AS status,
    amount,
    loaded_at
FROM RAW.ORDERS;

SELECT *
FROM STAGING.STG_ORDERS;
```

Question:

> Why is this staging table a reasonable transient-table candidate?

---

# Lab 7 — Time Travel

```sql
CREATE OR REPLACE TABLE RAW.TIME_TRAVEL_TEST AS
SELECT *
FROM RAW.ORDERS;

DELETE FROM RAW.TIME_TRAVEL_TEST
WHERE order_id = 2;
```

Inspect current state:

```sql
SELECT *
FROM RAW.TIME_TRAVEL_TEST;
```

Inspect an earlier state:

```sql
SELECT *
FROM RAW.TIME_TRAVEL_TEST
AT (OFFSET => -60);
```

Use a timestamp or statement ID when necessary if the offset does not capture the intended state.

---

# Lab 8 — UNDROP

```sql
DROP TABLE RAW.TIME_TRAVEL_TEST;

UNDROP TABLE RAW.TIME_TRAVEL_TEST;

SELECT *
FROM RAW.TIME_TRAVEL_TEST;
```

---

# Lab 9 — Zero-Copy Clone

```sql
CREATE OR REPLACE TABLE RAW.ORDERS_CLONE
CLONE RAW.ORDERS;

DELETE FROM RAW.ORDERS_CLONE
WHERE status = 'CANCELLED';

SELECT * FROM RAW.ORDERS_CLONE;
SELECT * FROM RAW.ORDERS;
```

Explain why the original remains unchanged.

---

# Lab 10 — Access-Control Design

Design roles for:

```text
ANALYST_ROLE
- Read curated reporting data
- Use BI warehouse

ETL_ROLE
- Read raw data
- Write staging and mart data
- Use ETL warehouse

DATA_ADMIN_ROLE
- Manage databases, schemas, and objects
```

Write the required grants without directly granting privileges to named users.

---

# Completion Checklist

- [ ] Created warehouse
- [ ] Created database and schemas
- [ ] Created permanent table
- [ ] Created transient table
- [ ] Created temporary table
- [ ] Tested Time Travel
- [ ] Tested `UNDROP`
- [ ] Created clone
- [ ] Verified clone independence
- [ ] Designed an RBAC structure


---

---
tags:
  - snowflake/sql
  - exercises
---

# SQL Practice Exercises

Complete these without immediately copying the examples from the topic pages.

---

# Window Functions and QUALIFY

## Exercise 1

Return the latest event for every order.

Expected grain:

```text
One row per order
```

## Exercise 2

Return only orders whose latest status is `SHIPPED`.

## Exercise 3

Return the first event for every order.

## Exercise 4

Show each event with its previous status.

## Exercise 5

Show each event with its next event timestamp.

## Exercise 6

Calculate the number of hours between the current event and the previous event.

Hint:

```sql
DATEDIFF('hour', previous_timestamp, event_timestamp)
```

## Exercise 7

Find the two highest-value current orders for each customer.

## Exercise 8

Create a current-order summary:

```text
customer_id
active_order_count
total_active_order_value
latest_order_timestamp
```

Exclude orders whose latest status is `CANCELLED`.

## Exercise 9

Detect actual status changes using `LAG`.

## Exercise 10

Explain why this is incorrect:

```sql
SELECT
    customer_id,
    SUM(amount)
FROM ORDER_EVENTS
GROUP BY customer_id;
```

---

# Advanced Joins

## Exercise 11

Return every customer with their orders, including customers without orders.

## Exercise 12

Return all orders, including orders whose customer is missing from the customer master.

## Exercise 13

Return customers who have at least one completed order, with exactly one row per customer.

## Exercise 14

Return customers who have never placed an order.

## Exercise 15

Return orders that have no payment records.

## Exercise 16

Return the latest payment attempt for each order.

## Exercise 17

Return only orders whose latest payment status is `SUCCESS`.

## Exercise 18

Create exactly one row per order containing:

```text
order_id
customer_name
order_amount
successful_paid_amount
total_item_amount
payment_difference
item_difference
```

Suggested calculations:

```sql
order_amount - successful_paid_amount
order_amount - total_item_amount
```

## Exercise 19

Use a full outer join to classify customer IDs as:

```text
CUSTOMER_MASTER_ONLY
ORDERS_ONLY
BOTH
```

Customer `101` must not repeat merely because multiple orders exist.

## Exercise 20

Write a self join that returns:

```text
customer_name
referrer_name
```

## Exercise 21

Assign each order to a discount band using a range join.

## Exercise 22

Use `ASOF JOIN` to determine the applicable price for each sales event.

## Exercise 23

Run:

```sql
SELECT *
FROM PAYMENTS p
JOIN ORDER_ITEMS i
    ON p.order_id = i.order_id;
```

Record:

```text
Payment input rows
Item input rows
Joined output rows
Distinct order IDs
```

Explain the multiplication.

---

# Verbal Mastery Questions

Answer aloud without reading the notes.

1. What is table grain?
2. Why can a join multiply rows?
3. What is a deterministic tie-breaker?
4. Why use `QUALIFY`?
5. `ROW_NUMBER` vs `RANK`?
6. `WHERE` vs `HAVING` vs `QUALIFY`?
7. When should `EXISTS` be used?
8. Why is `NOT EXISTS` safer than `NOT IN`?
9. Why can a right-table `WHERE` filter break a left join?
10. What is an `ASOF JOIN`?
11. Why should raw payments and raw order items not be directly aggregated after joining?
12. How do you diagnose a join explosion?

---

# Self-Assessment

Rate each area from 1 to 5.

| Area | Score |
|---|---:|
| Architecture |  |
| Warehouses |  |
| Object hierarchy |  |
| Table types |  |
| RBAC |  |
| Time Travel and cloning |  |
| Window functions |  |
| `QUALIFY` |  |
| Grain awareness |  |
| Advanced joins |  |
| Interview explanation |  |

## Ready to Move Forward When

- [ ] You can write latest-record logic without help
- [ ] You can explain join multiplication before running the query
- [ ] You can align grains before joining
- [ ] You can use semi and anti joins
- [ ] You can explain Snowflake foundations in two minutes
- [ ] You can complete at least 80% of the exercises independently


---

## Navigation

- [[00_Dashboard/Snowflake Dashboard|Snowflake Dashboard]]
- [[02_Notes/Snowflake/00 Snowflake Index|Snowflake Notes Index]]
- [[06_Revision/Snowflake/Snowflake Mistakes Log|Snowflake Mistakes Log]]
