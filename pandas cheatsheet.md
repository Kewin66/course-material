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

## 15. Resample (time buckets)

### resample
```python
df.resample("1min", on="dt", label="right", closed="right", origin="start_day",
  offset=None)["px"].ohlc()
```
<details>
<summary>Uses</summary>

- rule — `"1s"`, `"1min"`, `"5min"`, `"1h"`, `"D"` day, `"B"` business day, `"W-FRI"` week ending Fri, `"ME"` month end, `"QE"`, `"YE"`
- `on` — resample on a datetime column; omit if the index is already a `DatetimeIndex`
- `label` — stamp bucket with its `"left"` (start) or `"right"` (end) edge; bar close = `"right"`
- `closed` — which edge is inclusive; `"right"` means 09:31 bar = (09:30, 09:31]
- `origin` — `"start_day"` (midnight), `"start"` (first timestamp), `"epoch"`: anchor for bucket edges
- `offset` — shift edges, e.g. `"30min"` so hourly bars align to 09:30 open
- `.ohlc()` — open/high/low/close bars from ticks
- old aliases `"T"`, `"M"`, `"Q"` are deprecated → use `"min"`, `"ME"`, `"QE"`

</details>

### resample + agg
```python
df.resample("D", on="dt").agg(px=("px", "last"), qty=("qty", "sum"), n=("px", "count"))
```
<details>
<summary>Uses</summary>

- `"last"` — daily close price
- `"sum"` — daily volume
- `"count"` — ticks per bucket; `0` flags an empty/illiquid bucket

</details>

### per group (per ticker)
```python
df.groupby(["tkr", pd.Grouper(key="dt", freq="1min", label="right", closed="right")])
  .agg(px=("px", "last"), qty=("qty", "sum"))
```
<details>
<summary>Uses</summary>

- `pd.Grouper(key, freq)` — time bucketing inside a normal `groupby`
- extra keys (`"tkr"`) — separate bars per instrument in one pass
- `label` / `closed` — same meaning as in `resample`

</details>

### upsample / fill gaps
```python
df.set_index("dt").resample("1s").ffill(limit=5); df.set_index("dt").asfreq("1min")
```
<details>
<summary>Uses</summary>

- `.ffill(limit=N)` — carry last price forward, but at most N buckets (avoid stale data)
- `.asfreq(freq)` — regular grid, NaN where no data (spot missing bars)
- `.interpolate(method="time")` — fill numeric gaps weighted by time

</details>

---

## 16. Cumulative

### cumsum / cumprod / cummax / cummin
```python
df.px.cummax(skipna=True); df.qty.cumsum(); (1 + df.ret).cumprod(); df.px.cummin()
```
<details>
<summary>Uses</summary>

- `cummax` — running high-water mark → drawdown `df.px / df.px.cummax() - 1`
- `cummin` — running low (best entry price so far)
- `cumsum` — running position / PnL
- `cumprod(1 + ret)` — compounded equity curve
- `skipna` — `False` makes NaN propagate forward
- `df.groupby("tkr").px.cummax()` — per-group running max

</details>

### cumcount
```python
df.groupby("user").cumcount(ascending=True)
```
<details>
<summary>Uses</summary>

- 0,1,2… row number inside each group (nth order per customer)
- `ascending=False` — count from the end (last = 0)

</details>

---

## 17. Shift / Diff / Pct Change

```python
df.px.shift(periods=1, fill_value=None); df.px.diff(periods=1); df.px.pct_change(periods=1, fill_method=None)
```
<details>
<summary>Uses</summary>

- `shift(1)` — previous row (lag); `shift(-1)` — next row (lead)
- `shift(freq="1D")` — shift the time index instead of the data
- `fill_value` — value for the new empty slot
- `diff` — day-over-day change
- `pct_change` — simple returns
- `fill_method=None` — no silent forward-fill (default fill is deprecated)
- `df.groupby("tkr").px.shift(1)` — lag per ticker (no leakage across tickers)

</details>

---

## 18. Rolling / Expanding / EWM

### rolling
```python
df.px.rolling(window=20, min_periods=20, center=False).mean(); df.rolling("5min", on="dt").px.mean()
```
<details>
<summary>Uses</summary>

