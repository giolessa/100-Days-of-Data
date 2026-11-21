# Pandas — Module 4: Data Cleaning

Part of my data engineering study trail. This module covers how to diagnose and fix dirty data: missing values, duplicates, wrong types, inconsistent text and invalid dates.

## Key concepts

1. **Inspect before cleaning.** Four diagnostic commands: `df.info()`, `df.dtypes`, `df.isna().sum()` (nulls per column) and `df.duplicated().sum()` (repeated rows).
2. **Missing values are `NaN`** (or `NaT` for dates). `isna()` and `notna()` work like SQL's `IS NULL` and `IS NOT NULL`, returning boolean masks that can be used as filters.
3. **Two ways to handle nulls:**
   - `dropna()` removes rows. `subset=["col"]` looks only at that column, and `how="all"` only drops rows where **every** column is null.
   - `fillna(value)` fills them. It accepts a fixed value, the mean, the median, or a dictionary with one value per column: `df.fillna({"city": "Not specified"})`. It is the SQL `COALESCE`.
4. **Filling is not neutral.** Filling with 0, the mean or the median changes the analysis results. The median is safer than the mean when there are extreme values. Decide per column and document why.
5. **Duplicates:** `duplicated()` marks rows as `True` when they repeat an **earlier** row. `drop_duplicates()` removes them. `subset=["order"]` compares only that column, and `keep="last"` keeps the last occurrence.
6. **Type conversion:**
   - `astype(float)` converts but **raises an error** if any text is not a number.
   - `pd.to_numeric(col, errors="coerce")` converts and turns anything invalid into `NaN`.
   - `pd.to_datetime(col, format="%d/%m/%Y", errors="coerce")` converts dates and turns invalid ones into `NaT`. The explicit `format` avoids day/month ambiguity.
   - `int64` does not accept `NaN`, so a column with nulls becomes `float64`, unless you use the nullable `"Int64"` type (capital I).
7. **Text preview** (full topic in Module 7): `.str.strip()` removes spaces at the ends, `.str.lower()` and `.str.title()` standardize case. Without it, `"Ana"`, `"ana "` and `"ANA"` count as three different customers.
8. **Recommended order:** standardize text and convert types first, then remove duplicates, then handle nulls. Standardizing can reveal hidden duplicates, and converting can create new nulls.
9. **Methods return a new DataFrame** instead of changing the original, so reassign: `df = df.drop_duplicates()`. Keep a copy with `df_original = df.copy()` before starting.

## Example

```python
import pandas as pd
import numpy as np

students = pd.DataFrame({
    "name": ["Lia", " davi", "Lia", None, "RITA"],
    "grade": ["8.5", "4.0", "8.5", "no grade", "6.5"],
    "submitted": ["01/11/2026", "02/11/2026", "01/11/2026", "03/11/2026", "30/02/2026"],
})

students.isna().sum()                     # nulls per column
students.duplicated().sum()               # 1 repeated row

students["name"] = students["name"].str.strip().str.title()
students["grade"] = pd.to_numeric(students["grade"], errors="coerce")
students["submitted"] = pd.to_datetime(students["submitted"], format="%d/%m/%Y", errors="coerce")

students = students.drop_duplicates()
students["name"] = students["name"].fillna("unknown")
students["grade"] = students["grade"].fillna(students["grade"].median())
```

## Practice exercise: dirty orders

```python
import pandas as pd
import numpy as np

data = {
    "order": [201, 202, 203, 203, 204, 205, 206, 207],
    "customer": ["Ana", " bruno", "Carla", "Carla", "DIEGO", None, "Elisa", "ana "],
    "value": ["49.9", "249.9", "n/d", "n/d", "119.9", "35.0", "abc", "99.8"],
    "date": ["10/11/2026", "11/11/2026", "12/11/2026", "12/11/2026",
             "13/11/2026", "31/02/2026", "15/11/2026", "16/11/2026"],
    "city": ["Rio", "Niterói", None, None, "Rio", "São Gonçalo", "Rio", "Rio"],
}

df = pd.DataFrame(data)
```

