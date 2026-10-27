---
name: data-quality
description: Data quality and personal data in pipelines and analytics — tests (unique, not null, accepted values, referential integrity, freshness, volume, reconciliation), schema contracts, quarantine vs fail, dbt tests and contracts, Lakeflow expectations, Snowflake DMFs, and LGPD/GDPR handling with masking. Use when adding tests to a model or pipeline, deciding what to check, investigating bad data, or handling CPF, emails, health, or other personal data.
---

# Data quality

A table is trusted when its tests state the grain and the rules the consumers rely on, and run where a failure
stops bad data before a consumer reads it.

## Steps

1. **Write the promises.** For each table: its grain (the key that must be unique and non-null), the columns
   consumers filter or join on, the allowed values of each code column, the parents of each foreign key, how fresh
   it must be, and the expected daily volume. Done when every promise is one line a test can check.
2. **Map each promise to a check** from [Checks](#checks), in the tool the repo already uses
   ([Where the checks live](#where-the-checks-live)).
3. **Choose what a failure does** ([Fail, warn, or quarantine](#fail-warn-or-quarantine)). Done when each check
   names its action and who is told.
4. **Place them.** Source checks (freshness, volume, schema) run before the transformation; grain and integrity
   checks run on every model that consumers read; reconciliation runs at the end of the pipeline.
5. **Prove them red.** Each new check is run once against data that breaks it (a fixture, a unit test, or a
   duplicated row in a dev copy) and seen to fail. Done when every check has failed once on purpose.

## Checks

| Promise | Check |
| --- | --- |
| Grain | key `unique` and `not_null` (composite keys: unique combination) |
| Codes | `accepted_values` |
| Foreign keys | `relationships`: every child key exists in the parent; warehouses that do not enforce keys (Snowflake, Fabric, lakehouse tables) need this test |
| Schema | an enforced contract on names and types; a schema-change check on sources |
| Freshness | the newest load timestamp within a stated lag |
| Volume | rows loaded in the period within an expected band (a fixed band, or the last N periods) |
| Reconciliation | row count and control totals equal to the source, or to the previous layer, within a stated tolerance |
| Ranges | numbers within business bounds (no negative quantity, discount ≤ 100%), dates not in the future |
| Distribution | the share of nulls or of a category within a band, for columns that feed models or KPIs |

## Fail, warn, or quarantine

- **Fail** when publishing would be wrong: grain broken, reconciliation off, schema changed. The load stops and
  the previous good data stays visible.
- **Warn** when the data is usable but degraded: freshness slightly late, a distribution drifting. Someone is
  told; nothing stops.
- **Quarantine** when bad rows are few and the rest is good: split rows into valid and invalid on the negation of
  all rules, publish the valid ones, keep the invalid ones in a table with the rule they broke, and count them.
  A quarantine nobody reads is a silent drop: it gets a volume check of its own.

## Where the checks live

- **dbt**:
  - Generic tests: `unique`, `not_null`, `accepted_values`, `relationships`.
  - Singular tests: a `.sql` in `tests/` that returns the failing rows.
  - `severity`, `warn_if`, `error_if`: thresholds on the failing-row count.
  - `store_failures: true`: keeps the failing rows for inspection.
  - Unit tests (1.8+): `given` / `expect` rows, for logic, run in development and CI.
  - `contract: enforced: true`: fails at build on names, types, and column count.
  - `dbt freshness` with `loaded_at_field` and `warn_after` / `error_after`.
  - https://docs.getdbt.com/docs/build/data-tests · https://docs.getdbt.com/docs/build/unit-tests ·
    https://docs.getdbt.com/reference/resource-configs/contract ·
    https://docs.getdbt.com/reference/resource-properties/freshness
- **Databricks Lakeflow pipelines**:
  - Expectations: `EXPECT` warns and counts; `ON VIOLATION DROP ROW` drops and counts; `ON VIOLATION FAIL UPDATE`
    fails and rolls back. In Python: `@dp.expect`, `@dp.expect_or_drop`, `@dp.expect_or_fail`, `@dp.expect_all…`.
  - A validation table does not block downstream tables; split into pipelines with a job dependency to stop them.
  - https://docs.databricks.com/aws/en/ldp/expectations · https://docs.databricks.com/aws/en/ldp/expectation-patterns
- **Snowflake**: system data metric functions in `SNOWFLAKE.CORE`:
  - volume and freshness: `ROW_COUNT`, `FRESHNESS`;
  - nulls and keys: `NULL_COUNT`, `DUPLICATE_COUNT`, `REFERENTIAL_INTEGRITY_COUNT`;
  - codes, schema, and outliers: `ACCEPTED_VALUES`, `SCHEMA_CHANGE_COUNT`, `OUTLIER_ZSCORE_COUNT`.
  - https://docs.snowflake.com/en/user-guide/data-quality-system-dmfs
- **Great Expectations**: Expectations grouped in a Suite, bound to a Batch Definition by a Validation Definition,
  run by a Checkpoint. https://docs.greatexpectations.io/docs/core/introduction/gx_overview
- **None of these**: plain SQL assertions that return the failing rows, run by the orchestrator after each load.

## Personal data

Read [`references/personal-data.md`](references/personal-data.md) when a table holds data about people: what LGPD and
GDPR count as personal and sensitive data, what pseudonymised data still is, the incident deadlines, and the masking
features of each platform.
