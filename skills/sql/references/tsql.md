# T-SQL

SQL Server, Azure SQL, and Fabric Warehouse. Fabric differences are under [Fabric Warehouse](#fabric-warehouse).

## Reading the plan

- Read the actual execution plan, with `SET STATISTICS IO ON` for scan count and logical (cache) vs
  physical (disk) reads per table; LOB columns inflate logical reads.
  https://learn.microsoft.com/en-us/sql/t-sql/statements/set-statistics-io-transact-sql
- For window functions, an index keyed on the `PARTITION BY` columns then the `ORDER BY` columns removes the
  sort; stale statistics on them cause sort spills.
  https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql

## Traps

- **Implicit conversion**: the lower-precedence type converts to the higher one (`int` beats every string,
  `nvarchar` beats `varchar`). A `varchar` column compared with an `nvarchar` parameter is converted, which
  can turn a seek into a scan: match the parameter type to the column.
  https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-precedence-transact-sql
- **`AVG` of an integer column returns `int`**: the average is truncated. Cast the input:
  `AVG(CAST(qty AS decimal(18, 4)))`. https://learn.microsoft.com/en-us/sql/t-sql/functions/avg-transact-sql
- **`COUNT` returns `int`** and fails past 2,147,483,647 (error 8115): use `COUNT_BIG`.
  https://learn.microsoft.com/en-us/sql/t-sql/functions/count-transact-sql
- **Integer division** truncates. https://learn.microsoft.com/en-us/sql/t-sql/language-elements/divide-transact-sql
- **`datetime`** rounds to .000/.003/.007 s, so `'… 23:59:59.999'` is the next day: use `datetime2`, `date`,
  or `datetimeoffset`, and half-open ranges.
  https://learn.microsoft.com/en-us/sql/t-sql/data-types/datetime-transact-sql
- **`AT TIME ZONE`**: an input without offset is taken as already in that zone; DST gaps and overlaps resolve
  to fixed offsets. https://learn.microsoft.com/en-us/sql/t-sql/queries/at-time-zone-transact-sql
- **Order**: `TOP` and `OFFSET … FETCH` need a unique `ORDER BY`; an `ORDER BY` in a view, derived table, or
  `INSERT … SELECT` orders nothing. https://learn.microsoft.com/en-us/sql/t-sql/queries/select-order-by-clause-transact-sql
- **`NOT IN` with a NULL** in the list or subquery yields unknown; long literal `IN` lists raise errors
  8623/8632, so load them into a table. https://learn.microsoft.com/en-us/sql/t-sql/language-elements/in-transact-sql
- **`NOLOCK` / `READ UNCOMMITTED`** reads uncommitted data and can return rows twice or skip them: read
  consistent data with RCSI or `SNAPSHOT` isolation.
  https://learn.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-table
- **Parameter sniffing**: a plan compiled for the first value can be wrong for others. Microsoft calls hints
  a last resort; the options are `OPTION (RECOMPILE)`, `OPTIMIZE FOR (@p = … | UNKNOWN)`, and
  `USE HINT('DISABLE_PARAMETER_SNIFFING')`. https://learn.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-query

## Safe changes

- **`MERGE`**: errors when one target row matches several source rows (dedupe the source); filtering the
  target inside `ON` returns wrong results (filter in a CTE); it needs a `;` terminator; add `HOLDLOCK` for a
  concurrent upsert; at high concurrency separate `UPDATE` and `INSERT` can perform better.
  https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql
- **Count what changed**: `@@ROWCOUNT` (or `ROWCOUNT_BIG()`) right after the statement; `BEGIN TRAN`,
  `COMMIT`, `SET`, and `PRINT` reset it to 0. Run the change inside `BEGIN TRAN`, check the count against
  the predicted one, then `COMMIT` or `ROLLBACK`.
  https://learn.microsoft.com/en-us/sql/t-sql/functions/rowcount-transact-sql

## Parameters

Dynamic SQL goes through `sp_executesql` with typed parameters, never `EXEC(@string)` built with `+`; it also
lets the plan be reused.
https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-executesql-transact-sql

## Fabric Warehouse

- Primary keys and unique constraints are `NONCLUSTERED NOT ENFORCED` and foreign keys `NOT ENFORCED`: keys can
  hold duplicates, so test uniqueness before trusting a join. No `DEFAULT` constraints.
  https://learn.microsoft.com/en-us/fabric/data-warehouse/table-constraints
- Not supported: triggers, recursive queries, materialized views, synonyms, `SET ROWCOUNT`,
  `SET TRANSACTION ISOLATION LEVEL`, `FOR XML`; `MERGE` and `TRUNCATE` are.
  https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area
- Transactions are always `SNAPSHOT` with table-level locking: concurrent `UPDATE`/`DELETE`/`MERGE` on one
  table conflict even on different rows (errors 24556, 24706), so writers retry. No savepoints.
  https://learn.microsoft.com/en-us/fabric/data-warehouse/transactions
- No `datetimeoffset`; `AT TIME ZONE` works on `datetime2`. Only a subset of query hints applies, and
  `NOLOCK`, `@@ROWCOUNT`, and `SET STATISTICS IO` are not listed for it.
