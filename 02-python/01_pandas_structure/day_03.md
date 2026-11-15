# Pandas — Module 2: Selection with `loc`, `iloc` and Filters

Part of my data engineering study trail. This module covers how to select rows and columns by label or position, and how to filter a DataFrame with one or several conditions (the Pandas equivalent of `WHERE` and `SELECT` in SQL).

## Key concepts

1. **`loc` selects by label** (index name and column name). **`iloc` selects by position** (0, 1, 2, ...), like Python lists.
2. **Syntax:** `df.loc[rows, columns]` and `df.iloc[rows, columns]`. If the columns are omitted, all columns are returned.
3. **Slicing behaves differently:**
   - `iloc[0:3]` returns positions 0, 1, 2. The end is **excluded**.
   - `loc[0:3]` returns labels 0, 1, 2 **and 3**. The end is **included**.
4. **Boolean mask:** a condition like `df["nota"] > 7` returns a Series of `True`/`False`. Passing it to `df[...]` keeps only the `True` rows (like `WHERE`).
5. **Multiple conditions:** use `&` (and), `|` (or), `~` (not). Each condition must be wrapped in **parentheses**, because `&` and `|` have higher precedence than comparison operators. Python's `and` / `or` do not work here.
6. **Filter shortcuts:**
   - `col.isin([a, b])` is the SQL `IN`.
   - `col.between(a, b)` is the SQL `BETWEEN` and **includes both ends**.
7. **Filter + column selection in one call:** `df.loc[mask, ["col1", "col2"]]` (like `WHERE` and `SELECT` together).
8. **A single row is a Series.** `df.loc[4]` returns a Series. Use `df.loc[[4]]` (double brackets) to keep a DataFrame.

## Examples

```python
import pandas as pd

filmes = pd.DataFrame({
    "titulo": ["A", "B", "C", "D", "E"],
    "genero": ["drama", "ação", "drama", "comédia", "ação"],
    "nota": [8.1, 6.5, 7.2, 5.9, 8.8],
    "duracao": [120, 95, 140, 88, 150],
})

filmes.loc[2]                                   # row with label 2
filmes.iloc[0:3]                                # positions 0, 1, 2
filmes.loc[0:3]                                 # labels 0, 1, 2, 3 (4 rows)
filmes.loc[:, ["titulo", "nota"]]               # all rows, 2 columns
filmes.iloc[1:4, 0:2]                           # rows 1-3, columns 0-1

filmes[filmes["nota"] > 7]                                    # one filter
filmes[(filmes["nota"] > 7) & (filmes["duracao"] < 145)]      # AND
filmes[(filmes["genero"] == "ação") | (filmes["nota"] > 8)]   # OR
filmes[filmes["genero"].isin(["drama", "comédia"])]           # IN
filmes[filmes["nota"].between(6, 8)]                          # BETWEEN
filmes[~filmes["genero"].isin(["ação"])]                      # NOT

filmes.loc[filmes["nota"] > 7, ["titulo", "genero"]]          # filter + columns
```

## Practice exercise: clothing store, version 2

```python
import pandas as pd

dados = {
    "pedido": [101, 102, 103, 104, 105, 106, 107, 108],
    "cliente": ["Ana", "Bruno", "Carla", "Ana", "Diego", "Bruno", "Elisa", "Diego"],
    "cidade": ["Rio", "Niterói", "Rio", "Rio", "São Gonçalo", "Niterói", "Rio", "São Gonçalo"],
    "produto": ["camiseta", "tênis", "boné", "calça", "camiseta", "boné", "tênis", "calça"],
    "qtd": [2, 1, 3, 1, 5, 2, 1, 2],
    "preco_unit": [49.9, 249.9, 35.0, 119.9, 49.9, 35.0, 249.9, 119.9],
}

df = pd.DataFrame(dados)
df["faturamento"] = df["qtd"] * df["preco_unit"]
```

```python
# 1. Row with index 4 (all columns)
df.loc[4]

# 2. First 3 rows, only 'cliente' and 'produto' (positions 1 and 3)
df.iloc[:3, [1, 3]]

# 3. Orders with qtd >= 3
df[df["qtd"] >= 3]

# 4. Orders with qtd >= 2 AND preco_unit < 100
df[(df["qtd"] >= 2) & (df["preco_unit"] < 100)]

# 5. Orders from Ana or Diego
df[df["cliente"].isin(["Ana", "Diego"])]

# 6. Orders with preco_unit between 40 and 120 (inclusive)
df[df["preco_unit"].between(40, 120)]

# 7. Only 'cliente' and 'produto' for orders with faturamento > 100
df.loc[df["faturamento"] > 100, ["cliente", "produto"]]

# Bonus: orders that are not t-shirts and not from Rio
df[(~df["cidade"].isin(["Rio"])) & (~df["produto"].isin(["camiseta"]))]
```

Expected results:

| Item | Rows returned (order IDs) |
|---|---|
| 3 | 103, 105 |
| 4 | 101, 103, 105, 106 |
| 5 | 101, 104, 105, 108 |
| 6 | 101, 104, 105, 108 |
| 7 | 102, 103, 104, 105, 107, 108 (two columns only) |
| Bonus | 102, 106, 108 |

Equivalent forms for the bonus:

```python
# using != for single values
df[(df["cidade"] != "Rio") & (df["produto"] != "camiseta")]

# De Morgan: "neither A nor B" is "not (A or B)"
df[~((df["cidade"] == "Rio") | (df["produto"] == "camiseta"))]
```

## Takeaways

- `iloc` slices exclude the end; `loc` slices include it. Counting the rows returned is a good sanity check.
- When the task says "show the orders", return the full rows. Passing `["pedido"]` as the column argument returns only the IDs.
- Use the exact limits from the spec in `between`. It already includes both ends, so widening the range (for example `between(39, 121)`) can give wrong results with other data.
- Always wrap each condition in parentheses when combining with `&`, `|` and `~`.
- `isin` pays off when the list has several values; for a single value, `==` and `!=` are simpler.
- Re-run a cell after editing it, so the output on screen matches the code.

## Next

Module 3: creating new columns with `apply`, `map` and `np.where` (the equivalent of Excel formulas).