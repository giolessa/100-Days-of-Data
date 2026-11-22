# Pandas — Module 5: Aggregation

Part of my data engineering study trail. This module covers how to summarize data with `groupby`, named aggregations, `value_counts`, `pivot_table` and `crosstab` (the Pandas equivalents of SQL `GROUP BY` and Excel pivot tables).

## Key concepts

1. **`groupby` works in three steps:** split the table into groups (one per value of the key column), apply an aggregation to each group, and combine the results. It is SQL's `GROUP BY`. Example: `df.groupby("city")["revenue"].sum()` returns a **Series** with one row per city.
2. **Several aggregations at once** with `agg`. The most readable way is **named aggregation**, where you choose the name of each result column:
   `df.groupby("city").agg(total=("revenue", "sum"), orders=("order", "count"))`
   Each argument is `new_name=("column", "function")`. It is the equivalent of `SELECT SUM(...) AS total, COUNT(...) AS orders`.
3. **Common aggregation functions:** `sum`, `mean`, `min`, `max`, `count`, `nunique` and `size`. The important differences:
   - `size` counts the **rows** in the group, like `COUNT(*)`.
   - `count` counts the **non-null** values of a column, like `COUNT(column)`.
   - `nunique` counts **distinct** values, like `COUNT(DISTINCT column)`.
4. **More than one grouping key:** `groupby(["city", "category"])` creates a two-level index (a `MultiIndex`). To get a regular table with the keys as normal columns, finish with **`.reset_index()`** (or use `as_index=False`). This is a very common step, because it makes later filtering and joining easier.
5. **`value_counts()`** is the shortcut for "how many times does each value appear". The result is sorted from largest to smallest, and `normalize=True` returns proportions instead of counts.
6. **Ranking and top N:** `sort_values(ascending=False)` sorts and `.head(n)` takes the first `n`. It is `ORDER BY ... DESC LIMIT n`.
7. **`WHERE` vs `HAVING` in Pandas:** filtering **before** `groupby` is the `WHERE` (for example `df[df["qty"] > 1].groupby(...)`). Filtering the **result** of the aggregation is the `HAVING` (for example `totals[totals > 200]`).
8. **`pivot_table`** reshapes the aggregation into a table with rows and columns, like an Excel pivot table:
   `df.pivot_table(index="city", columns="category", values="revenue", aggfunc="sum", fill_value=0)`
   Combinations that do not exist are `NaN`, and `fill_value=0` replaces them with 0. **`pd.crosstab(df["city"], df["category"])`** follows the same idea but **counts** the rows in each combination.
9. **Null keys:** `groupby` **ignores** rows where the grouping column is `NaN` by default. With `dropna=False` they become their own group. This matters after the cleaning steps from Module 4.

## Examples

```python
import pandas as pd

grades = pd.DataFrame({
    "class": ["A", "A", "B", "B", "B", "C"],
    "subject": ["math", "lang", "math", "lang", "math", "lang"],
    "grade": [8.0, 6.5, 5.0, 7.5, 9.0, 4.0],
})

# one aggregation
grades.groupby("class")["grade"].mean()

# several aggregations with names (named aggregation)
summary = grades.groupby("class").agg(
    average=("grade", "mean"),
    highest=("grade", "max"),
    exams=("grade", "count"),
)

# two keys + reset_index to get a regular table
by_class_subject = (
    grades.groupby(["class", "subject"])["grade"].mean().reset_index()
)

# frequency of each value
grades["class"].value_counts()
grades["class"].value_counts(normalize=True)   # proportions

# top 2 classes by average
grades.groupby("class")["grade"].mean().sort_values(ascending=False).head(2)

# pivot table
grades.pivot_table(index="class", columns="subject", values="grade",
                   aggfunc="mean", fill_value=0)

# HAVING: only classes with an average above 6
averages = grades.groupby("class")["grade"].mean()
averages[averages > 6]
```

## Practice exercise: store sales, version 4

