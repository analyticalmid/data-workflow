# Snowflake

## Cost and caches

- Credits double with each warehouse size step, billed per second with a 60-second minimum per start. A larger
  size speeds a single query; multi-cluster serves concurrency.
  https://docs.snowflake.com/en/user-guide/warehouses-overview
- Suspending a warehouse drops its data cache; an idle running warehouse still bills.
  https://docs.snowflake.com/en/user-guide/performance-query-warehouse-cache
- The result cache needs identical query text and unchanged data, and skips queries with `RANDOM`,
  `UUID_STRING`, and similar functions. Turn it off (`ALTER SESSION SET USE_CACHED_RESULT = FALSE`) when
  timing a fix, or the second run measures the cache.
  https://docs.snowflake.com/en/user-guide/querying-persisted-results

## Reading the query profile

- Spilling to local storage means memory ran out; spilling to remote storage hurts badly: use a larger
  warehouse or smaller batches.
- Partitions scanned vs partitions total is the pruning: a small fraction means it works.
- An exploding join outputs far more rows than it reads, usually a missing or wrong join condition.
- https://docs.snowflake.com/en/user-guide/ui-snowsight-activity

## Clustering keys

For multi-terabyte tables with selective, degraded queries only; 3–4 columns at most, lowest cardinality first,
an expression for a very high-cardinality column. Reclustering costs credits.
https://docs.snowflake.com/en/user-guide/tables-clustering-keys

## Traps

- **Time**: `TIMESTAMP` means `TIMESTAMP_NTZ` by default (no zone); `LTZ` is UTC shown in the session zone; `TZ`
  stores an offset, not a zone, so DST arithmetic keeps the old offset. The `TIMEZONE` parameter defaults to
  `America/Los_Angeles`, not UTC: set it per session. https://docs.snowflake.com/en/sql-reference/data-types-datetime ·
  https://docs.snowflake.com/en/sql-reference/parameters
- **`TRY_CAST`** returns NULL instead of failing, and only on string input.
  https://docs.snowflake.com/en/sql-reference/functions/try_cast
- **`QUALIFY`** runs after the window functions and before `DISTINCT`.
  https://docs.snowflake.com/en/sql-reference/constructs/qualify
- **`MERGE`** errors when several source rows update or delete one target row while
  `ERROR_ON_NONDETERMINISTIC_MERGE` is true (the default); set to false, the row used is undefined. Duplicate
  source rows on the insert path insert once each. Dedupe the source.
  https://docs.snowflake.com/en/sql-reference/sql/merge
- Standard tables enforce `NOT NULL` and `CHECK` only; primary, unique, and foreign keys are declared, never
  enforced: test uniqueness before trusting a join.
  https://docs.snowflake.com/en/sql-reference/constraints-overview

## Safe changes

- Rehearse a destructive change on a zero-copy clone (`CREATE TABLE t_rehearsal CLONE t`): free until it
  diverges, writable, and can be taken `AT | BEFORE` a past point.
  https://docs.snowflake.com/en/sql-reference/sql/create-clone
- Time Travel (`AT | BEFORE (TIMESTAMP | OFFSET | STATEMENT)`) compares before and after; retention is 1 day by
  default. https://docs.snowflake.com/en/user-guide/data-time-travel

## Parameters

Bind `?`, `:name`, or `%name`; they do not mask values in error messages.
https://docs.snowflake.com/en/sql-reference/bind-variables
