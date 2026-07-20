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

---

# 3. Find Employees Without Department

## Scenario

Suppose you have an `employees` table.

| id | name | department_id |
|----|------|---------------|
| 1 | John | 101 |
| 2 | Jane | NULL |
| 3 | Mike | 102 |
| 4 | David | NULL |
| 5 | Alice | 103 |

We want to find employees who are **not assigned to any department**.

---

## Method 1: Using `IS NULL` (Most Common)

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

### Output

| id | name | department_id |
|----|------|---------------|
| 2 | Jane | NULL |
| 4 | David | NULL |

### Explanation

- `NULL` represents missing or unknown data.
- `IS NULL` is used to check for `NULL` values.
- Do **not** use `= NULL`; it will never return any rows.

---

## Method 2: Employees Without a Matching Department (Using `LEFT JOIN`)

Suppose you have a `departments` table.

### departments

| department_id | department_name |
|---------------|-----------------|
| 101 | IT |
| 102 | HR |
| 103 | Finance |

To find employees whose `department_id` does not exist in the `departments` table:

```sql
SELECT e.*
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

### Explanation

- `LEFT JOIN` returns all employees.
- If no matching department exists, the department columns become `NULL`.
- Filtering with `WHERE d.department_id IS NULL` returns employees without a valid department.

---

## Method 3: Using `NOT EXISTS`

```sql
SELECT *
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
);
```

### Explanation

- `NOT EXISTS` checks whether a matching department exists.
- If no matching department is found, the employee is returned.
- This method is efficient and commonly used in SQL interviews.

---

## Notes

- Use `IS NULL` when the department column itself is `NULL`.
- Use `LEFT JOIN` or `NOT EXISTS` when checking against a separate `departments` table.
- Avoid using `= NULL`; always use `IS NULL` or `IS NOT NULL`.

---

# 4. Calculate the Total Revenue Per Product

## Scenario

Suppose you have an `orders` table.

| order_id | product_name | quantity | price |
|----------|--------------|---------:|------:|
| 1 | Laptop | 2 | 50000 |
| 2 | Mouse | 5 | 500 |
| 3 | Laptop | 1 | 50000 |
| 4 | Keyboard | 3 | 1500 |
| 5 | Mouse | 2 | 500 |

We want to calculate the **total revenue generated by each product**.

> **Revenue = Quantity × Price**

---

## Method 1: Using `SUM()`

```sql
SELECT
    product_name,
    SUM(quantity * price) AS total_revenue
FROM orders
GROUP BY product_name;
```

### Output

| product_name | total_revenue |
|--------------|--------------:|
| Keyboard | 4500 |
| Laptop | 150000 |
| Mouse | 3500 |

### Explanation

- `quantity * price` calculates the revenue for each order.
- `SUM()` adds the revenue of all orders for the same product.
- `GROUP BY product_name` groups the records by product.

---

## Method 2: Include Total Quantity Sold

```sql
SELECT
    product_name,
    SUM(quantity) AS total_quantity_sold,
    SUM(quantity * price) AS total_revenue
FROM orders
GROUP BY product_name;
```

### Output

| product_name | total_quantity_sold | total_revenue |
|--------------|--------------------:|--------------:|
| Keyboard | 3 | 4500 |
| Laptop | 3 | 150000 |
| Mouse | 7 | 3500 |

---

## Method 3: Sort by Highest Revenue

```sql
SELECT
    product_name,
    SUM(quantity * price) AS total_revenue
FROM orders
GROUP BY product_name
ORDER BY total_revenue DESC;
```

### Explanation

- `ORDER BY total_revenue DESC` displays the products from highest to lowest revenue.

---

## Method 4: Revenue by Product ID

If your table stores product IDs instead of product names:

```sql
SELECT
    product_id,
    SUM(quantity * price) AS total_revenue
