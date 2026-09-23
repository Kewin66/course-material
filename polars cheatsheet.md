# Polars Cheatsheet

```python
import polars as pl
```

---

## 1. Read / Write

### read_csv / scan_csv
```python
pl.read_csv(path, separator=",", has_header=True, columns=["a"], schema_overrides={"a": pl.Int32},
  try_parse_dates=True, null_values=["NA"], n_rows=N, skip_rows=N)
```
<details>
<summary>Uses</summary>

- `separator` — `"\t"` for TSV, `"|"` for pipe files
- `has_header` — `False` when the file has no header row
- `columns` — load only needed columns
- `schema_overrides` — force types (IDs as `pl.Utf8`, downcast to `pl.Int32`)
- `try_parse_dates` — auto-parse date/datetime strings
- `null_values` — treat `"NA"`, `"-"` as null
- `n_rows` — peek at a huge file
- `skip_rows` — skip banner lines
- `pl.scan_csv(...)` — lazy version: nothing is read until `.collect()`, filters pushed down

</details>

### parquet / json
```python
pl.read_parquet(path, columns=["a"]); pl.scan_parquet(path); pl.read_json(path); pl.read_ndjson(path)
```
<details>
<summary>Uses</summary>

- `read_parquet` + `columns` — fast columnar load of only needed columns
- `scan_parquet` — lazy; reads only row groups/columns the query needs
- `read_json` — JSON array of records
- `read_ndjson` — JSONL / log files (one object per line)

</details>

### write / convert
```python
df.write_parquet(path, compression="zstd"); df.write_csv(path); df.to_pandas(); pl.from_pandas(pdf)
```
<details>
<summary>Uses</summary>

- `compression="zstd"` — small files, fast reads
- `write_csv` — share with tools that need CSV
- `to_pandas` / `from_pandas` — interop with pandas-only libraries
- `df.to_numpy()` — hand off to NumPy / ML code

</details>

---

## 2. Select Columns

### select
```python
df.select(pl.col("a"), pl.col("^c_.*$"), pl.col(pl.Float64), pl.exclude("b"))
```
<details>
<summary>Uses</summary>

- `pl.col("a")` — one column by name
- `pl.col("^c_.*$")` — regex (must start `^` and end `$`)
- `pl.col(pl.Float64)` — all columns of a dtype
- `pl.exclude("b")` — everything except
- `pl.all()` — every column (apply one op to all)

</details>

---

## 3. Filter Rows

### filter
```python
df.filter((pl.col("a") > 5) & pl.col("tkr").is_in(["X", "Y"]) & pl.col("px").is_between(1, 9))
```
<details>
<summary>Uses</summary>

- `& | ~` — AND / OR / NOT; wrap comparisons in `()`
- `is_in` — value in a list (tickers)
- `is_between` — inclusive range (dates, prices)
- `is_null()` / `is_not_null()` — missing data checks
- `df.head(n)` / `df.tail(n)` / `df.slice(offset, len)` — positional rows

</details>

---

## 4. Add / Modify Columns

### with_columns
```python
df.with_columns((pl.col("a") * 2).alias("a2"), ret=pl.col("px").pct_change().over("tkr"))
```
<details>
<summary>Uses</summary>

- `.alias("name")` — name the new column
- `name=expr` — keyword form, same as alias
- `.over("tkr")` — window per group, keeps row count (pandas `transform`)
- `.cast(pl.Float32)` — change dtype
- `.shift(1)` / `.diff()` / `.pct_change()` — lag, change, returns

</details>

### when / then / otherwise
```python
df.with_columns(side=pl.when(pl.col("qty") > 0).then(pl.lit("B")).otherwise(pl.lit("S")))
```
<details>
<summary>Uses</summary>

- `when().then()` — vectorized if/else
- chain `.when()` again — multiple branches (like SQL `CASE`)
- `pl.lit(x)` — constant value

</details>

---

## 5. Group By and Agg

### group_by + agg
```python
df.group_by("k", maintain_order=True).agg(pl.col("amt").sum().alias("tot"),
  pl.col("id").n_unique().alias("n"), pl.len().alias("rows"))
```
<details>
<summary>Uses</summary>

- `maintain_order` — keep first-seen group order (off by default, faster)
- `.sum() .mean() .max() .first() .last()` — standard aggs
- `.n_unique()` — distinct count
- `pl.len()` — rows per group
- `pl.col("px").filter(cond).mean()` — conditional agg inside a group

</details>

---

## 6. Join

### join
```python
df.join(r, on="k", how="left", left_on=None, right_on=None, suffix="_r", validate="1:1", coalesce=True)
```
<details>
<summary>Uses</summary>

- `how` — `inner`, `left`, `right`, `full`, `cross`, `semi` (keep matches), `anti` (keep non-matches)
- `on` — same key name
- `left_on` / `right_on` — different key names
- `suffix` — rename clashing right-side columns
- `validate` — `"1:1"`, `"1:m"`, `"m:1"`: raise on unexpected duplicates
- `coalesce` — merge key columns into one (for `full` joins)

</details>

### join_asof
```python
df.join_asof(r, on="dt", by="tkr", strategy="backward", tolerance="1s")
```
<details>
<summary>Uses</summary>

- `on` — nearest match on a sorted key (both frames sorted)
- `by` — exact match on this first (ticker)
- `strategy` — `"backward"` last known value (no look-ahead), `"forward"`, `"nearest"`
- `tolerance` — max gap, e.g. `"1s"`; ignore stale quotes

</details>

---

## 7. Concat

```python
pl.concat([d1, d2], how="vertical", rechunk=True)
```
<details>
<summary>Uses</summary>