```python
import pandas as pd

data = {
    "order": [301, 302, 303, 304, 305, 306, 307, 308, 309, 310, 311, 312],
    "customer": ["Ana", "Bruno", "Carla", "Ana", "Diego", "Bruno",
                 "Elisa", "Carla", "Diego", "Ana", "Elisa", "Bruno"],
    "city": ["Rio", "Niterói", "Rio", "Rio", "São Gonçalo", "Niterói",
             "Rio", "Rio", "São Gonçalo", "Rio", "Rio", "Niterói"],
    "product": ["t-shirt", "sneakers", "cap", "pants", "t-shirt", "cap",
                "sneakers", "t-shirt", "pants", "sneakers", "cap", "pants"],
    "category": ["clothing", "footwear", "accessory", "clothing", "clothing", "accessory",
                 "footwear", "clothing", "clothing", "footwear", "accessory", "clothing"],
    "qty": [2, 1, 3, 1, 5, 2, 1, 4, 2, 1, 2, 3],
    "unit_price": [49.9, 249.9, 35.0, 119.9, 49.9, 35.0,
                   249.9, 49.9, 119.9, 249.9, 35.0, 119.9],
}

df = pd.DataFrame(data)
df["revenue"] = df["qty"] * df["unit_price"]
```

```python
# 1. Total revenue by city
df.groupby("city")["revenue"].sum()

# 2. Number of orders per category, in two ways
df["category"].value_counts()
df.groupby("category").size().reset_index(name="count")

# 3. Summary per product with named aggregation
summary = df.groupby("product", as_index=False).agg(
    total_qty=("qty", "sum"),
    avg_revenue=("revenue", "mean"),
    max_revenue=("revenue", "max"),
)

# 4. Revenue by city and category, regular table, sorted from highest to lowest
by_city_category = (
    df.groupby(["city", "category"])["revenue"].sum().reset_index()
    .sort_values(by="revenue", ascending=False)
)

# 5. Pivot table: city in rows, category in columns, sum of revenue, missing combinations as 0
df.pivot_table(index="city", columns="category", values="revenue",
               aggfunc="sum", fill_value=0)

# 6. Top 3 customers by spending
top3 = (
    df.groupby("customer", as_index=False)["revenue"].sum()
    .sort_values(by="revenue", ascending=False)
    .head(3)
)

# Bonus: percentage of total revenue per city
summary_city = df.groupby("city", as_index=False)["revenue"].sum()
summary_city["percentage"] = (summary_city["revenue"] / summary_city["revenue"].sum() * 100).round(1)
summary_city = summary_city.sort_values(by="percentage", ascending=False)
```

Expected results:

| Item | Result |
|---|---|
| 1 | Niterói 679.6, Rio 1094.1, São Gonçalo 489.3 |
| 2 | clothing 6, footwear 3, accessory 3 |
| 3 | cap: 7 / 81.67 / 105.0 · pants: 6 / 239.80 / 359.7 · sneakers: 3 / 249.90 / 249.9 · t-shirt: 11 / 182.97 / 249.5 |
| 4 | Rio footwear 499.8 · São Gonçalo clothing 489.3 · Rio clothing 419.3 · Niterói clothing 359.7 · Niterói footwear 249.9 · Rio accessory 175.0 · Niterói accessory 70.0 |
| 5 | Niterói: accessory 70.0, clothing 359.7, footwear 249.9 · Rio: accessory 175.0, clothing 419.3, footwear 499.8 · São Gonçalo: accessory 0, clothing 489.3, footwear 0 |
| 6 | Bruno 679.6, Diego 489.3, Ana 469.6 |
| Bonus | Rio 48.3, Niterói 30.0, São Gonçalo 21.6 |

## Takeaways

- Check which aggregation function you are using: `size()` counts rows, `sum()` adds values. Using `size()` when the task asks for revenue runs without errors and returns plausible-looking numbers, so always compare the result with an independent check, such as the pivot table.
- Keep numeric columns numeric. Formatting a percentage as text (`"48.30%"`) removes the ability to sort, filter or sum it. Format only when displaying.
- Rounded percentages may not add up to exactly 100 (here 48.3 + 30.0 + 21.6 = 99.9). This is a rounding effect, not an error.
- Use named aggregation (`name=("column", "function")`) for readable, well-labeled results.
- Use `reset_index()` or `as_index=False` to turn the grouped result back into a regular table.
- Filter before `groupby` for the `WHERE` behavior and after it for the `HAVING` behavior.
- Keep names in one language: the output column is better named `count` than a Portuguese label inside an English notebook.

## Next

Module 6: combining tables with `merge` and `concat` (the Pandas equivalent of SQL JOINs).
