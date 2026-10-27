---
name: databricks-spark
description: Apache Spark (PySpark, Spark SQL, Structured Streaming) and the Databricks platform — execution and shuffles, UDFs vs built-ins, caching, Delta layout (OPTIMIZE, VACUUM, liquid clustering, deletion vectors), Unity Catalog, compute and cost, Lakeflow Jobs and pipelines, bundles, secrets, streaming checkpoints, and testing. Use when writing or reviewing Spark code or notebooks, when a job is slow, skewed, out of memory, or costly, or when designing tables and jobs on Databricks.
---

# Databricks and Spark

Spark work is judged at the scale it runs at, so every change is read against the plan and the Spark UI, never
against a sample's timing. Query-level SQL rules (ANSI mode, `EXPLAIN`, join hints, `MERGE`, time travel) live in the
`sql` skill's [`spark-sql.md`](../sql/references/spark-sql.md).

## Steps

1. **Pin the runtime.** Read the Databricks Runtime (or Spark) version from the cluster, job, or bundle config: the
   current LTS is DBR 18 (Spark 4.1); Spark 4 turns ANSI mode on. Done when the version is named.
2. **Find where the time goes** before changing code: the job's stages in the Spark UI, the shuffle read and write
   per stage, spill, the slowest task against the median (skew), and the physical plan. Done when the bottleneck is
   named with the metric that shows it.
3. **Change one thing**, from [Code](#code) or [Tables](#tables), and rerun at the same scale. Done when the same
   metric moved, and the output is unchanged (row count and control totals, per the contract).

## Code

- **Transformations are lazy; actions run them.** Each action starts a job; a shuffle splits it into stages. A
  shuffle (joins, `groupBy`, `repartition`, `distinct`) is the expensive part: count them in the plan.
- **The driver is one machine.** `collect()` and `toPandas()` bring every row to it: use `take(n)`, `limit`, or
  write the result to a table and read a summary.
- **Built-in functions first.** They run in the engine (and on Photon). A Python UDF serializes every row; a pandas
  UDF moves Arrow batches and is up to 100x faster than a Python UDF, but still slower than a built-in. Loops over
  rows in the driver are replaced by column expressions or window functions.
- **Partitions**: `spark.sql.shuffle.partitions` is 200 by default and AQE coalesces small ones. `repartition(n)`
  shuffles to a balanced count; `coalesce(n)` merges without a shuffle, and `coalesce(1)` can pull the upstream work
  onto one task: use `repartition` there.
- **Skew**: AQE splits a partition above 5x the median and above 256 MB in sort-merge joins; for skew it cannot see,
  find the hot keys with a `groupBy(key).count()` and handle them apart.
- **Caching**: on Databricks, avoid `cache()` as a habit and rely on the disk cache; cache only a result reused
  several times in one job, and `unpersist()` it.
- **Testing**: `pyspark.testing.utils.assertDataFrameEqual` and `assertSchemaEqual`, with pytest fixtures for the
  session, on small DataFrames built in the test. https://spark.apache.org/docs/latest/api/python/getting_started/testing_pyspark.html
- https://spark.apache.org/docs/latest/rdd-programming-guide.html ·
  https://spark.apache.org/docs/latest/sql-performance-tuning.html · https://docs.databricks.com/aws/en/udf/ ·
  https://docs.databricks.com/aws/en/lakehouse-architecture/performance-efficiency/best-practices

## Tables

- **Unity Catalog**: names are `catalog.schema.table`; managed tables are the default and recommended (a dropped
  one is recoverable for 7 days); files that are not tables live in volumes (`/Volumes/<catalog>/<schema>/<volume>/…`).
  https://docs.databricks.com/aws/en/data-governance/unity-catalog/ · https://docs.databricks.com/aws/en/volumes/
- **Partitioning**: none under 1 TB; liquid clustering from 1 TB to 100 TB and first above that; a partition holds at
  least 1 GB and partitions on a low-cardinality column only. https://docs.databricks.com/aws/en/tables/partitions
- **Small files**: `OPTIMIZE` bin-packs them and is idempotent; optimized writes are on by default for `MERGE`;
  auto compaction tunes file size. Predictive optimization runs `OPTIMIZE`, `VACUUM`, and `ANALYZE` on managed tables
  by itself. https://docs.databricks.com/aws/en/delta/optimize ·
  https://docs.databricks.com/aws/en/optimizations/predictive-optimization
- **`VACUUM`** keeps 7 days by default and ends time travel past its retention; a shorter retention can delete files
  of jobs still running, so preview with `DRY RUN`. https://docs.databricks.com/aws/en/delta/vacuum
- **Deletion vectors** make `DELETE`, `UPDATE`, and `MERGE` soft deletes: removing personal data for good takes
  `REORG TABLE … APPLY (PURGE)` and then `VACUUM`. https://docs.databricks.com/aws/en/delta/deletion-vectors

## Platform

- **Compute**: serverless for most new work; job compute for scheduled work, which costs much less than all-purpose
  compute; a SQL warehouse for interactive SQL; auto termination on everything interactive. Photon is on by default
  and does not accelerate UDFs or RDD code. https://docs.databricks.com/aws/en/lakehouse-architecture/cost-optimization/best-practices ·
  https://docs.databricks.com/aws/en/compute/photon
- **Lakeflow Jobs** orchestrate tasks as a DAG with time or event triggers. **Lakeflow pipelines** (formerly DLT;
  Python module `pyspark.pipelines` imported as `dp`) declare streaming tables, materialized views, flows, and
  expectations (see the `data-quality` skill). https://docs.databricks.com/aws/en/jobs/ ·
  https://docs.databricks.com/aws/en/ldp/concepts/
- **Declarative Automation Bundles** (formerly Databricks Asset Bundles) keep jobs and pipelines as YAML in the repo
  (`databricks.yml`) and deploy them with `databricks bundle validate | deploy | run`.
  https://docs.databricks.com/aws/en/dev-tools/bundles/
- **Secrets**: `dbutils.secrets.get(scope=…, key=…)`; output shows `[REDACTED]`, but redaction covers only the literal
  value, so a secret is never transformed or printed. https://docs.databricks.com/aws/en/security/secrets/

## Streaming

- Exactly-once needs a replayable source, a checkpoint, and an idempotent sink. The checkpoint survives restarts only
  while sources, the sink path, and stateful operations stay the same.
- A watermark bounds lateness: rows later than it are dropped, and state is cleaned only in append or update mode.
- `trigger(availableNow=True)` processes what exists and stops; it replaces the once trigger.
- `foreachBatch` is at-least-once; a `MERGE` inside it is made idempotent with `txnAppId` and `txnVersion = batchId`,
  and a new checkpoint needs a new `txnAppId`.
- https://spark.apache.org/docs/latest/streaming/apis-on-dataframes-and-datasets.html ·
  https://docs.databricks.com/aws/en/structured-streaming/delta-lake