FROM orders
GROUP BY product_id;
```

---

## Notes

- Revenue is calculated as **Quantity × Price**.
- `SUM()` aggregates the revenue across multiple orders.
- Use `GROUP BY` to calculate revenue for each product.
- `ORDER BY` can be used to identify the highest revenue-generating products.
- This is one of the most common SQL aggregation interview questions.

---

# 5. Get the Top 3 Highest Paid Employees

## Scenario

Suppose you have an `employees` table.

| id | name | salary |
|----|------|--------:|
| 1 | John | 50000 |
| 2 | Jane | 90000 |
| 3 | Mike | 75000 |
| 4 | David | 85000 |
| 5 | Alice | 95000 |
| 6 | Bob | 90000 |

We want to retrieve the **top 3 highest-paid employees**.

---

## Method 1: Using `ORDER BY` and `LIMIT` (MySQL, PostgreSQL)

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

### Output

| id | name | salary |
|----|------|--------:|
| 5 | Alice | 95000 |
| 2 | Jane | 90000 |
| 6 | Bob | 90000 |

### Explanation

- `ORDER BY salary DESC` sorts employees from highest to lowest salary.
- `LIMIT 3` returns the first three records.

---

## Method 2: Using `TOP` (SQL Server)

```sql
SELECT TOP 3 *
FROM employees
ORDER BY salary DESC;
```

### Explanation

- `TOP 3` retrieves the first three rows after sorting.

---

## Method 3: Using `FETCH FIRST` (Oracle 12c+, PostgreSQL)

```sql
SELECT *
FROM employees
ORDER BY salary DESC
FETCH FIRST 3 ROWS ONLY;
```

---

## Method 4: Top 3 Distinct Highest Salaries

If you need the **top 3 unique salary values**:

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

### Output

| salary |
|--------:|
| 95000 |
| 90000 |
| 85000 |

---

## Method 5: Using `DENSE_RANK()` (Recommended)

To retrieve **all employees whose salary falls within the top 3 distinct salary ranks**:

```sql
SELECT id, name, salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
) AS ranked_employees
WHERE salary_rank <= 3;
```

### Output

| id | name | salary |
|----|------|--------:|
| 5 | Alice | 95000 |
| 2 | Jane | 90000 |
| 6 | Bob | 90000 |
| 4 | David | 85000 |

### Explanation

- `DENSE_RANK()` assigns the same rank to employees with the same salary.
- `salary_rank <= 3` returns employees whose salaries are among the top three **distinct** salary values.
- This is the preferred solution for SQL interviews because it correctly handles salary ties.

---

## Notes

- Use `LIMIT` in **MySQL** and **PostgreSQL**.
- Use `TOP` in **SQL Server**.
- Use `FETCH FIRST` in **Oracle 12c+**.
- Use `DENSE_RANK()` when duplicate salaries should share the same rank.
- `LIMIT 3` returns exactly three rows, while `DENSE_RANK()` may return more than three employees if there are salary ties.

---

# 6. Customers Who Made Purchases but Never Returned Products

## Scenario

Suppose you have the following tables.

### customers

| customer_id | customer_name |
|------------:|---------------|
| 1 | John |
| 2 | Jane |
| 3 | Mike |
| 4 | Alice |

### orders

| order_id | customer_id | order_date |
|---------:|------------:|------------|
| 101 | 1 | 2025-01-10 |
| 102 | 2 | 2025-01-12 |
| 103 | 3 | 2025-01-15 |
| 104 | 1 | 2025-02-05 |

### returns

| return_id | order_id | customer_id | return_date |
|----------:|----------:|------------:|------------|
| 1 | 102 | 2 | 2025-01-20 |
| 2 | 103 | 3 | 2025-01-25 |

We want to find customers who have made **at least one purchase** but **never returned any product**.

---

## Method 1: Using `NOT EXISTS` (Recommended)

```sql
SELECT c.customer_id,
       c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
)
AND NOT EXISTS (
    SELECT 1
    FROM returns r
    WHERE r.customer_id = c.customer_id
);
```

### Output

| customer_id | customer_name |
|------------:|---------------|
| 1 | John |

### Explanation

- `EXISTS` ensures the customer has placed at least one order.
- `NOT EXISTS` ensures the customer has never made a return.
- This is the preferred approach because it is efficient and handles large datasets well.

---

## Method 2: Using `LEFT JOIN`

```sql
SELECT DISTINCT
    c.customer_id,
    c.customer_name
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
LEFT JOIN returns r
    ON c.customer_id = r.customer_id
