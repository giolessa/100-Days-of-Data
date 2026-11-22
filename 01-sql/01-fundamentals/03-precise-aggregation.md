# SQL — Module 3: Precise Aggregation

Part of my data engineering study trail. This module covers how aggregate functions treat `NULL`, how to count rows, values and distinct values correctly, how `WHERE` and `HAVING` differ, and how to build several metrics in one pass with conditional aggregation.

## Key concepts

1. **`COUNT(*)` counts rows.** `COUNT(column)` counts only the rows where the column is **not `NULL`**. `COUNT(DISTINCT column)` counts the **different** values (also ignoring `NULL`). It is the same trio as in Pandas: `size`, `count` and `nunique`.
2. **Aggregates ignore `NULL`.** `SUM`, `AVG`, `MIN`, `MAX` and `COUNT(column)` skip nulls. That is why `AVG(discount)` divides only by the rows that **have** a discount, not by all rows. To treat null as 0, use `AVG(COALESCE(discount, 0))`, and the result changes.
3. **Empty group:** `SUM` over a set with no values returns `NULL` (not 0). When you need 0, wrap it: `COALESCE(SUM(...), 0)`. `COUNT` is the exception and returns 0. `AVG` over a set with no values also returns `NULL`.
4. **`NULL` forms its own group in `GROUP BY`.** If the grouping column has nulls, they appear together in one row with `NULL`. To give that group a name, use `COALESCE(city, 'unknown')` in the `SELECT` and repeat the same expression in the `GROUP BY`.
5. **`WHERE` vs `HAVING`:** `WHERE` filters **rows before** grouping and cannot use aggregates. `HAVING` filters **groups after** aggregating, which is where conditions such as `HAVING SUM(...) > 350` go. It is better to filter the raw aggregate in `HAVING` and round only in the `SELECT`, so rounding does not decide who passes the filter.
6. **Conditional aggregation** puts a `CASE WHEN` **inside** the aggregate, to compute several counts or sums in a single pass, one column per condition:
   - `COUNT(CASE WHEN category = 'clothing' THEN 1 END)` counts only the rows that meet the condition (a `CASE` without `ELSE` returns `NULL`, and `COUNT` ignores `NULL`).
   - `SUM(CASE WHEN category = 'clothing' THEN value ELSE 0 END)` sums only the value of those rows.
   - A proportion as an average: `AVG(CASE WHEN discount IS NOT NULL THEN 1.0 ELSE 0 END)` gives the fraction of rows with a discount.
7. **PostgreSQL also has `FILTER`,** a shorter form of the same idea: `COUNT(*) FILTER (WHERE category = 'clothing')`. `CASE WHEN` works in any database, so it is worth mastering both.
8. **`GROUP BY` rule in PostgreSQL:** every column in the `SELECT` that is not inside an aggregate must be in the `GROUP BY`. When grouping by an expression (like the `COALESCE` above), the whole expression must appear in both places.
9. **Rounding in PostgreSQL:** `ROUND(value, 2)` only accepts `NUMERIC`. If the column is `REAL`, cast first: `ROUND(SUM(x)::numeric, 2)`.
10. **Name every computed column with `AS`.** Without an alias, an expression such as `COALESCE(city, 'unknown')` shows up as `coalesce`.

## Examples

With the table `students(id, name, class, grade, scholarship)`, where `grade` and `scholarship` can be `NULL`:

```sql
-- the three counts side by side
SELECT COUNT(*)              AS total_rows,
       COUNT(grade)          AS with_grade,
       COUNT(DISTINCT class) AS classes
FROM students;

-- average ignoring nulls vs treating null as 0
SELECT AVG(grade)              AS avg_ignoring_nulls,
       AVG(COALESCE(grade, 0)) AS avg_nulls_as_zero
FROM students;

-- named NULL group, with HAVING
SELECT COALESCE(class, 'no class')  AS class_name,
       COUNT(*)                     AS students,
       ROUND(AVG(grade)::numeric, 1) AS average
FROM students
GROUP BY COALESCE(class, 'no class')
HAVING COUNT(*) >= 2
ORDER BY average DESC;

-- conditional aggregation: one column per condition
SELECT class,
       COUNT(*)                                       AS total,
       COUNT(CASE WHEN grade >= 7 THEN 1 END)         AS passed,
       COUNT(*) FILTER (WHERE scholarship IS NOT NULL) AS scholarship_students
FROM students
GROUP BY class;
```

## Practice exercise: store sales in SQL

```sql
CREATE TABLE sales (
    id INTEGER PRIMARY KEY,
    customer TEXT,
    city TEXT,
    category TEXT,
    qty INTEGER,
    unit_price REAL,
    discount REAL
);

INSERT INTO sales VALUES
(1,  'Ana',   'Rio',         'clothing',  2, 49.9,  0.10),
(2,  'Bruno', 'Niterói',     'footwear',  1, 249.9, NULL),
(3,  'Carla', 'Rio',         'accessory', 3, 35.0,  0.05),
(4,  'Ana',   'Rio',         'footwear',  1, 249.9, NULL),
(5,  'Diego', 'São Gonçalo', 'clothing',  5, 49.9,  0.20),
(6,  'Bruno', 'Niterói',     'accessory', 2, 35.0,  NULL),
(7,  'Elisa', NULL,          'clothing',  1, 119.9, 0.15),
(8,  'Carla', 'Rio',         'clothing',  4, 49.9,  NULL),
(9,  'Diego', 'São Gonçalo', 'clothing',  2, 119.9, NULL),
(10, 'Elisa', NULL,          'footwear',  1, 249.9, NULL);
```

