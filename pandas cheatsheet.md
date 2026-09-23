# Pandas Cheatsheet

```python
import pandas as pd
```

---

## 1. Read

### read_csv
```python
pd.read_csv(path, sep=",", header=0, index_col=None, usecols=["a"],
  dtype={"a": "int32"}, parse_dates=["dt"], na_values=["NA"], nrows=N, skiprows=N, chunksize=N)
```
<details>
<summary>Uses</summary>

- `sep` — `"\t"` for TSV, `";"` for EU-style vendor files
- `header` — `None` when the file has no header row
- `index_col` — make `"dt"` the index straight away for time series
- `usecols` — load only needed columns from a wide file to save RAM
- `dtype` — keep IDs as `str` (leading zeros), downcast to `int32`/`category`
- `parse_dates` — turn date strings into `datetime64` on load
- `na_values` — treat `"NA"`, `"-"`, `"N/A"` as NaN
- `nrows` — peek at the first N rows of a huge file
- `skiprows` — skip junk/banner lines at the top
- `chunksize` — iterate a file bigger than memory, chunk by chunk

</details>

### read_json
```python
pd.read_json(path, orient="records", lines=True, dtype={"a": "int32"}, convert_dates=["dt"])
```
<details>
<summary>Uses</summary>

- `orient` — `"records"` = list of dicts (typical API output)
- `lines` — `True` for JSONL / log files (one object per line)
- `dtype` — stop pandas guessing types wrongly
- `convert_dates` — parse named date columns

</details>

---

## 2. Filter Rows and Columns

### loc / iloc
```python
df.loc[df.a > 5, ["a", "b"]]; df.iloc[0:5, 0:3]
```
<details>
<summary>Uses</summary>

- `loc[mask, cols]` — filter rows by condition and pick columns in one step (label based)
- `iloc[r, c]` — first/last N rows, positional slicing

</details>

### query / isin / between
```python
df.query("a > 5 and b == @x"); df[df.a.isin([1, 2]) & df.b.between(1, 9)]
```
<details>
<summary>Uses</summary>

- `query` + `@x` — readable filters using a local variable
- `isin` — keep rows whose value is in a list (e.g. tickers)
- `between` — inclusive range filter (dates, prices)
- `& | ~` — AND / OR / NOT; always wrap each condition in `()`

</details>

### filter
```python
df.filter(items=["a"], like="px", regex="^c_", axis=1)
```
<details>
<summary>Uses</summary>

- `items` — exact column names
- `like` — columns containing a substring (`"px"` → `bid_px`, `ask_px`)
- `regex` — pattern match (`"^c_"` → all columns starting `c_`)
- `axis` — `1` = columns, `0` = index labels

</details>

---

## 3. Index

### set_index
```python
df.set_index(["k1", "k2"], drop=True, append=False, verify_integrity=True)
```
<details>
<summary>Uses</summary>

- keys (list) — build a MultiIndex, e.g. `["dt", "tkr"]`
- `drop` — `False` keeps the column as well as the index
- `append` — add to the existing index instead of replacing it
- `verify_integrity` — raise if keys are duplicated (catch bad data early)

</details>

### reset_index
```python
df.reset_index(level=None, drop=False, names="idx")
```
<details>
<summary>Uses</summary>

- `level` — reset only one level of a MultiIndex
- `drop` — `True` throws the old index away (after filtering/sorting)
- `names` — name the new column(s) created from the index

</details>

---

## 4. Groupby and Agg

### groupby + agg
```python
df.groupby(["k"], as_index=False, sort=True, dropna=True, observed=True)
  .agg(tot=("amt", "sum"), n=("id", "nunique"))
```
<details>
<summary>Uses</summary>

- `as_index` — `False` returns keys as columns (flat, SQL-like)
- `sort` — `False` is faster when group order doesn't matter
- `dropna` — `False` keeps NaN keys as their own group
- `observed` — `True` on categoricals, to skip empty combinations
- named agg `new=(col, fn)` — clean output column names, no MultiIndex columns
- dict `{"a": ["mean", "max"]}` — several functions per column

</details>

### transform
```python
df.groupby("k")["x"].transform("mean")
```
<details>
<summary>Uses</summary>

- `transform` — same length as df: demeaning, % of group total, z-score

</details>

---

## 5. Merge

```python
pd.merge(l, r, how="left", on="k", left_on=None, right_on=None,
  left_index=False, right_index=False, suffixes=("_l", "_r"), indicator=True, validate="1:1")
```
<details>
<summary>Uses</summary>

- `how` — `inner` / `left` / `right` / `outer` / `cross` (all pairs)
- `on` — same key name in both frames
- `left_on` / `right_on` — key names differ (`"sym"` vs `"ticker"`)
- `left_index` / `right_index` — join on an index instead of a column
- `suffixes` — rename clashing non-key columns
- `indicator` — adds `_merge` column: find unmatched rows (reconciliation)
- `validate` — `"1:1"`, `"1:m"`, `"m:1"`: raise on unexpected duplicate keys

</details>

---

## 6. Join

```python
l.join(r, on=None, how="left", lsuffix="_l", rsuffix="_r", sort=False)
```
<details>
<summary>Uses</summary>

- (default) — index-to-index join; shortest syntax for aligned frames
- `on` — left column matched to right's index
- `how` — same options as merge; default is `left`
- `lsuffix` / `rsuffix` — required if column names clash
- `sort` — sort the result by the join key

</details>

---

## 7. Concat

```python
pd.concat([d1, d2], axis=0, join="outer", ignore_index=True, keys=["d1", "d2"], verify_integrity=False)
```
<details>
<summary>Uses</summary>