- `window=20` — last 20 rows (moving average, rolling vol with `.std()`)
- `window="5min"` — time-based window; needs sorted datetime index/`on`
- `min_periods` — rows required before emitting a value (else NaN)
- `center` — centered window (smoothing, not for trading signals)
- `closed` — `"left"` excludes current row (no look-ahead)
- `.apply(fn, raw=True)` — custom function; `raw=True` passes NumPy (faster)
- `df.groupby("tkr").px.rolling(20).mean()` — per ticker

</details>

### expanding
```python
df.px.expanding(min_periods=1).max()
```
<details>
<summary>Uses</summary>

- all rows so far: expanding mean/std (running stats)
- `.max()` — same as `cummax`

</details>

### ewm
```python
df.px.ewm(span=20, halflife=None, alpha=None, adjust=False, min_periods=0).mean()
```
<details>
<summary>Uses</summary>

- `span` — EMA like trading platforms (`alpha = 2/(span+1)`)
- `halflife` — decay by time to half weight (volatility models)
- `alpha` — set smoothing factor directly
- `adjust=False` — recursive EMA formula (matches most charting tools)

</details>

---

## 19. Rank / Top-N

### rank
```python
df.salary.rank(method="dense", ascending=False, pct=False)
```
<details>
<summary>Uses</summary>

- `method="dense"` — 1,2,2,3 (SQL `DENSE_RANK`, "Nth highest salary")
- `"min"` — 1,2,2,4 (SQL `RANK`)
- `"first"` — 1,2,3,4 by order (SQL `ROW_NUMBER`)
- `pct=True` — percentile rank 0–1 (cross-sectional signals)
- `df.groupby("dept").salary.rank(...)` — rank within group

</details>

### nlargest / idxmax / head
```python
df.nlargest(3, "salary", keep="all"); df.px.idxmax(); df.sort_values("x").groupby("k").head(2)
```
<details>
<summary>Uses</summary>

- `nlargest(n, col)` — top-N without full sort; `nsmallest` for bottom
- `keep="all"` — include ties
- `idxmax` — label of max row → `df.loc[df.groupby("k").x.idxmax()]` = top row per group
- `groupby().head(n)` — top-N per group after sort
- `groupby().nth(0)` / `.first()` — first row per group (`first` skips NaN)

</details>

---

## 20. Duplicates and Missing

### duplicated / drop_duplicates
```python
df.duplicated(subset=["id"], keep="first"); df.drop_duplicates(subset=["id"], keep="last", ignore_index=True)
```
<details>
<summary>Uses</summary>

- `subset` — dedupe on key columns only
- `keep="last"` — keep latest record (after sorting by time)
- `keep=False` — mark/drop every duplicated row
- `duplicated` — boolean mask to inspect dups first

</details>

### fillna / ffill / dropna
```python
df.fillna({"qty": 0}); df.px.ffill(limit=3); df.bfill(); df.dropna(subset=["px"], how="any", thresh=None)
```
<details>
<summary>Uses</summary>

- `fillna(dict)` — per-column defaults
- `ffill(limit)` — carry last price, bounded staleness
- `bfill` — fill from next value (rarely valid for trading; look-ahead)
- `dropna(subset)` — drop rows missing key fields
- `how="all"` — drop only fully empty rows
- `thresh=N` — keep rows with at least N non-null values

</details>

---

## 21. Conditional / Transform Columns

```python
df.assign(side=np.where(df.qty > 0, "B", "S")); df.px.where(df.px > 0, other=np.nan); df.px.mask(df.px < 0, 0)
```
<details>
<summary>Uses</summary>

- `assign(new=...)` — add columns in a chain; `lambda d:` to use earlier results
- `np.where(cond, a, b)` — vectorized if/else
- `np.select([c1, c2], [v1, v2], default)` — multi-branch (SQL `CASE`)
- `where(cond, other)` — keep where True, replace elsewhere
- `mask(cond, other)` — replace where True (inverse of where)

</details>

```python
df.tkr.map({"A": "Tech"}); df.replace({"x": {"N/A": np.nan}}); df.apply(fn, axis=1); df.pipe(fn)
```
<details>
<summary>Uses</summary>

- `map(dict)` — lookup / recode values (unmatched → NaN)
- `replace` — targeted value substitution
- `apply(axis=1)` — row-wise custom logic; slow, last resort
- `pipe(fn)` — plug your own function into a method chain

</details>

---

## 22. Binning

