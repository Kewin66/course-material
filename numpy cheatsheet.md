# NumPy Cheatsheet

```python
import numpy as np
```

---

## 1. Create Arrays

### from data / filled
```python
np.array([1, 2, 3], dtype=np.float64); np.zeros((3, 4)); np.ones(5); np.full((2, 2), 7); np.empty(10)
```
<details>
<summary>Uses</summary>

- `dtype` — set precision up front (`float32` halves memory)
- `zeros` / `ones` — init accumulators, masks
- `full` — constant fill (e.g. `np.nan` placeholder)
- `empty` — fastest; values are garbage until you write them

</details>

### ranges / identity
```python
np.arange(0, 10, 2); np.linspace(0, 1, 5, endpoint=True); np.eye(3)
```
<details>
<summary>Uses</summary>

- `arange(start, stop, step)` — integer steps, stop excluded
- `linspace(start, stop, num)` — N evenly spaced points; safe for floats
- `endpoint=False` — exclude stop
- `eye` — identity matrix

</details>

---

## 2. Random

```python
rng = np.random.default_rng(seed=42); rng.normal(loc=0, scale=1, size=(1000,))
```
<details>
<summary>Uses</summary>

- `default_rng(seed)` — modern, reproducible generator
- `normal(loc, scale, size)` — simulated returns / noise
- `rng.uniform(low, high, size)` — uniform draws
- `rng.integers(low, high, size, endpoint=False)` — random ints
- `rng.choice(a, size, replace=False, p=None)` — sample; `p` = weights
- `rng.permutation(x)` — shuffled copy (train/test split)

</details>

---

## 3. Dtype and Shape

### dtype / astype
```python
a.dtype; a.astype(np.float32, copy=False)
```
<details>
<summary>Uses</summary>

- `astype` — convert type (downcast to save RAM)
- `copy=False` — avoid a copy if already that type

</details>

### shape / reshape
```python
a.shape; a.ndim; a.size; a.reshape(2, -1); a.ravel(); a.T; a[:, None]; np.squeeze(a)
```
<details>
<summary>Uses</summary>

- `reshape(2, -1)` — `-1` lets NumPy infer that dimension
- `ravel` — flatten (view when possible); `flatten` always copies
- `.T` — transpose
- `a[:, None]` / `np.expand_dims(a, 1)` — add axis for broadcasting
- `squeeze` — drop size-1 axes

</details>

---

## 4. Indexing

### slice / mask / fancy
```python
a[1:3, ::2]; a[a > 0]; a[[0, 2]]; a[..., -1]
```
<details>
<summary>Uses</summary>

- slice `start:stop:step` — returns a **view** (no copy)
- boolean mask — filter values; returns a copy
- fancy `[[0, 2]]` — pick specific rows; returns a copy
- `...` — all leading axes (last column of any-D array)

</details>

### where / nonzero
```python
np.where(a > 0, a, 0); np.nonzero(a); np.argwhere(a > 0)
```
<details>
<summary>Uses</summary>

- `where(cond, x, y)` — vectorized if/else (clip negatives to 0)
- `nonzero` — indices of true / non-zero values
- `argwhere` — same, as (row, col) pairs

</details>

---

## 5. Combine / Split

```python
np.concatenate([a, b], axis=0); np.stack([a, b], axis=0); np.vstack([a, b]); np.hstack([a, b])
```
<details>
<summary>Uses</summary>

- `concatenate` — join along an existing axis
- `stack` — join along a **new** axis (list of 1-D → 2-D)
- `vstack` / `hstack` — rows / columns shortcuts
- `np.column_stack` — 1-D arrays as columns

</details>

```python
np.split(a, 3, axis=0); np.array_split(a, 4)
```
<details>
<summary>Uses</summary>

- `split` — equal parts (errors if not divisible)
- `array_split` — allows unequal parts (batching)

</details>

---

## 6. Aggregations

```python
a.sum(axis=0, keepdims=True); a.mean(axis=1); a.std(ddof=1); a.argmax(axis=0)
```
<details>
<summary>Uses</summary>

- `axis=0` — down columns; `axis=1` — across rows
- `keepdims` — keep shape for broadcasting (`a / a.sum(1, keepdims=True)`)
- `ddof=1` — sample std (matches pandas); default is `0`
- `argmax` / `argmin` — index of best / worst
- `np.nanmean` / `np.nanstd` / `np.nansum` — ignore NaN

</details>

### cumulative / diff
```python
np.cumsum(a, axis=0); np.cumprod(1 + r); np.diff(a, n=1, axis=-1, prepend=a[0])
```
<details>
<summary>Uses</summary>

- `cumsum` — running PnL
- `cumprod(1 + r)` — compounded equity curve
- `diff` — changes between elements; `prepend` keeps the same length

</details>

---

## 7. Sort / Search / Unique

```python
np.sort(a, axis=-1, kind="stable"); np.argsort(a); np.searchsorted(s, v, side="left")
```
<details>
<summary>Uses</summary>

- `sort` — sorted copy; `a.sort()` sorts in place
- `kind="stable"` — keep tie order
- `argsort` — ranking / reorder another array by this one
- `searchsorted` — binary search insert position in a sorted array (bucket lookup)
- `side="right"` — insert after equal values