- `axis` — `0` stack rows (daily files), `1` side by side
- `join` — `"inner"` keeps only common columns
- `ignore_index` — fresh 0..N index after stacking
- `keys` — tag each source; builds a MultiIndex level
- `verify_integrity` — raise if resulting index has duplicates

</details>

---

## 8. Sorting

### sort_values
```python
df.sort_values(by=["a", "b"], ascending=[True, False], na_position="last",
  kind="mergesort", ignore_index=True)
```
<details>
<summary>Uses</summary>

- `by` — one or more sort columns
- `ascending` — list for mixed order (date asc, volume desc)
- `na_position` — `"first"` to surface missing data
- `kind` — `"mergesort"` = stable, keeps prior order on ties
- `ignore_index` — reset index to 0..N after sort

</details>

### sort_index
```python
df.sort_index(level=0, ascending=True)
```
<details>
<summary>Uses</summary>

- `level` — sort one MultiIndex level; needed before fast `.loc` slicing
- `ascending` — direction of the sort

</details>

---

## 9. merge_ordered / merge_asof

### merge_ordered
```python
pd.merge_ordered(l, r, on="dt", left_by="tkr", how="outer", fill_method="ffill", suffixes=("_l", "_r"))
```
<details>
<summary>Uses</summary>

- `on` — ordered key, usually a date
- `left_by` — do the ordered merge separately per group (per ticker)
- `how` — default `"outer"`: keep all dates from both sides
- `fill_method` — `"ffill"` carries last value into gaps (prices, rates)
- `suffixes` — rename clashing columns

</details>

### merge_asof
```python
pd.merge_asof(l, r, on="dt", by="tkr", direction="backward", tolerance=pd.Timedelta("1s"))
```
<details>
<summary>Uses</summary>

- `on` — nearest-key match; both sides must be sorted on it
- `by` — exact match on this column first (ticker)
- `direction` — `"backward"` = last known quote (no look-ahead bias)
- `tolerance` — ignore matches older than this (stale quotes)

</details>

---

## 10. Sampling

```python
df.sample(n=None, frac=0.1, replace=False, weights="w", random_state=42, axis=0)
```
<details>
<summary>Uses</summary>

- `n` / `frac` — fixed count vs fraction of rows
- `replace` — `True` for bootstrap resampling
- `weights` — weighted sample (e.g. by volume)
- `random_state` — reproducible results
- `axis` — `1` samples columns

</details>

---

## 11. Melt (wide → long)

```python
df.melt(id_vars=["id"], value_vars=["q1", "q2"], var_name="qtr", value_name="val")
```
<details>
<summary>Uses</summary>

- `id_vars` — columns kept as identifiers
- `value_vars` — columns to unpivot (default: all others)
- `var_name` — name for the column holding old column names
- `value_name` — name for the values column

</details>

---

## 12. Stack / Unstack

```python
df.stack(level=-1, future_stack=True); df.unstack(level=-1, fill_value=0)
```
<details>
<summary>Uses</summary>

- `stack` — columns → rows (wide → long via index)
- `unstack` — rows → columns
- `level` — which column/index level to move (default innermost)
- `future_stack` — pandas ≥ 2.1 new behavior; keeps NaNs, avoids warning
- `fill_value` — fill holes created by unstacking

</details>

---

## 13. Pivot

### pivot
```python
df.pivot(index="dt", columns="tkr", values="px")
```
<details>
<summary>Uses</summary>

- `index` / `columns` / `values` — long → wide reshape; raises on duplicate pairs

</details>

### pivot_table
```python
df.pivot_table(index="dt", columns="tkr", values="px", aggfunc="mean",
  fill_value=0, margins=True, observed=True)
```
<details>
<summary>Uses</summary>

- `aggfunc` — handles duplicates: `"mean"`, `"sum"`, `"last"`, list of fns
- `fill_value` — replace empty cells
- `margins` — adds `All` row/col totals (summary reports)
- `observed` — skip unused category combos

</details>

---

## 14. EDA

### info / describe
```python
df.info(memory_usage="deep"); df.describe(include="all", percentiles=[.05, .5, .95])
```
<details>
<summary>Uses</summary>

- `memory_usage="deep"` — true RAM size incl. strings
- `include="all"` — describe object/category columns too
- `percentiles` — tail checks (5% / 95%) for outliers

</details>

### missing / uniques / counts
```python
df.isna().sum(); df.nunique(); df.a.value_counts(normalize=True, dropna=False)
```
<details>
<summary>Uses</summary>

- `isna().sum()` — missing count per column
- `nunique()` — spot constants, IDs, low-cardinality columns
- `normalize` — proportions instead of counts
- `dropna=False` — count NaN as its own value

</details>

### correlation
```python
df.corr(method="pearson", numeric_only=True, min_periods=30); df.a.corr(df.b)
```
<details>
<summary>Uses</summary>

- `method` — `pearson` linear, `spearman` rank/monotonic, `kendall` small samples
- `numeric_only` — skip text columns (avoids errors)
- `min_periods` — require N overlapping obs, else NaN (sparse data)
- `Series.corr` — single pair, e.g. two return series

</details>

### cov / crosstab / skew
```python
df.cov(numeric_only=True); pd.crosstab(df.a, df.b, normalize="index"); df.skew(numeric_only=True)
```
<details>
<summary>Uses</summary>

- `cov` — covariance matrix (portfolio risk)
- `crosstab` + `normalize="index"` — category frequency table as row %
- `skew` — asymmetry of distributions (return tails)

</details>

---

## Gotchas

<details>
<summary>Show</summary>

- **Many-to-many merge** multiplies rows silently → use `validate=`
- **`pivot` with duplicate pairs** raises `ValueError` → use `pivot_table` + `aggfunc`
- **`groupby` drops NaN keys** by default → `dropna=False`

</details>
