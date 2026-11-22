# 100 Days of Data

A 100-day study challenge focused on **data analysis and data engineering**, with the goal of landing an internship. Every study day produces notes (and, when it makes sense, a notebook or script) that are documented here.

- **Start date:** 2026-11-14
- **Plan:** see [ROADMAP.md](ROADMAP.md) for the weekly routine, the sprint checklist and the cloud certification goal
- **Day counter:** each "Day" in the table below is a day I actually studied

## Topics

SQL · Python · Pandas · NumPy · Power BI · data fundamentals · cloud computing (AWS) · pipelines · portfolio projects

## Progress

| Day | Date | Topic | What I learned | Notes | Code |
|---|---|---|---|---|---|
| 1 | 2026-11-14 | Pandas: Series and DataFrame | Series vs DataFrame, `shape`, `dtypes`, `info`, `describe`, `head`/`tail`, vectorized operations | [notes](02-python/02-pandas/01-series-and-dataframe.md) | [notebook](02-python/02-pandas/01-series-and-dataframe.ipynb) |
| 2 | 2026-11-15 | SQL: NULL, COALESCE, CASE WHEN, DISTINCT | `IS NULL`, `COALESCE`, `CASE WHEN`, `DISTINCT`, logical order of execution | [notes](01-sql/01-fundamentals/01-null-coalesce-case-distinct.md) | — |
| 3 | 2026-11-16 | Pandas: `loc`, `iloc` and filters | label vs position selection, boolean masks, `&` `\|` `~`, `isin`, `between` | [notes](02-python/02-pandas/02-loc-iloc-filters.md) | [notebook](02-python/02-pandas/02-loc-iloc-filters.ipynb) |
| 4 | 2026-11-17 | NumPy fundamentals + SQL JOINs | arrays, `shape`, `dtype`, vectorization; `INNER`/`LEFT JOIN`, anti-join, self join | [numpy](02-python/01-numpy/01-arrays-shape-dtype-vectorized-operations.md) · [sql](01-sql/01-fundamentals/02-joins.md) | [notebook](02-python/01-numpy/01-arrays-shape-dtype-vectorized-operations.ipynb) |
| 5 | 2026-11-18 | Cloud: concepts | IaaS/PaaS/SaaS, deployment models, shared responsibility, regions and AZs | [notes](06-cloud/01-cloud-concepts.md) | — |
| 6 | 2026-11-19 | Pandas: new columns | `np.where`, `np.select`, `map`, `apply` (`axis=1`) | [notes](02-python/02-pandas/03-new-columns.md) | [notebook](02-python/02-pandas/03-new-columns.ipynb) |
| 7 | 2026-11-21 | Pandas: data cleaning | nulls, duplicates, `errors="coerce"`, dates, order of cleaning steps | [notes](02-python/02-pandas/04-data-cleaning.md) | [notebook](02-python/02-pandas/04-data-cleaning.ipynb) |
| 8 | 2026-11-22 | Pandas: aggregation + SQL: precise aggregation | `groupby`, named aggregation, `pivot_table`; `COUNT` variants, NULL in aggregates, `HAVING`, conditional aggregation | [pandas](02-python/02-pandas/05-aggregation.md) · [sql](01-sql/01-fundamentals/03-precise-aggregation.md) | [notebook](02-python/02-pandas/05-aggregation.ipynb) |

## Repository structure

```
100-Days-of-Data/
├── README.md
├── ROADMAP.md
├── 01-sql/
│   └── 01-fundamentals/
├── 02-python/
│   ├── 01-numpy/
│   └── 02-pandas/
├── 03-data-modeling/
├── 04-etl-pipelines/
├── 05-bi-and-visualization/
├── 06-cloud/
├── 07-projects/
└── resources/
```

## Naming conventions

- Folders and files use lowercase with hyphens.
- Inside each topic folder, files are numbered by module order, for example `04-data-cleaning.md`.
- Each module has a notes file (`.md`) and, when there is code, a notebook with the same name (`.ipynb`).
- Commit messages follow the pattern `docs(scope): short description`.