The revenue of a row is `qty * unit_price`. The `discount` column is only used in item 4.

### 1. Total rows, rows with a discount and distinct customers

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) FILTER (WHERE discount IS NOT NULL) AS discount_rows,
    COUNT(DISTINCT customer) AS customers_distinct
FROM sales;
```

Result: 10, 4 and 5.

### 2. Orders and revenue by city, only cities above 350, highest first

```sql
SELECT
    COALESCE(city, 'unknown') AS city_name,
    COUNT(id) AS number_orders,
    ROUND(SUM(qty * unit_price)::numeric, 2) AS total_revenue
FROM sales
GROUP BY COALESCE(city, 'unknown')
HAVING SUM(qty * unit_price) > 350
ORDER BY total_revenue DESC;
```

| city_name | number_orders | total_revenue |
|---|---|---|
| Rio | 4 | 654.30 |
| São Gonçalo | 2 | 489.30 |
| unknown | 2 | 369.80 |

Niterói (319.90) does not pass the `HAVING` filter. `ORDER BY` runs last, so it can use the alias `total_revenue`.

### 3. Clothing and footwear orders per city (conditional aggregation)

```sql
SELECT
    COALESCE(city, 'unknown') AS city_name,
    COUNT(CASE WHEN category = 'clothing' THEN 1 END) AS clothing_orders,
    COUNT(CASE WHEN category = 'footwear' THEN 1 END) AS footwear_orders
FROM sales
GROUP BY COALESCE(city, 'unknown');
```

| city_name | clothing_orders | footwear_orders |
|---|---|---|
| São Gonçalo | 2 | 0 |
| unknown | 1 | 1 |
| Niterói | 0 | 1 |
| Rio | 2 | 1 |

### 4. Average discount per category, two ways

```sql
SELECT
    category,
    ROUND(AVG(discount)::numeric, 3) AS avg_ignoring_nulls,
    ROUND(AVG(COALESCE(discount, 0))::numeric, 3) AS avg_nulls_as_zero
FROM sales
GROUP BY category;
```

| category | avg_ignoring_nulls | avg_nulls_as_zero |
|---|---|---|
| footwear | NULL | 0.000 |
| accessory | 0.050 | 0.025 |
| clothing | 0.150 | 0.090 |

**Why they differ:** `NULL` is the absence of a value, not zero. `AVG(discount)` only counts the rows that have a discount, while `AVG(COALESCE(discount, 0))` turns each null into 0 and counts it in the divisor.

- `accessory`: 1 discount (0.05) in 2 rows, so 0.05 / 1 = 0.050 versus 0.05 / 2 = 0.025.
- `clothing`: 3 discounts adding up to 0.45 in 5 rows, so 0.45 / 3 = 0.150 versus 0.45 / 5 = 0.090.
- `footwear`: no row has a discount. `AVG` over no values returns `NULL`, while with `COALESCE` the 3 rows become 0, so the average is 0.000.

### 5. Customers who bought in more than one category

```sql
SELECT
    customer,
    COUNT(DISTINCT category) AS qty_category
FROM sales
GROUP BY customer
HAVING COUNT(DISTINCT category) > 1;
```

Result: Ana, Bruno, Carla and Elisa, with 2 categories each. Diego only bought clothing.

### Bonus: percentage of orders with a discount per city

```sql
SELECT
    COALESCE(city, 'unknown') AS city_name,
    ROUND(COUNT(discount) * 100.0 / COUNT(*), 1) AS pct_with_discount
FROM sales
GROUP BY COALESCE(city, 'unknown');
```

| city_name | pct_with_discount |
|---|---|
| São Gonçalo | 50.0 |
| unknown | 50.0 |
| Niterói | 0.0 |
| Rio | 50.0 |

An equivalent form uses the proportion-as-average idea: `ROUND(AVG(CASE WHEN discount IS NOT NULL THEN 1.0 ELSE 0 END) * 100, 1)`.

## Takeaways

- `COUNT(*)`, `COUNT(column)` and `COUNT(DISTINCT column)` answer three different questions. Choose according to what is being counted.
- Aggregates skip `NULL`. Decide whether a missing value should count as zero (`COALESCE`) or be left out, because the average changes.
- `HAVING` filters groups after aggregating. Filter the raw aggregate there and round only for display.
- `ORDER BY` is not implicit. If the task asks for a ranking, add it explicitly.
- Name every computed column and double-check the names against the spec (for example `footwear_orders`, not `footware_orders`). Copied aliases from another query can silently mislabel a column.
- Include the grouping column in the `SELECT`, otherwise the result cannot be read.
- Use conditional aggregation to build several metrics in one query instead of running one query per condition.

## Next

Module 4: subqueries, CTEs and set operations (`IN`, `EXISTS`, `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`).