WHERE r.customer_id IS NULL;
```

### Explanation

- `INNER JOIN` selects customers who have made purchases.
- `LEFT JOIN` includes return records if they exist.
- `WHERE r.customer_id IS NULL` filters customers who have never returned a product.

---

## Method 3: Using `NOT IN`

```sql
SELECT DISTINCT
    c.customer_id,
    c.customer_name
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
WHERE c.customer_id NOT IN (
    SELECT customer_id
    FROM returns
);
```

### Explanation

- Retrieves customers with purchases.
- Excludes customers found in the `returns` table.
- Use this method only when `customer_id` in the `returns` table cannot contain `NULL` values.

---

## Notes

- `NOT EXISTS` is generally the most reliable and interview-preferred solution.
- `LEFT JOIN ... IS NULL` is another common and readable approach.
- Avoid `NOT IN` if the subquery may return `NULL` values, as it can produce unexpected results.
- Ensure the customer has at least one purchase before checking for returns.

---

# 7. Show the Count of Orders Per Customer

## Scenario

Suppose you have the following tables.

### customers

| customer_id | customer_name |
|------------:|---------------|
| 1 | John |
| 2 | Jane |
| 3 | Mike |
| 4 | Alice |

### orders

| order_id | customer_id | order_date |
|---------:|------------:|------------|
| 101 | 1 | 2025-01-10 |
| 102 | 2 | 2025-01-12 |
| 103 | 1 | 2025-01-18 |
| 104 | 3 | 2025-01-22 |
| 105 | 1 | 2025-02-05 |
| 106 | 2 | 2025-02-10 |

We want to calculate the **number of orders placed by each customer**.

---

## Method 1: Using `GROUP BY` (Most Common)

```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id;
```

### Output

| customer_id | total_orders |
|------------:|-------------:|
| 1 | 3 |
| 2 | 2 |
| 3 | 1 |

### Explanation

- `COUNT(*)` counts the number of orders.
- `GROUP BY customer_id` groups all orders for each customer.
- Customers with no orders are not included.

---

## Method 2: Display Customer Name with Order Count

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name;
```

### Output

| customer_id | customer_name | total_orders |
|------------:|---------------|-------------:|
| 1 | John | 3 |
| 2 | Jane | 2 |
| 3 | Mike | 1 |

### Explanation

- Joins the `customers` and `orders` tables.
- Groups the data by customer.
- Displays both the customer name and the total number of orders.

---

## Method 3: Include Customers with Zero Orders

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name
ORDER BY total_orders DESC;
```

### Output

| customer_id | customer_name | total_orders |
|------------:|---------------|-------------:|
| 1 | John | 3 |
| 2 | Jane | 2 |
| 3 | Mike | 1 |
| 4 | Alice | 0 |

### Explanation

- `LEFT JOIN` returns all customers.
- `COUNT(o.order_id)` counts only matching orders.
- Customers with no orders appear with a count of `0`.

---

## Method 4: Count Orders Placed in the Current Year

```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders
FROM orders
WHERE YEAR(order_date) = YEAR(CURDATE())
GROUP BY customer_id;
```

> **Note:** The above query is for **MySQL**.

---

## Notes

- `COUNT(*)` counts all rows in each group.
- `COUNT(column_name)` counts only non-`NULL` values.
- Use `LEFT JOIN` if customers with no orders should also be included.
- `GROUP BY` is required when using aggregate functions like `COUNT()`.
- This is one of the most frequently asked SQL aggregation interview questions.

---

# 8. Retrieve All Employees Who Joined in 2023

## Scenario

Suppose you have an `employees` table.

| employee_id | employee_name | department | joining_date |
|-------------|---------------|------------|--------------|
| 1 | John | IT | 2022-08-15 |
| 2 | Jane | HR | 2023-01-10 |
| 3 | Mike | Sales | 2023-06-20 |
| 4 | Alice | Finance | 2024-02-01 |
| 5 | David | IT | 2023-11-05 |

We want to retrieve **all employees who joined in the year 2023**.

---

## Method 1: Using `YEAR()` (MySQL)

```sql
SELECT *
FROM employees
WHERE YEAR(joining_date) = 2023;
```

### Output

| employee_id | employee_name | department | joining_date |
|-------------|---------------|------------|--------------|
| 2 | Jane | HR | 2023-01-10 |
| 3 | Mike | Sales | 2023-06-20 |
| 5 | David | IT | 2023-11-05 |

### Explanation

- `YEAR(joining_date)` extracts the year from the date.
- The query returns employees whose joining year is **2023**.

---

## Method 2: Using a Date Range (Recommended)

```sql
SELECT *
FROM employees
WHERE joining_date >= '2023-01-01'
  AND joining_date < '2024-01-01';
