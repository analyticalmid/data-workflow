---
type: llm
weight: 2
---
PASS if the response does all of the following:
- Rewrites `Margin %` with `DIVIDE(SUM(Sales[Margin]), SUM(Sales[Revenue]))`, explaining that `IFERROR` is slow and that returning 0 instead of BLANK misleads.
- Explains that `FILTER(ALL(Product), …)` removes every existing filter on `Product`, which is why the category slicer is ignored, and rewrites it as a column filter, e.g. `CALCULATE([Revenue], KEEPFILTERS(Product[Color] = "Red"))` (or `Product[Color] = "Red"` while explaining the difference).
- Rewrites `Revenue PY` to use the date table's column, `SAMEPERIODLASTYEAR('Date'[Date])`, explaining that time intelligence must use the marked date table rather than the fact's date column.

FAIL if any of the three stays unfixed.