```python
# 1. Diagnostics: shape, dtypes, nulls per column, duplicated rows
print(df.shape)                              # (8, 5)
print(df.dtypes)
print(df.isna().sum())                       # customer: 1, city: 2
print(f"There is/are {df.duplicated().sum()} duplicate row(s)")   # 1

# 2. Customer: no leading/trailing spaces, first letter of each name capitalized
df["customer"] = df["customer"].str.strip().str.title()

# 3. Value: convert to numeric, invalid text becomes NaN
df["value"] = pd.to_numeric(df["value"], errors="coerce")
print(f"There are {df['value'].isna().sum()} null values")        # 3

# 4. Date: convert to datetime (day/month/year), nonexistent dates become NaT
df["date"] = pd.to_datetime(df["date"], format="%d/%m/%Y", errors="coerce")
print(f"There are {df['date'].isna().sum()} null dates")          # 1 (31/02/2026)

# 5. Duplicates: remove repeated rows
df = df.drop_duplicates()
print(f"{len(df)} rows remaining.")                               # 7

# 6. Nulls: fill customer, city and value
df["customer"] = df["customer"].fillna("Unknown")
df["city"] = df["city"].fillna("Not specified")
df["value"] = df["value"].fillna(df["value"].median())            # median of valid values: 99.8

# 7. Final check
print(df.dtypes)
print(df.isna().sum())                                            # only "date" still has 1 null
```

Expected final state:

| Column | dtype | Nulls |
|---|---|---|
| order | int64 | 0 |
| customer | object | 0 |
| value | float64 | 0 |
| date | datetime64[ns] | 1 |
| city | object | 0 |

### Why the date is left as `NaT`

- A date is a fact, not a quantity. Order 205 happened on a real day that is unknown.
- Any fill value (a mean date, the most common date, a fixed date) would be invented and would distort analysis by month, sales trends and period filters.
- Filling `value` with the median is an estimate that keeps the column usable (though it also changes the results), while an invented date states something false about when the order happened.
- What to do instead: keep `NaT`, exclude the order from date-based analysis with `df.dropna(subset=["date"])`, and report the error (`31/02`) to whoever owns the data.
- Rule of thumb: fill only when the estimate is reasonable for the intended use, and leave `NaN`/`NaT` when any value would be a guess.

### Bonus: mean value and distinct customers

```python
mean_value = df["value"].mean()                  # 107.73 (with the median filled in)
print(df["customer"].nunique())                  # 6, includes "Unknown"
```

- **Skipping the name standardization does not raise an error.** It silently gives 7 distinct customers instead of 6, because `"Ana"` and `"ana "` are counted as different people. This is the dangerous kind of mistake: the code runs and the result looks plausible.
- **The error that does appear comes from skipping the numeric conversion** (step 3), because `.mean()` does not work on text.
- **The `"Unknown"` placeholder counts as a customer.** The real count is 5 (Ana, Bruno, Carla, Diego, Elisa). Counting before `fillna` would give 5, since `nunique()` ignores nulls.
- **The mean depends on the fill.** With the median filled in, the mean is 107.73. Using only the 5 valid values, it is 110.90.

## Takeaways

- Follow the order: standardize text, convert types, remove duplicates, then handle nulls.
- `errors="coerce"` turns bad values into `NaN`/`NaT` instead of raising an error. Count them afterwards to know how much data was affected.
- Silent errors (such as unstandardized names) are worse than loud ones, so check counts like `nunique()` after cleaning.
- Every fill is a decision that changes the results. Document what was filled and why.
- Do not fill dates. Keep `NaT` and handle it explicitly.
- When copying a code block, reread the message in the `print`: the code can run without errors and still answer the wrong question.
- When the output of a cell is hidden or collapsed, expand it before sharing, so results can be checked.

## Next

Module 5: aggregation with `groupby`, `agg`, `value_counts` and `pivot_table`.
