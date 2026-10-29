# Pandas ML Cheat Sheet

```python
import pandas as pd
```

## Load data

```python
df = pd.read_csv("data.csv")
df = pd.read_csv("data.csv", index_col="id")  # use a column as the index

df = pd.read_excel("data.xlsx")
df = pd.read_parquet("data.parquet")

df.head()          # first 5 rows
df.shape           # (rows, columns)
df.columns         # column names
df.dtypes          # data types
df.info()          # types and non-null counts
df.describe()      # summary statistics for numeric columns
```

## Explore data

```python
df.sample(5)                        # random rows
df.describe(include="object")       # summary for non-numeric columns
df.describe(include="all")          # every column

df["age"].value_counts()            # frequency of each value
df["age"].value_counts(dropna=False)  # include NaN in the counts
df["age"].nunique()                 # number of distinct values
df["age"].unique()                  # the distinct values

df["age"].mean()                    # also: median, std, min, max, sum
df["age"].quantile([0.25, 0.5, 0.75])

df.corr(numeric_only=True)          # pairwise correlations
df.groupby("sex")["survived"].mean()          # aggregate by category
pd.crosstab(df["sex"], df["survived"])        # counts across two columns
pd.crosstab(df["sex"], df["survived"], normalize="index")  # row proportions
```

## Missing values and duplicates

```python
df.isna()                                   # boolean frame
df.isna().sum()                             # missing count per column
df.isna().sum().sort_values(ascending=False)
df.isna().mean()                            # fraction missing per column
df.isna().any(axis=1)                       # rows with any missing value

df.duplicated().sum()                       # number of duplicate rows
df["id"].duplicated().sum()                 # duplicates in one column
```

## Clean data

```python
df = df.dropna()                            # drop rows with any NaN
df = df.dropna(subset=["age", "fare"])      # only require these columns
df = df.drop(columns=["ticket", "cabin"])   # remove columns
df = df.drop_duplicates()

df["age"] = df["age"].fillna(df["age"].median())
df["embarked"] = df["embarked"].fillna(df["embarked"].mode()[0])

df["sex"] = df["sex"].str.lower().str.strip()
df["age"] = pd.to_numeric(df["age"], errors="coerce")  # bad values become NaN

df = df.rename(columns={"Sex": "sex"})
df["sex_code"] = df["sex"].map({"male": 0, "female": 1})
df["family_size"] = df["sibsp"] + df["parch"] + 1

df["fare"] = df["fare"].clip(upper=df["fare"].quantile(0.99))  # cap outliers
```

## Create DataFrames

From a dictionary of columns:

```python
df = pd.DataFrame({
    "name": ["Ada", "Linus", "Grace"],
    "age": [36, 55, 40],
    "score": [91.5, 88.0, 97.0],
})
```

From rows:

```python
df = pd.DataFrame(
    [["Ada", 36], ["Linus", 55]],
    columns=["name", "age"],
)
```

Add or update a column:

```python
df["passed"] = df["score"] >= 90
df["age_next_year"] = df["age"] + 1
```

## Concatenate DataFrames

Stack rows (same columns):

```python
combined = pd.concat([df1, df2], axis=0, ignore_index=True)
```

`ignore_index=True` creates a fresh `0, 1, 2, ...` index instead of preserving the original indexes.

Put columns side by side (indexes are aligned):

```python
combined = pd.concat([features, labels], axis=1)
```

Keep track of which frame each row came from:

```python
combined = pd.concat(
    [train_df, test_df],
    keys=["train", "test"],
    names=["split", "original_index"],
)
```

By default, concat uses the union of columns and fills missing values with `NaN`. Use `join="inner"` to keep only columns present in every frame:

```python
combined = pd.concat([df1, df2], ignore_index=True, join="inner")
```

## Index scenarios

The index labels rows; it is not necessarily a column or a row count.

```python
df.index                    # inspect index labels
df = df.set_index("id")     # make a column the index
df = df.reset_index()       # turn the index back into a column
df = df.reset_index(drop=True)  # discard the old index
```

Select by **index label** with `.loc`:

```python
df.loc[5]           # row whose index label is 5
df.loc[5:10]        # labels 5 through 10, inclusive
df.loc[[2, 5, 9]]   # selected labels
```

Select by **row position** with `.iloc`:

```python
df.iloc[0]          # first row
df.iloc[0:5]        # first five rows (stop is excluded)
df.iloc[[0, 3, 8]]  # selected row positions
```

After filtering, the original index is retained unless you reset it:

```python
adults = df.loc[df["age"] >= 18].reset_index(drop=True)
```

## Access columns and cells

```python
df["age"]                 # one column (Series)
df[["name", "age"]]       # multiple columns (DataFrame)

df.loc[5, "age"]           # cell at index label 5, column "age"
df.iloc[0, 1]               # cell at first row, second column
df.loc[:, ["name", "age"]] # all rows, selected columns
df.iloc[:5, :2]             # first 5 rows, first 2 columns
```

Prefer `df["column"]` over `df.column`: bracket access works for column names that contain spaces or match DataFrame methods, such as `"class"` or `"mean"`.

## Conditional access and filtering

```python
df.loc[df["age"] >= 18]  # rows meeting one condition

# Combine conditions: use &, |, and parentheses
adults_with_high_scores = df.loc[
    (df["age"] >= 18) & (df["score"] >= 90)
]

# Either condition
subset = df.loc[
    (df["age"] < 18) | (df["score"] >= 95)
]

# Match values from a set
subset = df.loc[df["name"].isin(["Ada", "Grace"])]

# Check for missing values
missing_scores = df.loc[df["score"].isna()]
known_scores = df.loc[df["score"].notna()]
```

Select rows **and** columns conditionally:

```python
result = df.loc[df["score"] >= 90, ["name", "score"]]
```

Avoid Python’s `and` / `or` between pandas conditions; use `&` / `|` and wrap each condition in parentheses.
