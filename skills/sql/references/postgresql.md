# PostgreSQL

Also Supabase, which runs PostgreSQL. Sources are the current manual, linked per section.

## Reading the plan

- `EXPLAIN (ANALYZE, BUFFERS)` runs the statement. Wrap anything that writes in `BEGIN; EXPLAIN ANALYZE …;
  ROLLBACK;`. BUFFERS shows shared, local, and temp blocks hit (cache) and read (disk).
- Cost is `(startup, total)` in arbitrary units. `rows` is what a node outputs, not what it scanned.
- Actual time and rows are averages per loop: multiply by `loops`.
- The first thing to read is estimated vs actual rows; a gap of an order of magnitude points to stale or
  missing statistics. `Rows Removed by Filter` shows work thrown away.
- Nodes: Seq Scan reads the table; Index Scan fetches in index order (few rows, or a matching `ORDER BY`);
  Bitmap Index + Bitmap Heap Scan collects row locations, sorts them, then reads pages; Nested Loop runs
  the inner side once per outer row; Hash Join hashes one input in memory; Merge Join needs both sorted.
- https://www.postgresql.org/docs/current/using-explain.html ·
  https://www.postgresql.org/docs/current/sql-explain.html

## Indexes and sargability

- A B-tree serves `< <= = >= >`, `BETWEEN`, `IN`, `IS [NOT] NULL`, and `LIKE 'prefix%'`; `LIKE '%suffix'`
  does not, and pattern matching outside the C locale needs a `*_pattern_ops` operator class.
- A function on the column (`lower(email) = …`) is served only by an expression index on the same
  expression, which costs on every insert and non-HOT update.
- A partial index serves a query only when its `WHERE` implies the index predicate at planning time, so a
  parameterized clause (`x < $1`) does not use it.
- A multicolumn index narrows the scan by equality on its leading columns plus one range on the next;
  columns after that are only checked. More than three columns rarely pays.
- `INCLUDE` enables index-only scans, which only skip the heap when the visibility map marks most pages
  all-visible (a vacuumed table).
- https://www.postgresql.org/docs/current/indexes-types.html ·
  https://www.postgresql.org/docs/current/indexes-expressional.html ·
  https://www.postgresql.org/docs/current/indexes-partial.html ·
  https://www.postgresql.org/docs/current/indexes-multicolumn.html ·
  https://www.postgresql.org/docs/current/indexes-index-only-scans.html

## Statistics

- Autovacuum analyzes by rows changed and never analyzes partitioned parents or foreign tables: run
  `ANALYZE` on those by hand, and after a bulk load.
- https://www.postgresql.org/docs/current/routine-vacuuming.html

## Traps

- **CTEs** (v12+): a side-effect-free CTE referenced once is inlined; one referenced twice is materialized,
  so outer filters and indexes stop reaching it. Force with `MATERIALIZED` / `NOT MATERIALIZED`.
  Data-modifying CTEs share one snapshot; only `RETURNING` passes changes between them.
  https://www.postgresql.org/docs/current/queries-with.html
- **Time**: `timestamp` (without time zone) silently drops a zone in the input; `timestamptz` stores UTC and
  displays in the session `TimeZone`. `ts AT TIME ZONE 'America/Sao_Paulo'` turns a `timestamptz` into the
  wall clock there. `date_trunc` on `timestamptz` cuts in the session zone unless given a zone. `now()` is
  the transaction start. https://www.postgresql.org/docs/current/datatype-datetime.html ·
  https://www.postgresql.org/docs/current/functions-datetime.html
- **Integer division** truncates toward zero; `avg(integer)` returns `numeric`.
  https://www.postgresql.org/docs/current/functions-math.html
- **Evaluation order** of sub-expressions is undefined, and a constant `1/0` can fail at planning time even in
  an unreached `CASE` branch: filter with `WHERE` or `FILTER (WHERE …)`.
  https://www.postgresql.org/docs/current/sql-expressions.html
- **`DISTINCT ON`** needs its expressions to lead the `ORDER BY`, and an `ORDER BY` that breaks ties to be
  deterministic. https://www.postgresql.org/docs/current/sql-select.html

## Safe changes

- `INSERT … ON CONFLICT (target) DO UPDATE` is an atomic upsert; `EXCLUDED` is the proposed row; touching one
  row twice in a statement is a cardinality error. https://www.postgresql.org/docs/current/sql-insert.html
- `RETURNING` works on `INSERT`, `UPDATE`, `DELETE`, and `MERGE`, with `old.*` / `new.*`: run the change in a
  transaction, read what it returned, then `COMMIT` or `ROLLBACK`.
  https://www.postgresql.org/docs/current/dml-returning.html
- `SET TRANSACTION READ ONLY` blocks writes to non-temporary tables, DDL, and `TRUNCATE` for exploration
  sessions. https://www.postgresql.org/docs/current/sql-set-transaction.html
- `SET statement_timeout = '30s'` and `lock_timeout` per session bound a heavy query; set them per session,
  never in `postgresql.conf`. https://www.postgresql.org/docs/current/runtime-config-client.html

## Parameters

Drivers bind `$1`, `$2` (or the driver's placeholder); a value concatenated into the text is an injection.
