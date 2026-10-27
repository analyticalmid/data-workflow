# pandas 3.x

pandas 3.0 (2026-01) and 3.1 (2026-10) change behaviour that 2.x code relies on, often without an error. Sources:
https://pandas.pydata.org/docs/whatsnew/v3.0.0.html (whatsnew), https://pandas.pydata.org/docs/user_guide/migration.html
(migration), https://pandas.pydata.org/docs/whatsnew/v3.1.0.html.

## Copy-on-Write is the only mode

- Any indexing result, a column taken as a Series included, behaves as a copy. Chained assignment
  (`df[mask]["col"] = value`) never writes to `df`; it warns with `ChainedAssignmentError`. Write
  `df.loc[mask, "col"] = value`.
- An in-place method on a selected column (`df["col"].replace(1, 5, inplace=True)`) leaves `df` unchanged. Assign
  the result back: `df["col"] = df["col"].replace(1, 5)`.
- `SettingWithCopyWarning` is gone, and so is the need for defensive `.copy()` calls.
- `to_numpy()` returns a read-only array when it shares memory with the frame.
- The `copy=` keyword of `merge`, `reindex`, `astype`, `rename` and others has no effect and is deprecated.
- In-place methods (`fillna`, `replace`, `where`, `clip`) return `self` instead of `None`; 3.1 deprecates
  `inplace=` on `drop`, `rename`, `reset_index`, `set_index`, and `sort_values`.

## The default string dtype is `str`

- Text columns infer `str`, backed by PyArrow when installed, else NumPy `object`. It holds only strings and
  missing values (always `np.nan`); assigning a non-string fails.
- `ser.dtype == object` is now `False` for text: test with `pd.api.types.is_string_dtype(ser.dtype)`, which works
  on 2.x and 3.x.
- `astype("str")` keeps missing values instead of writing the text `"nan"`; `ser.map(str)` restores the old result.
- `Series.values` on text returns an ExtensionArray; use `to_numpy()`.

## Datetime resolution

- Parsed strings default to microseconds (`us`) instead of nanoseconds; 3.1 does the same for numeric input to
  `to_datetime`, `to_timedelta`, `Timestamp`, and `Timedelta`. `date_range` no longer defaults to `ns`.
- `astype("int64")` on `us` data yields numbers 1000 times smaller than the old `ns` epoch values: call
  `.dt.as_unit("ns")` first when a consumer expects nanoseconds.
- Time zones are `zoneinfo`, not `pytz`; ambiguous or nonexistent local times raise `ValueError`.
- `offsets.Day` is a calendar day, not 24 hours, across daylight-saving changes.

## Removed

- Offset aliases `M`, `Q`, `Y`, `BM`, `BQ`, `BY`, `SM`: use `ME`, `QE`, `YE`, `BME`, `BQE`, `BYE`, `SME`.
- `DataFrame.applymap` (use `map`), `fillna(method=)` (use `ffill` / `bfill`), `axis=` on groupby, rolling and
  resample, `errors="ignore"` in `to_datetime` / `to_numeric` / `to_timedelta`, `infer_datetime_format`,
  `date_parser`, `delim_whitespace`.

## Other changes that move numbers

- `groupby(..., observed=)` and `pivot_table(..., observed=)` default to `True`: unobserved categories no longer
  appear as empty groups.
- Nullable dtypes return `<NA>` where they returned `NaN` (for example `ser / 0`).
- `merge` gained `how="left_anti"` and `"right_anti"`.
- `concat(..., union_categories=True)` (3.1) keeps a categorical when categories differ.
- 3.1 removes the numexpr engine (`engine="numexpr"` raises `ImportError`) and deprecates `engine="fastparquet"`.
- pandas 3 needs Python 3.11+; 3.1 needs NumPy 2.0.2+.