```python
pd.cut(df.age, bins=[0, 18, 65, 120], labels=["kid", "adult", "senior"], right=False); pd.qcut(df.x, q=4, labels=False, duplicates="drop")
```
<details>
<summary>Uses</summary>

- `cut` — fixed edges (age groups, price bands)
- `right=False` — intervals `[a, b)`
- `labels` — names for bins; `False` returns bin numbers
- `qcut(q=4)` — equal-count buckets (quartiles, decile portfolios)
- `duplicates="drop"` — handle repeated edges in skewed data

</details>

---

## 23. Dtypes, Strings, Datetimes

### conversions
```python
pd.to_numeric(df.x, errors="coerce", downcast="float"); df.tkr.astype("category"); df.memory_usage(deep=True)
```
<details>
<summary>Uses</summary>

- `errors="coerce"` — bad strings → NaN instead of crash
- `downcast` — shrink to smallest safe type
- `category` — repeated strings (tickers) use far less memory, faster groupby
- `memory_usage(deep=True)` — find the heavy columns

</details>

### .str
```python
df.name.str.contains("abc", case=False, na=False, regex=True); df.s.str.split("_", expand=True); df.s.str.extract(r"(\d+)")
```
<details>
<summary>Uses</summary>

- `contains` — text filter; `na=False` avoids NaN in the mask
- `split(expand=True)` — split into columns
- `extract(regex)` — pull a capture group into a column
- `.str.strip() .lower() .replace()` — cleaning

</details>

### .dt / to_datetime
```python
pd.to_datetime(df.t, format="%Y-%m-%d %H:%M:%S", errors="coerce", utc=True); df.ts.dt.floor("1min"); df.ts.dt.tz_convert("America/New_York")
```
<details>
<summary>Uses</summary>

- `format` — explicit format is much faster and unambiguous
- `errors="coerce"` — bad dates → `NaT`
- `utc=True` — normalize mixed time zones
- `.dt.floor("1min")` — bucket key for manual resampling
- `.dt.date .dt.hour .dt.dayofweek` — feature extraction
- `.dt.tz_localize` (set) vs `.dt.tz_convert` (change) — time zones

</details>

---

## 24. Interview Patterns

### top-N per group
```python
df.sort_values("sal", ascending=False).groupby("dept").head(3)
```
<details>
<summary>Uses</summary>

- classic "top 3 earners per department"; alt: `rank(method="dense") <= 3`

</details>

### Nth highest
```python
df.sal.drop_duplicates().nlargest(2).iloc[-1]
```
<details>
<summary>Uses</summary>

- "second highest salary"; `drop_duplicates` handles ties

</details>

### max drawdown
```python
(df.px / df.px.cummax() - 1).min()
```
<details>
<summary>Uses</summary>

- worst peak-to-trough loss; `cummax` gives the running peak

</details>

### consecutive streaks (gaps and islands)
```python
g = (df.up != df.up.shift()).cumsum(); df.groupby(g).up.agg(["first", "size"])
```
<details>
<summary>Uses</summary>

- new group id every time the value changes → length of each run
- "longest winning streak", "consecutive login days"

</details>

### latest record per key
```python
df.sort_values("ts").drop_duplicates("id", keep="last")
```
<details>
<summary>Uses</summary>

- current state from an event log (latest order status)

</details>

### share of group total
```python
df.amt / df.groupby("k").amt.transform("sum")
```
<details>
<summary>Uses</summary>

- portfolio weights, % of department spend, without a merge

</details>

### first event after a condition
```python
df[df.groupby("id").flag.cumsum() > 0].groupby("id").head(1)
```
<details>
<summary>Uses</summary>

- first trade after signal, first purchase after signup

</details>

---

## Gotchas

<details>
<summary>Show</summary>

- **Chained assignment** `df[df.a > 0]["b"] = 1` may not write → use `df.loc[df.a > 0, "b"] = 1`
- **`shift`/`rolling` without `groupby`** leak values across tickers
- **Time-based `rolling("5min")`** needs a sorted datetime index / `on`
- **`apply(axis=1)`** is a Python loop → vectorize first
- **Resample with default `label`/`closed`** on minute bars stamps bars at the start (left); trading bars usually want `"right"` → set both explicitly
- **Many-to-many merge** multiplies rows silently → use `validate=`
- **`pivot` with duplicate pairs** raises `ValueError` → use `pivot_table` + `aggfunc`
- **`groupby` drops NaN keys** by default → `dropna=False`

</details>
