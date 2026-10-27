---
name: etl-pipelines
description: ETL and ELT pipelines and storage architecture — extraction from APIs, databases (watermarks, CDC), files and streams; idempotent loads, reruns and backfills, dbt incremental models; medallion layers; lake vs warehouse vs lakehouse; Delta, Iceberg, Parquet; dimensional modeling (grain, facts, SCD); orchestration. Use when writing or reviewing any load job, ingestion script, Airflow DAG or task, dbt model, or Lakeflow job; when a load duplicates, misses updates, or breaks on rerun; when choosing storage; or when modeling a warehouse.
---

# ETL pipelines

A pipeline is correct when running it twice, late, or for a past period gives the same tables as running it once on
time. Every design decision below serves that property: **idempotent**.

## Steps

1. **Name the consumer and the contract.** Who reads the output, at what grain, how fresh, with which tests (the
   `data-quality` skill). Done when the output tables, their grain, and their freshness are written down.
2. **Profile each source** (`/data-workflow:data-profile`): volume, change rate, the column that marks changes, how
   deletes appear, and the access limits (rate limits, query windows). Done when each source has its extraction method
   from [Extraction](#extraction).
3. **Lay out the layers.** Raw data lands untouched, cleaning and conforming happen once, and business tables sit on
   top; read [`references/storage.md`](references/storage.md) for medallion layers, lake vs warehouse vs lakehouse,
   and table formats. For a warehouse or a BI model, read
   [`references/dimensional-modeling.md`](references/dimensional-modeling.md). Done when every table has its layer, its
   grain, and its key.
4. **Make each load idempotent** with a pattern from [Loading](#loading), and decide how a backfill runs. Done when a
   rerun of any period is a safe, stated command.
5. **Schedule and observe.** Orchestrate per [Orchestration](#orchestration); every task logs rows read, written, and
   rejected. Done when a failed run says which period failed and how to rerun it.
6. **Prove it.** Run one period twice and compare row counts and control totals; run a backfill of two past periods.
   Done when both runs leave the tables identical.

## Extraction

- **APIs**: follow the API's pagination to its end marker (an empty next-page token, not a short page), keep the other
  request arguments fixed across pages, and expect page tokens to expire. Retry 408, 429, and 5xx with exponential
  backoff and jitter, honouring `Retry-After`; fail fast on authentication or validation errors; retry in one layer
  only. https://google.aip.dev/158 · https://www.rfc-editor.org/rfc/rfc6585#section-4 ·
  https://learn.microsoft.com/en-us/azure/architecture/patterns/retry
- **Databases by watermark** (`updated_at` or an increasing key): an incrementing key sees inserts only; a timestamp
  alone loses rows that tie on it after a crash, rows updated without touching it, and rows committed late with an
  earlier timestamp. Use timestamp plus key, an inclusive `>=` cursor with a dedupe on the key, and a lookback window
  for late commits. Deletes are invisible to a watermark.
  https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/overview.html ·
  https://docs.airbyte.com/platform/using-airbyte/core-concepts/sync-modes/incremental-append
- **CDC** (log-based, e.g. Debezium, Snowflake streams, SQL Server change tracking) captures inserts, updates, and
  deletes without a timestamp column. After a crash it delivers some changes twice, so the consumer dedupes on key and
  log position. https://debezium.io/documentation/faq/ · https://docs.snowflake.com/en/user-guide/streams-intro
- **Files**: land them as they are, record the source file path, and decide what a new or changed column does. Auto
  Loader keeps the schema in `cloudFiles.schemaLocation`, infers JSON and CSV columns as strings unless
  `cloudFiles.inferColumnTypes` is on, and puts mismatches in `_rescued_data`; `addNewColumns` fails the stream once and
  resumes on restart. https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/schema
- **Streams**: assume at-least-once; make the sink idempotent with keyed upserts, or store the offset in the same
  transaction as the output. https://kafka.apache.org/43/design/design/

## Loading

- **Upsert on a key** (`MERGE`): dedupe the source to one row per key first, add the period to the merge condition so
  the target is pruned, and give `WHEN NOT MATCHED BY SOURCE … DELETE` a condition.
  https://docs.databricks.com/aws/en/delta/merge
- **Replace a period**: `replaceWhere` / `INSERT … REPLACE WHERE` overwrites exactly the rows of a predicate, atomically,
  and rejects rows outside it; an `INSERT OVERWRITE` without a partition spec truncates the whole table.
  https://docs.delta.io/latest/delta-batch.html ·
  https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-syntax-dml-insert-into
- **Append** only for immutable events, deduplicated downstream on an event ID.
- **dbt incremental**: set `unique_key` (non-null; a list for composite keys), or the model appends duplicates; write
  SQL valid with and without `is_incremental()`; reprocess late data with a lookback window on the filter; know that
  `on_schema_change` never backfills old rows and `--full-refresh` rebuilds.
  https://docs.getdbt.com/docs/build/incremental-models · https://docs.getdbt.com/docs/build/incremental-strategy
- **History**: slowly changing dimensions follow [`references/dimensional-modeling.md`](references/dimensional-modeling.md);
  on Databricks, `AUTO CDC … INTO` handles SCD 1 and 2 from out-of-order changes.

## Orchestration

- A task is a transaction: it reads a fixed period keyed by the run's interval (`data_interval_start` and
  `data_interval_end` in Airflow, the job's parameters in Lakeflow Jobs), never "latest" or `now()`, and writes with an
  idempotent pattern, so clearing and rerunning it is safe.
- The logical date is the start of the interval, and the run fires after the interval ends. Backfill a range with the
  orchestrator's backfill command rather than a loop in code; leave catchup off unless every past run is wanted.
- Retries with a delay for transient errors; a timeout per task; a permanent error fails without retrying.
- https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html ·
  https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html · https://docs.databricks.com/aws/en/jobs/
