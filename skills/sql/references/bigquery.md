# BigQuery (GoogleSQL)

## Cost comes first

- On-demand queries bill the bytes read. Estimate with a dry run before running anything large
  (`bq query --dry_run`, or `dryRun` in the API); it is free, but reports 0 bytes on tables with row-level
  security and may on federated sources. Cap a run with "maximum bytes billed": over the cap it fails
  without a charge. https://docs.cloud.google.com/bigquery/docs/best-practices-costs ·
  https://docs.cloud.google.com/bigquery/docs/running-queries
- `SELECT *` bills every column, and `LIMIT` does not reduce bytes on a non-clustered table: select columns,
  and use table preview to look at rows. https://docs.cloud.google.com/bigquery/docs/best-practices-performance-compute
- Cached results are free, for about 24 hours, but not with a destination table, changed tables, recent
  streaming inserts, or non-deterministic functions. https://docs.cloud.google.com/bigquery/docs/cached-results

## Pruning

- `require_partition_filter` rejects a query without a filter that eliminates partitions, through views too;
  the predicate must reference only the partition column.
- These do not prune: a subquery or other dynamic value, a comparison with another column, arithmetic or
  unsupported functions on the partition column, an `OR` with another column. Isolate the column and compare
  it with a constant expression. https://docs.cloud.google.com/bigquery/docs/querying-partitioned-tables
- Clustering takes up to 4 columns, in filter order; its savings show only after the run, so the dry run
  overestimates. https://docs.cloud.google.com/bigquery/docs/clustered-tables

## Reading the plan

The execution details show stages, `totalSlotMs`, records read and written, shuffle and spilled bytes; a
stage whose compute max is far above its average is skewed.
https://docs.cloud.google.com/bigquery/docs/query-plan-explanation

## Traps

- A `WITH` clause is not guaranteed to materialize and can run once per reference; a self-join can square the
  rows (use a window function); aggregate before joining.
  https://docs.cloud.google.com/bigquery/docs/best-practices-performance-compute
- `TIMESTAMP` is an absolute instant, `DATETIME` has no zone, and functions default to UTC: pass the zone
  (`DATE(ts, 'America/Sao_Paulo')`). https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/data-types ·
  https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/timestamp_functions
- `SAFE_CAST` turns runtime cast errors into NULL; count them.
  https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/conversion_functions
- `APPROX_COUNT_DISTINCT` is an estimate: label it as one.
  https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/approximate_aggregate_functions
- `MERGE … WHEN MATCHED THEN UPDATE` fails when several source rows match one target row: dedupe the source.
  https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax

## Parameters

Named `@name` or positional `?`, never both in one query; they cannot stand for table or column names.
https://docs.cloud.google.com/bigquery/docs/parameterized-queries
