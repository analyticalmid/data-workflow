# Power Query M

## Query folding

- Folding turns the steps into one source query; the result is full, partial, or none. CSV and Excel never fold.
  Check with View Native Query (or View data source query), or the step folding indicators in Power Query Online:
  every step up to the last folding indicator folds. https://learn.microsoft.com/en-us/power-query/query-folding-basics ·
  https://learn.microsoft.com/en-us/power-query/step-folding-indicators
- Order steps so the folding ones come first: filters, column removal, joins, and grouping at the top; steps that do
  not fold (custom text functions such as `Text.Proper`, index columns, many custom columns) at the end.
- DirectQuery and Dual tables must fold completely. Steps after a native SQL query never fold; pass parameters with
  `Value.NativeQuery` under a read-only account. https://learn.microsoft.com/en-us/power-bi/guidance/power-query-folding

## Structure

- Filter early, sort last, and keep the first rows while developing.
- Set the correct data type on every column.
- Split long queries into referenced staging queries, with load disabled on the helpers.
- Parameters for server and database names, and custom functions for repeated logic.
- https://learn.microsoft.com/en-us/power-query/best-practices

## Incremental refresh

- Filter on the reserved datetime parameters `RangeStart` and `RangeEnd` (case-sensitive) as `>= RangeStart` and
  `< RangeEnd`, so no row lands in two partitions.
- The filter must fold, or every refresh reads the whole source.
- https://learn.microsoft.com/en-us/power-bi/connect-data/incremental-refresh-overview

## Privacy and errors

- Privacy levels (Private, Organizational, Public) keep one source's data from reaching another and can block
  folding; "Ignore the Privacy Levels" can leak data and is not honoured after publishing.
  https://learn.microsoft.com/en-us/power-query/privacy-levels
- `try … otherwise …` for a fallback, `try … catch (error) => …` to read the error, and a bare `try` returning a record
  with `HasError`, `Value`, and `Error`. Count the rows that fell into the fallback instead of hiding them.
  https://learn.microsoft.com/en-us/power-query/error-handling
