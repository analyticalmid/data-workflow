---
name: python-data
description: Data wrangling in Python with pandas and numpy — loading, cleaning, joining, reshaping, aggregating, dates and time zones, memory and speed. Use when writing or reviewing pandas or numpy code, when a DataFrame total does not match its source, or when a notebook is slow or runs out of memory.
---

# Python data wrangling

Code that changes data is judged by whether its numbers reconcile with the source, so every transformation
here is written to be checked.

## Steps

1. **Pin the versions.** Read `pd.__version__` and `np.__version__` from the environment or the lock file.
   pandas 3.x changed copy semantics, the string dtype, and datetime resolution; on 3.x, or when moving code from
   2.x, read [`references/pandas-3.md`](references/pandas-3.md). For array code, read
   [`references/numpy.md`](references/numpy.md). Done when both versions are named.
2. **Load lean.** Read only the columns needed (`usecols=` for CSV, `columns=` and `filters=` with the `pyarrow`
   engine for Parquet), with explicit `dtype=` for keys and codes, and parse dates on purpose. Done when no column
   was inferred by accident: a key with leading zeros is a string, a code is a `category`.
3. **Inspect before transforming.** `df.dtypes`, `df.memory_usage(deep=True)`, null share per column, and the
   uniqueness of every key you will join or group on (`df[key].duplicated().sum()`). Write the grain: one row
   per what. Done when the grain of every frame is written down.
4. **Transform** with the rules below.
5. **Reconcile.** After each step that can change rows, compare the row count and a control total (sum of an
   amount, distinct count of the key) with the step before. Done when every difference is explained.
6. **Persist** intermediate and final data as Parquet, which keeps dtypes, rather than CSV.

## Missing data

- The marker depends on the dtype: `np.nan` for NumPy floats, `NaT` for datetimes, `pd.NA` for nullable dtypes
  (`Int64`, `Float64`, `boolean`, `string`), `None` kept as is in `object`. An `int64` column that gains a missing
  value turns `float64`; ask for `Int64` when integers must stay integers, or call `convert_dtypes()` after
  reading.
- Test with `isna()`: `np.nan == np.nan` and `NaT == NaT` are `False`, and `pd.NA == pd.NA` is `<NA>`.
  `pd.NA` follows three-valued logic and `bool(pd.NA)` raises, so combine masks with `&`, `|`, `~`.
- Reductions skip missing values. `sum` of an empty or all-missing column is `0`; pass `min_count=1` when that
  must read as missing.
- `groupby` and `value_counts` drop missing keys by default (`dropna=True`), so the totals silently lose those
  rows: pass `dropna=False` and look at the missing group.
- `read_csv` treats `""`, `"NA"`, `"N/A"`, `"NULL"`, `"NaN"` and more as missing even with `dtype=`; pass
  `keep_default_na=False` and your own `na_values=` when one of those is a real value.

## Joins

- Every `merge` states its cardinality: `validate="m:1"` for facts onto a dimension, `"1:1"` or `"1:m"` as the
  grain says. A failed check raises `MergeError` before any memory is spent; duplicate keys on both sides would
  produce a cartesian product.
- `indicator=True` adds `_merge` (`left_only`, `right_only`, `both`): count it to see what failed to match.
- pandas matches missing keys to each other, unlike SQL: drop or fill null keys before the merge.
- Keys on both sides carry the same dtype; cast them explicitly first.
- `concat` defaults to an outer join of columns, filling the gaps with missing values; collect frames in a list
  and concatenate once, never inside a loop.

## Performance and memory

- Vectorize: column arithmetic, `.str`, `.dt`, `np.where`, `Series.map` with a dict. `apply` row-wise and
  `iterrows` are the last resort (`iterrows` also loses dtypes per row; `itertuples` is faster).
- Built-in groupby methods (`sum`, `mean`, `size`, `nunique`) beat a UDF; select the column before aggregating.
- Low-cardinality strings become `category`; numbers shrink with `pd.to_numeric(..., downcast=)`. Measure with
  `memory_usage(deep=True)` before and after.
- Larger than memory: load fewer columns, process in chunks (`chunksize=`), or move the work to the engine that
  holds the data (SQL, Spark).

## Dates and time zones

- `pd.to_datetime(..., format=...)` with an explicit format; `errors="coerce"` only when the resulting `NaT`s are
  counted. `dayfirst=True` is not strict, and `format="mixed"` guesses per row.
- `tz_localize` attaches a zone to naive times (wall clock kept); `tz_convert` moves aware times between zones.
  Mixed zones need `utc=True`. Daylight-saving gaps and overlaps raise until `ambiguous=` / `nonexistent=` say
  what to do.
- `resample` closes and labels on the left, except month, quarter, year, and week ends, which default to the
  right: set `closed=` and `label=` when bins meet a report boundary.

## Aggregation and reshape

- Named aggregation keeps output columns explicit: `df.groupby("region").agg(revenue=("amount", "sum"))`.
  `transform` returns one value per input row, `agg` one per group.
- `pivot` raises on duplicate index and column pairs; `pivot_table` aggregates them, with `aggfunc="mean"` by
  default, so state the `aggfunc` every time.
- `explode` turns an empty list into a missing value and repeats the index; `melt` drops the index unless
  `ignore_index=False`.

## Reproducibility

Random draws come from `np.random.default_rng(seed)` passed down explicitly; versions are pinned in the lock file;
a notebook runs top to bottom on a fresh kernel before its numbers are reported.
