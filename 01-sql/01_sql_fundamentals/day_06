# SQL — Module 2: JOINs

Part of my data engineering study trail. This module covers how to combine rows from different tables: `INNER JOIN`, `LEFT JOIN`, anti-joins, self joins, and aggregating after a join.

## Key concepts

1. **A JOIN combines rows from two tables** using a condition written in `ON`, usually a foreign key equal to a primary key: `ON o.customer_id = c.id`.
2. **Aliases** (`customers c`, `orders o`) shorten names. Once a table has an alias, only the alias can be used: `c.id` works, `customers.id` does not.
3. **`INNER JOIN`** returns only the rows that have a match in **both** tables. Rows without a match disappear.
4. **`LEFT JOIN`** returns **all** rows from the left table (the one after `FROM`). When there is no match on the right, the right-side columns are `NULL`. It does not filter the left table, it preserves it entirely.
5. **Anti-join** (rows with no match): use a `LEFT JOIN` and filter `WHERE right_table.id IS NULL`. This is the standard way to answer "customers who never ordered".
6. **`FULL JOIN`** returns everything from both sides, with `NULL` where there is no match. `RIGHT JOIN` is the mirror of `LEFT JOIN`, and in practice it can almost always be replaced by a `LEFT JOIN` with the tables swapped. In SQLite, `RIGHT` and `FULL` joins exist only from version 3.39.
7. **Self join** joins a table with itself using two different aliases. The classic case is an employee and their manager stored in the same table.
8. **`ON` vs `WHERE` in a `LEFT JOIN`:** a condition in `ON` is applied **before** the `NULL`s are filled in, so the left side stays complete. A condition on the right table in `WHERE` is applied **after**, and it removes the `NULL` rows, which turns the `LEFT JOIN` into an `INNER JOIN` in practice. The exception is `IS NULL` (the anti-join), which is exactly what you want.
9. **`COUNT(*)` vs `COUNT(column)` with `LEFT JOIN`:** `COUNT(*)` counts the row even when the right side is `NULL`, while `COUNT(o.id)` skips `NULL`s. To count orders per customer, use `COUNT(o.id)`.
10. **`GROUP BY` in PostgreSQL:** every column in the `SELECT` that is not inside an aggregate must appear in the `GROUP BY` (use the id **and** the name).

## Examples

With the tables `students(id, name, class_id)` and `classes(id, title)`:

```sql
-- INNER: only students that have a class
SELECT s.name, c.title
FROM students s
INNER JOIN classes c ON s.class_id = c.id;

-- LEFT: all students, even without a class (title comes as NULL)
SELECT s.name, c.title
FROM students s
LEFT JOIN classes c ON s.class_id = c.id;

-- anti-join: students without a class
SELECT s.name
FROM students s
LEFT JOIN classes c ON s.class_id = c.id
WHERE c.id IS NULL;

-- count per class, including empty classes
SELECT c.title, COUNT(s.id) AS total_students
FROM classes c
LEFT JOIN students s ON s.class_id = c.id
GROUP BY c.id, c.title;
```

## Practice exercise: clothing store in SQL

```sql
CREATE TABLE customers (
    id INTEGER PRIMARY KEY,
    name TEXT,
    city TEXT
);

CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    product TEXT,
    qty INTEGER,
    unit_price REAL
);

CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name TEXT,
    manager_id INTEGER
);

INSERT INTO customers VALUES
(1, 'Ana',   'Rio'),
(2, 'Bruno', 'Niterói'),
(3, 'Carla', 'Rio'),
(4, 'Diego', 'São Gonçalo'),
(5, 'Elisa', 'Rio'),
(6, 'Fábio', 'Niterói');

INSERT INTO orders VALUES
(101, 1, 't-shirt',  2, 49.9),
(102, 2, 'sneakers', 1, 249.9),
(103, 1, 'pants',    1, 119.9),
(104, 4, 't-shirt',  5, 49.9),
(105, 2, 'cap',      2, 35.0),
(106, 7, 'cap',      1, 35.0),
(107, 4, 'pants',    2, 119.9);

INSERT INTO employees VALUES
(1, 'Marta',  NULL),
(2, 'Paulo',  1),
(3, 'Laura',  1),
(4, 'Renan',  2),
(5, 'Sofia',  2);
```