- `how="vertical"` — stack rows, same schema
- `"vertical_relaxed"` — stack rows, upcast mismatched types
- `"diagonal"` — stack rows, missing columns filled with null
- `"horizontal"` — side by side, same height
- `rechunk` — contiguous memory after concat (faster later ops)

</details>

---

## 8. Sort / Unique / Sample

### sort
```python
df.sort(["a", "b"], descending=[False, True], nulls_last=True)
```
<details>
<summary>Uses</summary>

- `descending` — list for mixed order
- `nulls_last` — push missing values to the end
- `maintain_order=True` — stable sort

</details>

### unique
```python
df.unique(subset=["a"], keep="first", maintain_order=True)
```
<details>
<summary>Uses</summary>

- `subset` — dedupe on these columns only
- `keep` — `"first"`, `"last"`, `"any"`, `"none"` (drop all dups)
- `maintain_order` — keep original row order

</details>

### sample
```python
df.sample(n=None, fraction=0.1, with_replacement=False, shuffle=True, seed=42)
```
<details>
<summary>Uses</summary>

- `n` / `fraction` — count vs fraction
- `with_replacement` — bootstrap
- `seed` — reproducible

</details>

---

## 9. Reshape

### unpivot (was melt)
```python
df.unpivot(index=["id"], on=["q1", "q2"], variable_name="qtr", value_name="val")
```
<details>
<summary>Uses</summary>

- `index` — identifier columns kept
- `on` — columns to unpivot (wide → long)
- `variable_name` / `value_name` — output column names

</details>

### pivot
```python
df.pivot(on="tkr", index="dt", values="px", aggregate_function="mean")
```
<details>
<summary>Uses</summary>

- `on` — values become new column names
- `index` — rows
- `values` — cell values
- `aggregate_function` — `"first"`, `"last"`, `"sum"`, `"mean"`, `"len"`; required if duplicates

</details>

### explode
```python
df.explode("fills")
```
<details>
<summary>Uses</summary>

- list column → one row per element (e.g. order with list of fills)

</details>

---

## 10. Resample (time buckets)

### group_by_dynamic
```python
df.sort("dt").group_by_dynamic("dt", every="1m", closed="right", label="right", group_by="tkr")
  .agg(o=pl.col("px").first(), h=pl.col("px").max(), l=pl.col("px").min(), c=pl.col("px").last())
```
<details>
<summary>Uses</summary>

- `every` — `"1s"`, `"1m"` minute, `"5m"`, `"1h"`, `"1d"` day, `"1w"`, `"1mo"` month, `"1q"`, `"1y"`
- `period` — window length if different from `every` (overlapping windows)
- `offset` — shift edges, e.g. `"30m"` to align to 09:30
- `closed` / `label` — same meaning as pandas; `"right"` for bar-close stamps
- `group_by` — separate bars per ticker
- first/max/min/last — OHLC bars
- data must be sorted by `"dt"`

</details>

### upsample / fill gaps
```python
df.upsample(time_column="dt", every="1m", group_by="tkr").with_columns(pl.col("px").forward_fill(limit=5))
```
<details>
<summary>Uses</summary>

- `upsample` — insert missing timestamps on a regular grid
- `forward_fill(limit=N)` — carry last value, max N steps

</details>

### rolling
```python
df.with_columns(ma=pl.col("px").rolling_mean(window_size=20).over("tkr"))
```
<details>
<summary>Uses</summary>

- `rolling_mean/std/max/min/sum` — moving stats over N rows
- `min_samples` (older: `min_periods`) — rows needed before emitting a value
- `.over("tkr")` — per instrument
- `df.rolling("dt", period="5m")` — time-based window instead of row count

</details>

---

## 11. Lazy

```python
df.lazy().filter(pl.col("a") > 5).group_by("k").agg(pl.col("x").sum()).collect()
```
<details>
<summary>Uses</summary>

- `.lazy()` / `scan_*` — build a query plan; optimizer pushes filters & column selection down
- `.collect()` — execute
- `.explain()` — print optimized plan
- `.sink_parquet(path)` — stream result to disk without loading it all

</details>

---

## 12. EDA

### describe / schema / nulls
```python
df.describe(percentiles=(0.05, 0.5, 0.95)); df.schema; df.null_count(); df.estimated_size("mb")
```
<details>
<summary>Uses</summary>

- `describe` + `percentiles` — summary with tail checks
- `schema` — column names and dtypes
- `null_count` — missing per column
- `estimated_size("mb")` — memory footprint

</details>

### counts / uniques
```python
df["a"].value_counts(sort=True, normalize=True); df.select(pl.all().n_unique())
```
<details>
<summary>Uses</summary>

- `value_counts` + `normalize` — frequency as proportions
- `n_unique` on all — spot constants / IDs

</details>

### correlation
```python
df.select(pl.corr("a", "b", method="spearman")); df.select(pl.col(pl.Float64)).corr()
```
<details>
<summary>Uses</summary>

- `pl.corr(a, b, method)` — one pair; `"pearson"` or `"spearman"`
- `df.corr()` — full Pearson matrix (numeric columns only)
- `pl.cov(a, b)` — covariance

</details>

---

## Gotchas

<details>
<summary>Show</summary>

- **`"1m"` is minute, `"1mo"` is month** in duration strings
- **`group_by` does not keep order** unless `maintain_order=True`
- **`join_asof` / `group_by_dynamic` need sorted keys** → `.sort("dt")` first
- **No index** in Polars: use columns for keys; `with_row_index()` if you need row numbers
- **Wrap each comparison in `()`** when combining with `&` / `|`

</details>
