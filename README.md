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

Skills, commands, and reviewers land release by release; this table lists what is in the current one.

## License

[MIT](LICENSE)
