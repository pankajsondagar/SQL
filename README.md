# SQL Queries

A collection of commonly used SQL interview questions and practical database queries.

---

## 1. Find Duplicate Records in a Table

### Scenario
Suppose you have a table named `employees`.

| id | name | email | department |
|----|------|-------|------------|
| 1 | John | john@example.com | IT |
| 2 | Jane | jane@example.com | HR |
| 3 | John | john@example.com | IT |
| 4 | Mike | mike@example.com | Sales |

We want to find duplicate records based on the `email` column.

### Query

```sql
SELECT
    email,
    COUNT(*) AS total_records
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;
```

### Output

| email | total_records |
|-------|---------------|
| john@example.com | 2 |

### Explanation

- `GROUP BY` groups rows having the same email.
- `COUNT(*)` counts how many records exist for each email.
- `HAVING COUNT(*) > 1` filters only duplicate values.

---

## Find Complete Duplicate Rows

If you want to find rows where all selected columns are duplicated:

```sql
SELECT
    name,
    email,
    department,
    COUNT(*) AS total_records
FROM employees
GROUP BY
    name,
    email,
    department
HAVING COUNT(*) > 1;
```

---

## Retrieve Complete Duplicate Records

To display the full duplicate rows:

```sql
SELECT *
FROM employees
WHERE email IN (
    SELECT email
    FROM employees
    GROUP BY email
    HAVING COUNT(*) > 1
)
ORDER BY email;
```

### Output

| id | name | email | department |
|----|------|-------|------------|
| 1 | John | john@example.com | IT |
| 3 | John | john@example.com | IT |

---

## MySQL 8+ (Using Window Function)

```sql
SELECT *
FROM (
    SELECT *,
           COUNT(*) OVER (PARTITION BY email) AS duplicate_count
    FROM employees
) t
WHERE duplicate_count > 1;
```

---

## Notes

- Replace `email` with any column(s) you want to check for duplicates.
- Use multiple columns in `GROUP BY` if duplicates are determined by more than one field.
- `HAVING` is used to filter grouped data, whereas `WHERE` filters individual rows before grouping.
