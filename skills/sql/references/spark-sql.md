# Spark SQL and Databricks SQL

Engine behaviour (shuffles, partitions, clusters) lives in the `databricks-spark` skill; this file is about the
query.

## ANSI mode decides wrong result vs error

- With `spark.sql.ansi.enabled = true` (default in Spark 4 and Databricks Runtime 17+), integer overflow raises
  `ARITHMETIC_OVERFLOW`, `CAST('a' AS INT)` raises `CAST_INVALID_INPUT`, and division by zero fails. With it
  off, overflow wraps, decimal overflow and bad casts return NULL, silently.
  https://spark.apache.org/docs/latest/sql-ref-ansi-compliance.html ·
  https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-ansi-compliance
- Databricks SQL warehouses default `ANSI_MODE` to true only for accounts created on or after 2022-10-19: check
  it before assuming errors surface. https://docs.databricks.com/aws/en/sql/language-manual/parameters/ansi_mode
- `try_cast`, `try_divide`, `try_add`, `try_mod` return NULL on purpose; use them where NULL is the intended
  answer, and count the NULLs they produce.

## Reading the plan

- `EXPLAIN FORMATTED` gives the physical plan outline with node details; `EXTENDED` adds the logical plans.
  https://spark.apache.org/docs/latest/sql-ref-syntax-qry-explain.html
- In the Spark UI SQL tab, an `Exchange` is a shuffle (bytes written and read, fetch wait);
  `BroadcastExchange` shows the broadcast size; scans show scan time.
  https://spark.apache.org/docs/latest/web-ui.html
- `BroadcastHashJoin` ships the small side to every executor; `SortMergeJoin` shuffles and sorts both.
  `autoBroadcastJoinThreshold` defaults to 10 MB, and AQE converts to broadcast at runtime when one side turns
  out small. Hints (`BROADCAST` > `MERGE` > `SHUFFLE_HASH` > `SHUFFLE_REPLICATE_NL`) are not guaranteed.
  https://spark.apache.org/docs/latest/sql-performance-tuning.html

## Pruning

- Delta keeps per-file min, max, and null counts on the first 32 columns; a filter on such a column skips
  files. Filter directly on the column, not on an expression of it.
  https://docs.databricks.com/aws/en/delta/data-skipping
- Dynamic file pruning applies to joins on tables of 10 GB and 10 files or more.
  https://docs.databricks.com/aws/en/optimizations/dynamic-file-pruning

## Traps

- **`MERGE INTO`** fails with `DELTA_MULTIPLE_SOURCE_ROW_MATCHING_TARGET_ROW_IN_MERGE` when several source rows
  match one target row: dedupe the source first. Every `WHEN` clause but the last needs a condition, and
  `WHEN NOT MATCHED BY SOURCE` without one can rewrite a large part of the target.
  https://docs.databricks.com/aws/en/sql/language-manual/delta-merge-into
- **Time**: `TIMESTAMP` is stored in UTC and shown in the session time zone; `TIMESTAMP_NTZ` ignores zones.
  CSV and JSON reads infer `TIMESTAMP`, not NTZ. https://docs.databricks.com/aws/en/sql/language-manual/data-types/timestamp-type ·
  https://docs.databricks.com/aws/en/sql/language-manual/data-types/timestamp-ntz-type
- **`QUALIFY`** (DBR 10.4+ and Databricks SQL) needs a window function in `SELECT` or `QUALIFY`, and no
  aggregate in its condition. https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-syntax-qry-select-qualify

## Safe changes

- Compare before and after with time travel: `SELECT … FROM t VERSION AS OF 41` or `TIMESTAMP AS OF '…'`, and
  undo with `RESTORE TABLE t TO VERSION AS OF 41`. Rely on the last 7 days only, unless the table's retention
  settings were raised. https://docs.databricks.com/aws/en/delta/history
- `DESCRIBE HISTORY t` returns each write with its `operationMetrics` column: read it to confirm how many rows
  a `MERGE`, `UPDATE`, or `DELETE` touched. https://docs.databricks.com/aws/en/delta/history

## Layout

Liquid clustering replaces partitioning and `ZORDER` for new tables and is incompatible with both; run
`OPTIMIZE … FULL` after changing the keys. https://docs.databricks.com/aws/en/delta/clustering
