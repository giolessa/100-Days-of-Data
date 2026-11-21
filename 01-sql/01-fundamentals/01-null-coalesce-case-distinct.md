# SQL — Module 1: NULL, COALESCE, CASE WHEN and DISTINCT

Part of my data engineering study trail. This module covers how SQL handles missing values and how to build conditional logic and remove duplicates.

## Key concepts

1. **`NULL` means "unknown or missing".** It is not zero and not an empty string.
2. **Comparing with `NULL` never returns true.** `col = NULL` and `col != NULL` both evaluate to `UNKNOWN`, and `WHERE` only keeps rows where the condition is true. Use `IS NULL` / `IS NOT NULL`.
3. **`NULL` contaminates arithmetic.** `10 + NULL` is `NULL`, so a calculation over a nullable column can silently disappear.
4. **`COALESCE(a, b, ...)`** returns the first non-`NULL` value. It is the standard way to say "if empty, use this instead".
5. **`CASE WHEN`** is SQL's if/else. Conditions are evaluated top to bottom and the first true one wins. `ELSE` covers the rest.
6. **`DISTINCT`** removes duplicate rows. It treats all `NULL`s as a single value.
7. **Logical execution order:** `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. Because `SELECT` runs after `WHERE`, an alias created in `SELECT` cannot be used in `WHERE`. A CTE or subquery is the usual workaround.

## Examples

```sql
-- rows with missing values
SELECT nome FROM alunos WHERE nota IS NULL;

-- replace NULL before doing math
SELECT nome, nota - COALESCE(faltas, 0) * 0.1 AS nota_ajustada
FROM alunos;

-- conditional classification
SELECT nome,
       CASE
           WHEN nota >= 7 THEN 'aprovado'
           WHEN nota >= 5 THEN 'recuperação'
           ELSE 'reprovado'
       END AS situacao
FROM alunos;

-- unique values
SELECT DISTINCT turma FROM alunos;
```

## Practice exercise: store orders

```sql
CREATE TABLE pedidos (
    id INTEGER PRIMARY KEY,
    cliente TEXT,
    cidade TEXT,
    produto TEXT,
    qtd INTEGER,
    preco_unit REAL,
    desconto REAL
);

INSERT INTO pedidos VALUES
(1, 'Ana',   'Rio',     'camiseta', 2, 49.9,  0.10),
(2, 'Bruno', 'Niterói', 'tênis',    1, 249.9, NULL),
(3, 'Carla', NULL,      'boné',     3, 35.0,  0.05),
(4, 'Ana',   'Rio',     'calça',    1, 119.9, NULL),
(5, 'Diego', 'Rio',     'camiseta', 5, 49.9,  0.20),
(6, 'Bruno', 'Niterói', 'boné',     2, 35.0,  NULL),
(7, 'Elisa', NULL,      'tênis',    1, 249.9, 0.15);
```

`desconto` is a fraction (0.10 = 10%).

### 1. Orders with no city

```sql
SELECT * FROM pedidos WHERE cidade IS NULL;
```

### 2. Final value of each order

```sql
SELECT id,
       (qtd * preco_unit) * (1 - COALESCE(desconto, 0)) AS valor_final
FROM pedidos;
```

### 3. Size band per order

The alias `valor_final` cannot be reused in the same `SELECT`, so a CTE computes it first.

```sql
WITH pedidos_calculados AS (
    SELECT id,
           (qtd * preco_unit) * (1 - COALESCE(desconto, 0)) AS valor_final
    FROM pedidos
)
SELECT id,
       valor_final,
       CASE
           WHEN valor_final >= 200 THEN 'grande'
           WHEN valor_final >= 80  THEN 'médio'
           ELSE 'pequeno'
       END AS faixa
FROM pedidos_calculados;
```

Expected final values: 89.82, 249.90, 99.75, 119.90, 199.60, 70.00, 212.415. Order 5 lands at 199.60, just under 200, so it is `médio`.

### 4. Distinct cities, excluding NULL

```sql
SELECT DISTINCT cidade
FROM pedidos
WHERE cidade IS NOT NULL;
```

## Takeaways

- `WHERE cidade = NULL` returns **zero rows**, with no error, because the comparison is `UNKNOWN`.
- `WHERE cidade != 'Rio'` also drops rows where `cidade` is `NULL`. To include them: `WHERE cidade != 'Rio' OR cidade IS NULL`.
- `COALESCE(x, 0)` is shorter than `CASE WHEN x IS NOT NULL THEN x ELSE 0 END` and is the idiomatic choice.
- Use the exact column names requested in a spec.

## Next

Module 2: JOINs (`INNER`, `LEFT`, `FULL`, self join) and finding rows without a match.
