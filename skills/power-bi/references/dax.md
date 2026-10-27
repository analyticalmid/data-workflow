# DAX

## Contexts

- **Row context** is the current row, created by calculated columns and iterators (`SUMX`, `FILTER`, `ADDCOLUMNS`).
  **Filter context** is the set of values allowed in each column, created by visuals, slicers, and `CALCULATE`.
  https://learn.microsoft.com/en-us/dax/dax-overview
- **Context transition**: `CALCULATE` turns the current row context into a filter context, and so does any measure
  referenced inside an iterator. It filters by every column of the row, so it relies on rows being unique.
  https://www.sqlbi.com/articles/understanding-context-transition/
- **Measures, not calculated columns**, for anything aggregated: a calculated column is computed at refresh and stored;
  a measure is computed in the visual's context.

## CALCULATE

- A filter argument replaces the existing filter on the same columns; wrap it in `KEEPFILTERS` to intersect instead.
- Filter on a column, not a table: `CALCULATE([Revenue], KEEPFILTERS(Product[Color] = "Red"))`, never
  `FILTER(ALL(Product), …)`, which scans the table and silently drops the user's other product filters. `FILTER` is for
  conditions on measures, across columns, or with `OR`.
  https://learn.microsoft.com/en-us/dax/best-practices/dax-avoid-avoid-filter-as-filter-argument
- Clear filters with `REMOVEFILTERS` (or `ALL` / `ALLEXCEPT` when a table is needed as a value).
  https://learn.microsoft.com/en-us/dax/calculate-function-dax

## Writing measures

- **`DIVIDE(numerator, denominator)`** when the denominator can be zero or blank; `/` only for a constant. Return BLANK,
  not 0, so empty rows disappear. https://learn.microsoft.com/en-us/dax/best-practices/dax-divide-function-operator
- **No `IFERROR` / `ISERROR`**: they add scans and hide data problems; fix the data in Power Query and use `IF` or
  `DIVIDE`. https://learn.microsoft.com/en-us/dax/best-practices/dax-error-functions
- **Variables** (`VAR … RETURN`) evaluate once, in the context where they are defined, outside the filters of the
  `RETURN`; they replace `EARLIER`. https://learn.microsoft.com/en-us/dax/best-practices/dax-variables
- **Iterators**: `SUM(col)` is `SUMX` over one column; `SUMX` is for row-level expressions (`quantity * price`).
  https://dax.guide/sumx/
- **Semi-additive facts** (balances, stock): the last value in the period, e.g. `LASTNONBLANKVALUE` or the closing
  date's value, never `SUM` across dates.

## Time intelligence

Classic functions (`SAMEPERIODLASTYEAR`, `DATESYTD`, `TOTALYTD`) need a marked date table with no missing dates and
take its date column, never a fact's date column. Calendar-based time intelligence (preview) handles fiscal and week
calendars. https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-time-intelligence

## Formatting

- Columns always with their table (`Sales[Amount]`); measures never (`[Revenue]`).
- One argument per line once there are two or more, indented four spaces, the closing parenthesis aligned with the
  function; a space before parentheses and around operators; line breaks before an operator.
- https://www.sqlbi.com/articles/rules-for-dax-code-formatting/