```

### Explanation

- Retrieves employees who joined between **January 1, 2023** and **December 31, 2023**.
- This approach is generally more efficient because it can utilize an index on the `joining_date` column.

---

## Method 3: Using `BETWEEN`

```sql
SELECT *
FROM employees
WHERE joining_date BETWEEN '2023-01-01' AND '2023-12-31';
```

### Explanation

- `BETWEEN` includes both the start and end dates.
- This method is suitable when `joining_date` is a `DATE` column.

> **Note:** If `joining_date` is a `DATETIME` column, prefer the **date range** approach (`>= '2023-01-01' AND < '2024-01-01'`) to avoid missing records with time values on `2023-12-31`.

---

## Method 4: PostgreSQL

```sql
SELECT *
FROM employees
WHERE EXTRACT(YEAR FROM joining_date) = 2023;
```

---

## Method 5: SQL Server

```sql
SELECT *
FROM employees
WHERE YEAR(joining_date) = 2023;
```

---

## Notes

- `YEAR()` is supported in **MySQL** and **SQL Server**.
- `EXTRACT(YEAR FROM date)` is commonly used in **PostgreSQL** and **Oracle**.
- Using a **date range** (`>=` and `<`) is the recommended approach for better performance on indexed date columns.
- This is a common SQL interview question for testing date filtering.

---

# 9. Calculate Average Order Value Per Customer

## Scenario

Suppose you have an `orders` table.

| order_id | customer_id | order_amount |
|---------:|------------:|-------------:|
| 101 | 1 | 500 |
| 102 | 2 | 1200 |
| 103 | 1 | 700 |
| 104 | 3 | 1500 |
| 105 | 2 | 800 |
| 106 | 1 | 1000 |

We want to calculate the **average order value (AOV)** for each customer.

> **Average Order Value (AOV) = Total Order Amount / Number of Orders**

---

## Method 1: Using `AVG()` (Most Common)

```sql
SELECT
    customer_id,
    AVG(order_amount) AS average_order_value
FROM orders
GROUP BY customer_id;
```

### Output

| customer_id | average_order_value |
|------------:|--------------------:|
| 1 | 733.33 |
| 2 | 1000.00 |
| 3 | 1500.00 |

### Explanation

- `AVG(order_amount)` calculates the average value of all orders placed by each customer.
- `GROUP BY customer_id` groups orders by customer.

---

## Method 2: Using `SUM()` and `COUNT()`

```sql
SELECT
    customer_id,
    SUM(order_amount) AS total_order_amount,
    COUNT(*) AS total_orders,
    SUM(order_amount) / COUNT(*) AS average_order_value
FROM orders
GROUP BY customer_id;
```

### Output

| customer_id | total_order_amount | total_orders | average_order_value |
|------------:|-------------------:|-------------:|--------------------:|
| 1 | 2200 | 3 | 733.33 |
| 2 | 2000 | 2 | 1000.00 |
| 3 | 1500 | 1 | 1500.00 |

### Explanation

- `SUM(order_amount)` calculates the total amount spent by each customer.
- `COUNT(*)` counts the number of orders.
- Dividing the total amount by the number of orders gives the average order value.

---

## Method 3: Display Customer Name Along with Average Order Value

```sql
SELECT
    c.customer_id,
    c.customer_name,
    AVG(o.order_amount) AS average_order_value
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name;
```

### Output

| customer_id | customer_name | average_order_value |
|------------:|---------------|--------------------:|
| 1 | John | 733.33 |
| 2 | Jane | 1000.00 |
| 3 | Mike | 1500.00 |

---

## Method 4: Include Customers with No Orders

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(AVG(o.order_amount), 0) AS average_order_value
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name;
```

