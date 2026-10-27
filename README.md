# data-workflow

English · [Português](README.pt-BR.md)

A data workflow for [Claude Code](https://code.claude.com), from the query to the dashboard. It loads the
right way to do each data task into the agent: reviewing and fixing SQL across dialects, wrangling data with
pandas and numpy, designing ETL and storage (medallion, lake, warehouse, lakehouse), working in Databricks
and Spark, building Power BI models in DAX and Power Query, designing dashboards, and testing data quality.

## Install

```sh
claude plugin marketplace add analyticalmid/data-workflow
claude plugin install data-workflow@data-workflow
```

Inside a session: `/plugin marketplace add analyticalmid/data-workflow`, then
`/plugin install data-workflow@data-workflow`.

## The contract

Every session starts with [`contract.md`](contract.md): read by default, a go-ahead before any statement that
changes data on a shared target, grain stated before every join, every transformation reconciled, credentials
and personal data kept out of the output. The repository's own instructions add to it and win where they
disagree.

## What ships

| Piece | What it does |
| --- | --- |
| Contract | The always-on rules above, printed at session start |
| `sql` skill | Review, verify, and fix queries: correctness (fan-out, NULL, grain, windows, time), performance from the plan, safety. References for PostgreSQL, T-SQL (SQL Server, Azure SQL, Fabric), Spark SQL / Databricks, BigQuery, and Snowflake |
| `/data-workflow:sql-review` | Runs that review on a query, a file, an object, or the SQL in the current diff, and verifies the fix against the original |
| `sql-reviewer` agent | Read-only second reviewer for long queries, migrations, and statements that write |
| `python-data` skill | pandas and numpy: lean loading, missing data, validated merges, vectorization and memory, dates and time zones, reshaping, reconciliation; references for pandas 3 and NumPy 2 changes |
| `/data-workflow:data-profile` | Profiles a file, a DataFrame, or a warehouse table: per-column stats, suspicious values, candidate keys and grain, freshness, personal data flagged |

Skills, commands, and reviewers land release by release; this table lists what is in the current one.

## License

[MIT](LICENSE)
