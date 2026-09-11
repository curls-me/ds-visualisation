# EDA Guideline

A reusable checklist for exploring a new dataset before visualizing it, distilled from the "Getting to Know the Data" workflow in [4_Visualization_exercise.ipynb](4_Visualization_exercise.ipynb).

**Rule of thumb**: always explore before you plot. Surprises here save hours later.

## 1. Load the data

```python
import pandas as pd

df = pd.read_csv("data/your_file.csv")
print(f"Shape: {df.shape}")
df.head()
```

## 2. Explore the data

**Shape, dtypes, non-null counts**

```python
df.info()
```

**Unique values in categorical columns** — spot typos, inconsistent labels, or surprisingly high cardinality

```python
for col in ["column_a", "column_b", "column_c"]:
    print(f"\n{col} ({df[col].nunique()} unique values):")
    print(df[col].value_counts().head(10))
```

**Missing values** — which columns, how many

```python
missing = df.isna().sum().sort_values(ascending=False)
missing[missing > 0]
```

**Summary statistics** — numeric columns

```python
df.describe()
```

Add `include="all"` to also see counts/uniques for categorical columns:

```python
df.describe(include="all")
```

## 3. Ask yourself (checkpoint)

Before writing any cleaning or plotting code, answer:

1. How many rows are there in total?
2. How many unique categories does each key column have?
3. Which column has the most missing values — and is that a problem for the question you're trying to answer?
4. Are any columns in the "wrong" format to plot directly (e.g. a range stored as text, a date stored as a string)?
5. Which column would you group/filter by to answer your actual question?

## 4. Clean the data

Common patterns, in order of how often you'll need them:

**Drop missing values in the columns you actually need** (not the whole df)

```python
subset = df[["col_a", "col_b"]].copy()
subset = subset.dropna()
```

**Convert a messy string column to a usable numeric one** — e.g. a range like `"$10,000-$14,999"` → lower bound as an int

```python
def get_first_number(x):
    """Extract the lower bound from a range string."""
    x = x.split("-")[0]
    x = x.replace(",", "").replace(">", "").replace("$", "").strip()
    return int(x)

subset["value_numeric"] = subset["value_range"].apply(get_first_number)
```

**Filter to the categories you care about**

```python
categories_of_interest = ["Category A", "Category B", "Category C"]
filtered = subset[subset["category_col"].isin(categories_of_interest)]
```

**Note (pandas 3.0+)**: always `.copy()` after slicing a DataFrame before adding new columns, to avoid `SettingWithCopyWarning` under Copy-on-Write.

## 5. Then, and only then — visualize

Once you can answer the checkpoint questions above and the data is clean, move on to choosing chart type and library (see [1_Exploratory_vs_explanatory_viz.md](1_Exploratory_vs_explanatory_viz.md)).

---

## Applying this to Scibloom

The same shape/missing-values/unique-categories pass is worth running on any dataset Scibloom ingests or displays — it's the fastest way to catch bad data (inconsistent category labels, unexpected nulls, wrongly-typed fields) before it reaches a chart, report, or user-facing feature.