### Explanation

- `LEFT JOIN` returns all customers, including those with no orders.
- `AVG()` returns `NULL` for customers without orders.
- `COALESCE(..., 0)` replaces `NULL` with `0`.

---

## Notes

- `AVG()` automatically ignores `NULL` values.
- `GROUP BY` is required to calculate the average for each customer.
- `COALESCE()` is useful for handling customers who have not placed any orders.
- Average Order Value (AOV) is a key e-commerce KPI used to measure customer purchasing behavior.

---

# 10. Get the Latest Order Placed by Each Customer

## Scenario

Suppose you have an `orders` table.

| order_id | customer_id | order_date | order_amount |
|---------:|------------:|------------|-------------:|
| 101 | 1 | 2025-01-10 | 500 |
| 102 | 2 | 2025-01-12 | 1200 |
| 103 | 1 | 2025-02-05 | 700 |
| 104 | 3 | 2025-01-20 | 1500 |
| 105 | 2 | 2025-02-15 | 800 |
| 106 | 1 | 2025-03-01 | 1000 |

We want to retrieve the **latest order placed by each customer**.

---

## Method 1: Using a Correlated Subquery (Works in Most SQL Databases)

```sql
SELECT *
FROM orders o
WHERE order_date = (
    SELECT MAX(order_date)
    FROM orders
    WHERE customer_id = o.customer_id
);
```

### Output

| order_id | customer_id | order_date | order_amount |
|---------:|------------:|------------|-------------:|
| 106 | 1 | 2025-03-01 | 1000 |
| 105 | 2 | 2025-02-15 | 800 |
| 104 | 3 | 2025-01-20 | 1500 |

### Explanation

- The inner query finds the latest (`MAX`) order date for each customer.
- The outer query returns the corresponding order details.

---

## Method 2: Using `ROW_NUMBER()` (Recommended)

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    order_amount
FROM (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_id
               ORDER BY order_date DESC
           ) AS rn
    FROM orders
) AS latest_orders
WHERE rn = 1;
```

### Explanation

- `PARTITION BY customer_id` creates a separate ranking for each customer.
- `ORDER BY order_date DESC` ranks the newest order first.
- `ROW_NUMBER() = 1` returns the latest order for each customer.

---

## Method 3: Using `JOIN` with `MAX()`

```sql
SELECT o.*
FROM orders o
JOIN (
    SELECT
        customer_id,
        MAX(order_date) AS latest_order_date
    FROM orders
    GROUP BY customer_id
) latest
ON o.customer_id = latest.customer_id
AND o.order_date = latest.latest_order_date;
```

### Explanation

- The subquery finds the latest order date for each customer.
- The `JOIN` retrieves the complete order details.

---

## Method 4: Display Customer Name Along with Latest Order

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    o.order_amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
JOIN (
    SELECT
        customer_id,
        MAX(order_date) AS latest_order_date
    FROM orders
    GROUP BY customer_id
) latest
ON o.customer_id = latest.customer_id
AND o.order_date = latest.latest_order_date;
```

### Output

| customer_id | customer_name | order_id | order_date | order_amount |
|------------:|---------------|---------:|------------|-------------:|
| 1 | John | 106 | 2025-03-01 | 1000 |
| 2 | Jane | 105 | 2025-02-15 | 800 |
| 3 | Mike | 104 | 2025-01-20 | 1500 |

---

## Notes

- `ROW_NUMBER()` is the preferred solution in modern SQL databases because it is flexible and efficient.
- If multiple orders have the same latest date, `ROW_NUMBER()` returns only one row. Use `RANK()` or `DENSE_RANK()` if you want all tied latest orders.
- The `JOIN + MAX()` approach is compatible with most SQL databases.
- This is one of the most common SQL interview questions involving window functions and grouping.