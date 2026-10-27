---
type: llm
weight: 2
---
PASS if the response identifies at least three of these four pandas 3 changes and fixes each one it identifies:
- Parsed datetimes default to microsecond resolution, so `astype("int64")` yields values 1000 times smaller than nanoseconds; fix with `.dt.as_unit("ns")` before the cast (or an equivalent explicit conversion).
- Text columns now infer the `str` dtype, so `dtype == object` no longer selects them and nothing gets stripped; fix with `pd.api.types.is_string_dtype` or `select_dtypes` including strings.
- `df["status"].replace(..., inplace=True)` modifies a copy under Copy-on-Write and leaves `df` unchanged; fix by assigning the result back.
- The `"M"` offset alias was removed in favour of `"ME"`.

FAIL if it identifies fewer than three, or claims the original code is fine on pandas 3.1.
