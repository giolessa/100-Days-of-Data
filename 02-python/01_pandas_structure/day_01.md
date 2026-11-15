# Pandas — Module 1: Series and DataFrame

Part of my data engineering study trail. This module covers the basic structures of Pandas and how to inspect a table before analyzing it.

## Key concepts

1. **DataFrame** is the whole table, with rows and columns (like a spreadsheet or a SQL table).
2. **Series** is a single column, with an index beside it. `df["col"]` returns a Series.
3. **Index** is the label of each row. By default it is 0, 1, 2, ...
4. **Inspection commands:**
   - `df.shape` returns `(rows, columns)`. It is an attribute, so no parentheses.
   - `df.dtypes` shows the type of each column (`int64`, `float64`, `object` for text).
   - `df.info()` prints a summary (columns, types, non-null counts) and returns `None`, so do not wrap it in `print()`.
   - `df.describe()` gives statistics for the numeric columns.
   - `df.head(n)` / `df.tail(n)` show the first or last `n` **rows**.
5. **Series have their own methods** such as `.mean()`, `.max()` and `.sum()`. For a single number, `df["col"].max()` is better than `df["col"].describe()["max"]`, which computes every statistic just to use one.
6. **Vectorized operations** work on the whole column at once, with no `for` loop: `df["a"] * df["b"]`.

## Example

```python
import pandas as pd

dados = {
    "produto": ["camiseta", "calça", "boné", "tênis"],
    "qtd": [10, 4, 7, 2],
    "preco": [49.9, 119.9, 35.0, 249.9],
}

df = pd.DataFrame(dados)

print(df.shape)         # (4, 3)
print(df.dtypes)
print(df["preco"])      # a Series
print(df["preco"].mean())
```

## Practice exercise: clothing store

```python
import pandas as pd

dados = {
    "pedido": [101, 102, 103, 104, 105, 106],
    "cliente": ["Ana", "Bruno", "Carla", "Ana", "Diego", "Bruno"],
    "produto": ["camiseta", "tênis", "boné", "calça", "camiseta", "boné"],
    "qtd": [2, 1, 3, 1, 5, 2],
    "preco_unit": [49.9, 249.9, 35.0, 119.9, 49.9, 35.0],
}

df = pd.DataFrame(dados)
```

```python
# 1. Number of rows
total_linhas = df.shape[0]          # 6

# 2. Type of each column
print(df.dtypes)

# 3. Highest value in qtd
maior_qtd = df["qtd"].max()         # 5

# 4. Average unit price
media = df["preco_unit"].mean()     # ~89.93

# 5. Last 3 rows
print(df.tail(3))

# Bonus: total revenue
df["faturamento"] = df["preco_unit"] * df["qtd"]
print(df["faturamento"].sum())      # 894.1
```

## Takeaways

- `tail(3)` returns the last 3 **rows**, not columns. Mixing up rows and columns causes confusing errors later with `loc` and `iloc`.
- `df.shape[0]` is the row count and `df.shape[1]` is the column count.
- Assigning to a new key (`df["faturamento"] = ...`) creates a new column.
- Floats can show tiny imprecision (for example `894.1000000000001`). Use `round(total, 2)` when displaying.

## Next

Module 2: selection with `loc`, `iloc` and filters with multiple conditions.