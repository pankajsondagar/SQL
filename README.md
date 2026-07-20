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

---

# 2. Retrieve the Second Highest Salary from the Employee Table

## Scenario

Suppose you have an `employees` table.

| id | name | salary |
|----|------|--------|
| 1 | John | 50000 |
| 2 | Jane | 75000 |
| 3 | Mike | 90000 |
| 4 | David | 75000 |
| 5 | Alice | 60000 |

We want to retrieve the **second highest salary** from the table.

---

## Method 1: Using `MAX()` (Works in Most SQL Databases)

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

### Output

| second_highest_salary |
|------------------------|
| 75000 |

### Explanation

- The inner query finds the highest salary.
- The outer query finds the maximum salary that is less than the highest salary.
- This method correctly handles duplicate highest salaries.

---

## Method 2: Using `ORDER BY` with `LIMIT` (MySQL)

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

### Explanation

- `DISTINCT` removes duplicate salaries.
- `ORDER BY salary DESC` sorts salaries from highest to lowest.
- `OFFSET 1` skips the highest salary.
- `LIMIT 1` returns the second highest salary.

---

## Method 3: Using `DENSE_RANK()` (MySQL 8+, PostgreSQL, SQL Server, Oracle)

```sql
SELECT salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
) AS ranked_salary
WHERE salary_rank = 2;
```

### Output

| salary |
|--------|
| 75000 |

### Explanation

- `DENSE_RANK()` assigns rankings based on distinct salary values.
- Employees with the same salary receive the same rank.
- Filtering for `salary_rank = 2` returns the second highest salary.

---

## Method 4: Retrieve Employee(s) with the Second Highest Salary

```sql
SELECT *
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
    WHERE salary < (
        SELECT MAX(salary)
        FROM employees
    )
);
```

### Output

| id | name | salary |
|----|------|--------|
| 2 | Jane | 75000 |
| 4 | David | 75000 |

---

## Notes

- Use `DISTINCT` when duplicate salary values exist.
- `DENSE_RANK()` is the preferred approach for modern SQL databases.
- The `MAX()` method is compatible with almost all SQL databases.
- If there is no second highest salary (all employees have the same salary), these queries return `NULL` or no rows depending on the method.
