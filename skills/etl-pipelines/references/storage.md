# Storage architecture

## Lake, warehouse, lakehouse

- **Data lake**: files in object storage, any format, schema on read; cheap and flexible, and a swamp without
  governance. https://learn.microsoft.com/en-us/azure/architecture/data-guide/scenarios/data-lake
- **Data warehouse**: schema on write, built for BI and SQL, often in a proprietary format (Snowflake, BigQuery,
  Fabric Warehouse, SQL Server).
- **Lakehouse**: open table formats on the lake (Delta, Iceberg) that add transactions, schema enforcement, and
  time travel, so one copy of the data serves engineering and BI. https://docs.databricks.com/aws/en/lakehouse/
- Choose by who reads and how: SQL-only teams on structured data fit a warehouse; mixed workloads (BI, Spark, ML) on
  the same data fit a lakehouse; raw files nobody queries directly fit a lake landing zone in either.

## Medallion layers

| Layer | Holds | Rules |
| --- | --- | --- |
| Bronze | data exactly as it arrived, appended, with load metadata (source file, load time) | kept for reprocessing and audit; most fields as string or variant; no cleaning |
| Silver | validated, cleaned, deduplicated, typed, conformed data, joined to reference data | built only from bronze, never written directly by ingestion; grain and keys tested |
| Gold | business-level tables: facts and dimensions, aggregates, KPI tables | shaped for its consumers; the layer BI and analysts read |

- Medallion is a recommended practice, not a requirement: a small pipeline can merge layers when it states why.
  https://docs.databricks.com/aws/en/lakehouse/medallion
- Microsoft Fabric recommends one lakehouse per layer, ideally each in its own workspace, shortcuts in bronze instead
  of copies, Delta for silver and gold, files of 128 MB to 1 GB, and liquid clustering over partitioning in silver and
  gold; gold can be a Fabric Warehouse. https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture
- Personal data is minimized at silver: gold carries only what its consumers need (see the `data-quality` skill).

## Table formats

- **Delta Lake**: ACID transactions with serializable isolation, schema enforcement (unknown columns and type
  mismatches are rejected; `mergeSchema` per write to evolve), time travel (`VERSION AS OF`, `TIMESTAMP AS OF`; about 30
  days of log, 7 days of deleted files by default), `MERGE`, `UPDATE`, `DELETE`.
  https://docs.delta.io/latest/delta-intro.html · https://docs.delta.io/latest/delta-batch.html
- **Apache Iceberg**: columns tracked by ID, so schema changes are metadata-only; hidden partitioning (a partition such as
  `day(event_time)` that queries need not filter on explicitly); partition evolution without rewriting old data; time
  travel and rollback; optimistic concurrency. https://iceberg.apache.org/docs/latest/ ·
  https://iceberg.apache.org/docs/latest/partitioning/ · https://iceberg.apache.org/docs/latest/evolution/
- **Parquet** (the file format under both): columnar, split into row groups of column chunks and pages; min/max
  statistics let readers skip pages and row groups, so sorted or clustered data reads less.
  https://parquet.apache.org/docs/concepts/ · https://parquet.apache.org/docs/file-format/pageindex/
- **Small files** slow every read: compact them (`OPTIMIZE` on Delta, which is idempotent) or let the platform do it
  (predictive optimization, auto compaction). https://docs.databricks.com/aws/en/delta/optimize