Order 106 points to customer 7, who does not exist. SQLite does not enforce foreign keys by default, so the row was accepted on purpose.

### 1. Order id, customer name and product for orders that have a matching customer

```sql
SELECT o.id,
       co.name,
       o.product
FROM orders o
INNER JOIN customers co ON o.customer_id = co.id;
```

Returns orders 101, 102, 103, 104, 105 and 107. Order 106 is left out.

### 2. All customers with the id and product of their orders (NULL when there is none)

```sql
SELECT co.name,
       o.id,
       o.product
FROM customers co
LEFT JOIN orders o ON o.customer_id = co.id;
```

Using `o.id` (the order id) instead of `co.id` makes the `NULL` appear in both order columns for Carla, Elisa and Fábio.

### 3. Customers who never placed an order

```sql
SELECT co.name
FROM customers co
LEFT JOIN orders o ON o.customer_id = co.id
WHERE o.id IS NULL;
```

Returns Carla, Elisa and Fábio.

### 4. Total spent per customer (0 when there are no orders)

```sql
SELECT co.name,
       COALESCE(ROUND(SUM(o.qty * o.unit_price)::numeric, 2), 0) AS total_spent
FROM customers co
LEFT JOIN orders o ON o.customer_id = co.id
GROUP BY co.id, co.name
ORDER BY total_spent DESC;
```

| name | total_spent |
|---|---|
| Diego | 489.30 |
| Bruno | 319.90 |
| Ana | 219.70 |
| Carla | 0 |
| Elisa | 0 |
| Fábio | 0 |

`COALESCE` goes **outside** the `SUM`, because the `NULL` only appears after summing (a customer with no orders). The `::numeric` cast is PostgreSQL syntax and is needed because `ROUND(value, 2)` only works with `NUMERIC` there. In SQLite, `ROUND(SUM(...), 2)` works directly.

### 5. Each employee with their manager's name

```sql
SELECT e.name AS employee,
       COALESCE(ma.name, 'no manager') AS manager
FROM employees e
LEFT JOIN employees ma ON e.manager_id = ma.id;
```

| employee | manager |
|---|---|
| Marta | no manager |
| Paulo | Marta |
| Laura | Marta |
| Renan | Paulo |
| Sofia | Paulo |

### Bonus: orders that point to a customer that does not exist

```sql
SELECT o.id AS order_id
FROM orders o
LEFT JOIN customers co ON o.customer_id = co.id
WHERE co.id IS NULL;
```

Returns order 106. This is the same anti-join as item 3, with the tables swapped. An earlier attempt used `WHERE o.customer_id >= 6`, which only worked because of today's data. Testing the result of the join (`co.id IS NULL`) keeps working when the data changes.

## Takeaways

- Use the alias in every column reference: `c.id`, not `customers.id`.
- To keep every row of one table, put it on the **left** of a `LEFT JOIN`. To find rows without a match, test the right table's key with `IS NULL`.
- Aggregate after a `LEFT JOIN` with `SUM` or `COUNT(o.id)`, group by id **and** name, and wrap the result in `COALESCE` to show 0 instead of `NULL`.
- `REAL` columns have floating-point imprecision (for example `99.80000305175781`). For money, use `NUMERIC(10,2)` columns, or cast to `numeric` before rounding in PostgreSQL.
- Add an `ORDER BY`: without it, the row order is not guaranteed.
- Keep each query's comment aligned with what the query does.
- A foreign key that is not enforced can leave orphan rows (like order 106). An anti-join is a quick way to audit them.

## Next

Module 3 SQL: precise aggregation (`COUNT(*)` vs `COUNT(column)`, `COUNT(DISTINCT ...)`, conditional aggregation).