</details>

```python
np.unique(a, return_counts=True, return_index=True, return_inverse=True); np.partition(a, -k)[-k:]; np.isin(a, b)
```
<details>
<summary>Uses</summary>

- `return_counts` — frequency table
- `return_index` — first occurrence positions
- `return_inverse` — map each value to its unique id (label encoding)
- `partition` — top-k in O(n) without full sort
- `isin` — membership mask

</details>

---

## 8. Elementwise Math

```python
np.log(a); np.log1p(r); np.exp(a); np.sqrt(a); np.abs(a); np.clip(a, -3, 3); np.round(a, 2)
```
<details>
<summary>Uses</summary>

- `log` — log prices; `np.diff(np.log(p))` = log returns
- `log1p` — accurate `log(1 + x)` for small returns
- `clip(lo, hi)` — winsorize outliers
- `round(decimals)` — tick/price rounding
- `np.maximum(a, b)` — elementwise max of two arrays (`np.max` = reduction)

</details>

---

## 9. Broadcasting

```python
(a - a.mean(axis=0)) / a.std(axis=0)
```
<details>
<summary>Uses</summary>

- column-wise z-score with no loops
- rule: shapes align from the right; each dim must match or be 1
- `a[:, None] - b[None, :]` — all-pairs difference matrix

</details>

---

## 10. Linear Algebra

```python
a @ b; np.linalg.solve(A, y); np.linalg.inv(A); np.linalg.eigh(S); np.linalg.norm(a, ord=2, axis=1)
```
<details>
<summary>Uses</summary>

- `@` / `np.matmul` — matrix multiply (portfolio `w @ cov @ w`)
- `solve(A, y)` — solve Ax = y; faster and more stable than `inv(A) @ y`
- `inv` — explicit inverse (only if you really need it)
- `eigh` — eigen-decomposition of symmetric matrices (PCA on covariance)
- `norm(ord, axis)` — vector/row lengths

</details>

```python
np.linalg.lstsq(X, y, rcond=None); np.linalg.cholesky(S); np.einsum("ij,jk->ik", a, b)
```
<details>
<summary>Uses</summary>

- `lstsq` — linear regression / OLS betas
- `cholesky` — correlated random draws: `L @ z`
- `einsum` — explicit index notation for complex tensor ops

</details>

---

## 11. Stats / EDA

```python
np.percentile(a, [5, 50, 95], axis=0, method="linear"); np.quantile(a, 0.99)
```
<details>
<summary>Uses</summary>

- `percentile` (0–100) / `quantile` (0–1) — tails, VaR
- `method` — interpolation rule (`"linear"`, `"lower"`, `"nearest"`)

</details>

```python
np.corrcoef(x, y); np.cov(X, rowvar=False, ddof=1); np.histogram(a, bins=20, density=False)
```
<details>
<summary>Uses</summary>

- `corrcoef` — correlation matrix
- `rowvar=False` — columns are variables (usual table layout)
- `ddof=1` — sample covariance
- `histogram` — counts + bin edges; `density=True` for a PDF

</details>

```python
np.isnan(a).sum(); np.isclose(a, b, rtol=1e-5, atol=1e-8); np.allclose(a, b)
```
<details>
<summary>Uses</summary>

- `isnan().sum()` — count missing
- `isclose` / `allclose` — safe float comparison (never `==` on floats)

</details>

---

## 12. Rolling Window

```python
np.lib.stride_tricks.sliding_window_view(a, window_shape=20).mean(axis=-1)
```
<details>
<summary>Uses</summary>

- zero-copy windows of length N → moving average / std
- output length is `len(a) - N + 1`
- `axis` — which axis to slide along for 2-D arrays

</details>

---

## 13. Datetime (time buckets)

```python
t = np.datetime64("2026-09-23T09:30"); t + np.timedelta64(1, "m"); ts.astype("datetime64[m]")
```
<details>
<summary>Uses</summary>

- `datetime64` — vectorized timestamps
- `timedelta64(1, "m")` — minute step; `"s"`, `"h"`, `"D"`
- `.astype("datetime64[m]")` — floor to minute (bucket key); `"datetime64[D]"` → day
- `np.arange(start, stop, np.timedelta64(1, "m"))` — regular minute grid
- `np.busday_count(d1, d2)` — business days between dates

</details>

---

## 14. Save / Load

```python
np.save("a.npy", a); np.load("a.npy", mmap_mode="r"); np.savez_compressed("f.npz", a=a, b=b)
```
<details>
<summary>Uses</summary>

- `save` / `load` — fast binary single array
- `mmap_mode="r"` — read slices of a huge file without loading it all
- `savez_compressed` — several named arrays in one file
- `np.loadtxt(path, delimiter=",", skiprows=1)` — simple CSV of numbers

</details>

---

## Gotchas

<details>
<summary>Show</summary>

- **Slices are views**: editing `b = a[1:3]` changes `a` → use `.copy()`
- **`std` default `ddof=0`** vs pandas `ddof=1` → numbers differ
- **Integer overflow is silent** (`int32` sums) → cast to `int64` / `float64`
- **NaN poisons reductions** → use `nan*` functions
- **Never `==` on floats** → `np.isclose`

</details>
