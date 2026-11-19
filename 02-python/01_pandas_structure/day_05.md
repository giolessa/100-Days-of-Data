# Pandas — Module 3: New Columns with `np.where`, `np.select`, `map` and `apply`

Part of my data engineering study trail. This module covers how to create new columns from existing ones, from simple arithmetic to conditional logic (the Pandas equivalent of Excel formulas and SQL `CASE WHEN`).

## Key concepts

1. **Creating a column** is assigning to a key that does not exist yet: `df["new"] = ...`. When the calculation is a direct operation between columns (`df["a"] * df["b"]`), that is the best option: fast and simple.
2. **`np.where(condition, value_if_true, value_if_false)`** is a vectorized if/else over the whole column. Excel `IF`, SQL `CASE WHEN` with two outcomes. Requires `import numpy as np`.
3. **`np.select(conditions, values, default=...)`** is `CASE WHEN` with several bands. Conditions are evaluated **in order** and the first true one wins, so the order matters (check the most restrictive first). Excel `IFS`.
4. **`.map(dictionary)`** replaces each value of a Series using a dictionary. It works like a simple lookup (`VLOOKUP`). Values missing from the dictionary become `NaN`, so make sure every key is covered.
5. **`.apply(function)`** applies a function to each value of a Series. With `df.apply(function, axis=1)` the function receives the **whole row**, so it can use several columns (`row["qty"]`). `axis=1` means "one row at a time".
6. **Order of preference** (fastest to slowest): direct operation between columns, then `np.where` / `np.select`, then `map`, then `apply`. Use `apply` only when the logic does not fit the others. On millions of rows the difference is large.

## Examples

```python
import pandas as pd
import numpy as np

students = pd.DataFrame({
    "name": ["Lia", "Davi", "Rita", "Caio"],
    "course": ["ADS", "ADS", "Law", "Medicine"],
    "grade": [8.5, 4.0, 6.5, 9.2],
})

# 1) np.where: two outcomes
students["passed"] = np.where(students["grade"] >= 6, "yes", "no")

# 2) np.select: several bands (order matters, like CASE WHEN)
conditions = [students["grade"] >= 9, students["grade"] >= 6]
values = ["excellent", "good"]
students["level"] = np.select(conditions, values, default="insufficient")

# 3) map: translation through a dictionary
areas = {"ADS": "exact sciences", "Law": "humanities", "Medicine": "health"}
students["area"] = students["course"].map(areas)

# 4) apply on a column: the function receives one value
def initial(name):
    return name[0]

students["initial"] = students["name"].apply(initial)

# 5) apply by row (axis=1): the function receives the whole row
def summary(row):
    return f"{row['name']} ({row['course']}): {row['grade']}"

students["summary"] = students.apply(summary, axis=1)
```

## Practice exercise: clothing store, version 3

```python
import pandas as pd
import numpy as np

data = {
    "order": [101, 102, 103, 104, 105, 106, 107, 108],
    "customer": ["Ana", "Bruno", "Carla", "Ana", "Diego", "Bruno", "Elisa", "Diego"],
    "city": ["Rio", "Niterói", "Rio", "Rio", "São Gonçalo", "Niterói", "Rio", "São Gonçalo"],
    "product": ["t-shirt", "sneakers", "cap", "pants", "t-shirt", "cap", "sneakers", "pants"],
    "qty": [2, 1, 3, 1, 5, 2, 1, 2],
    "unit_price": [49.9, 249.9, 35.0, 119.9, 49.9, 35.0, 249.9, 119.9],
}

df = pd.DataFrame(data)
df["revenue"] = df["qty"] * df["unit_price"]
```

```python
# 1. free_shipping: "yes" if revenue >= 100, "no" otherwise (np.where)
df["free_shipping"] = np.where(df["revenue"] >= 100, "yes", "no")

# 2. tier: "large" if revenue >= 200, "medium" if >= 80, "small" otherwise (np.select)
conditions = [df["revenue"] >= 200, df["revenue"] >= 80]
values = ["large", "medium"]
df["tier"] = np.select(conditions, values, default="small")

# 3. category: t-shirt and pants -> clothing, cap -> accessory, sneakers -> footwear (map)
outfit = {"t-shirt": "clothing", "pants": "clothing", "cap": "accessory", "sneakers": "footwear"}
df["category"] = df["product"].map(outfit)

# 4. sale_type: "wholesale" if qty >= 5, "retail" otherwise (apply on a Series)
def classify_qty(qty):
    return "wholesale" if qty >= 5 else "retail"

df["sale_type"] = df["qty"].apply(classify_qty)

# 5. summary: one sentence per order (apply with axis=1)
def summary(row):
    return f"{row['customer']} bought {row['qty']}x {row['product']} in {row['city']}"

df["summary"] = df.apply(summary, axis=1)
```

Expected results:

| order | free_shipping | tier | category | sale_type | summary |
|---|---|---|---|---|---|
| 101 | no | medium | clothing | retail | Ana bought 2x t-shirt in Rio |
| 102 | yes | large | footwear | retail | Bruno bought 1x sneakers in Niterói |
| 103 | yes | medium | accessory | retail | Carla bought 3x cap in Rio |
| 104 | yes | medium | clothing | retail | Ana bought 1x pants in Rio |
| 105 | yes | large | clothing | wholesale | Diego bought 5x t-shirt in São Gonçalo |
| 106 | no | small | accessory | retail | Bruno bought 2x cap in Niterói |
| 107 | yes | large | footwear | retail | Elisa bought 1x sneakers in Rio |
| 108 | yes | large | clothing | retail | Diego bought 2x pants in São Gonçalo |

## Takeaways

- In `np.select`, the order of the conditions matters. Putting `>= 80` before `>= 200` would classify every value above 80 as "medium".
- `map` returns `NaN` for any value that is not in the dictionary, so check that all keys are covered.
- With `apply(axis=1)`, read every column the output needs. Compare the result with the requested format (a missing column in the f-string is an easy mistake).
- Prefer vectorized options (`np.where`, `np.select`) over `apply` whenever possible.
- Follow PEP 8 and indent with 4 spaces, which also keeps the repository consistent.
- Leaving `df` alone on the last line of a notebook cell shows a formatted table, easier to read than `print(df)`, which splits wide tables into blocks.

## Next

Module 4: data cleaning (nulls, duplicates, type conversion and dates).
