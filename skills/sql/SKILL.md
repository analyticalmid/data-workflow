---
name: sql
description: SQL review, verification, and fixes in PostgreSQL, Spark SQL / Databricks, T-SQL (SQL Server, Azure SQL, Fabric), BigQuery, and Snowflake. Use when writing a query, reviewing one, chasing a wrong result or a duplicate, speeding up a slow or costly query, or rewriting one safely.
---

# SQL

A query is reviewed on three passes (correctness, performance, safety) and is only called fixed once the
fix is verified against the original on the same input.

## Steps

1. **Pin the dialect and version.** Read it from the repo (connection config, dbt `profiles.yml`, migration
   tool, file extension) or from the syntax. Then read the matching reference before judging anything:
   - PostgreSQL (and Supabase) → [`references/postgresql.md`](references/postgresql.md)
   - T-SQL: SQL Server, Azure SQL, Fabric Warehouse → [`references/tsql.md`](references/tsql.md)
   - Spark SQL, Databricks SQL → [`references/spark-sql.md`](references/spark-sql.md)
   - BigQuery → [`references/bigquery.md`](references/bigquery.md)
   - Snowflake → [`references/snowflake.md`](references/snowflake.md)

   Done when the dialect is named and its reference is read; an unknown dialect is asked about, not guessed.
2. **State the question and the grain.** One sentence on what the query answers, then the grain of every
   input (one row per what) and of the output. Done when each table and CTE has its grain written down.
3. **Correctness pass.** Apply every rule of [Correctness](#correctness) and the dialect's own traps. Done
   when each rule is marked `ok`, `finding`, or `n/a` for this query.
4. **Performance pass.** Read the plan the dialect reference describes, at real volume when it exists. Find
   where the time or the bytes go before proposing anything. Done when each finding names the plan node or
   metric that shows it.
5. **Safety pass.** Values arrive as bound parameters; a statement that changes data is wrapped as the
   contract requires (counted target, go-ahead, undo path). Done when every literal and every write is
   accounted for.
6. **Fix.** The smallest rewrite that removes each finding. A fix that changes the result on purpose (it was
   a bug) says so in one line: what the old query returned and what the new one returns.
7. **Verify.** Run [Verification](#verification). Done when every fixed query carries its evidence, or is
   marked `unverified` with the reason (no access, no data) and the exact command for the user to run.

## Correctness

- **Fan-out.** A join to a table at a finer grain multiplies the rows of the coarser side, so a `SUM` or
  `COUNT` of a coarser-grain column over the join counts it once per match. Aggregate the finer table to the
  join key first, or aggregate before joining. A `DISTINCT` added to make the numbers look right is a symptom
  of fan-out, not a fix.
- **Join keys are unique where the grain says so.** Check with `GROUP BY key HAVING COUNT(*) > 1` before
  trusting a dimension join; an unenforced key (lakehouse, Fabric, Snowflake) can hold duplicates.
- **Outer joins stay outer.** A `WHERE` condition on the optional side of a `LEFT JOIN` removes the
  NULL-filled rows and turns it into an inner join; the condition belongs in `ON`.
- **NULL comparisons.** `= NULL`, `<> x`, and `NOT IN (subquery)` all yield unknown on NULL: `NOT IN` returns
  no rows at all when the subquery holds one NULL. Use `IS NULL`, `IS [NOT] DISTINCT FROM`, and
  `NOT EXISTS`.
- **Aggregates over nothing.** `SUM`, `AVG`, `MIN`, `MAX` skip NULLs and return NULL over zero rows; wrap
  them in `COALESCE` where the consumer expects 0. `COUNT(col)` skips NULLs, `COUNT(*)` does not.
- **Division.** Integer by integer truncates (`5/2 = 2`); cast one side to a decimal type. Divide by
  `NULLIF(denominator, 0)` where zero is possible.
- **Time ranges are half-open.** `ts >= start AND ts < next_start`, never `BETWEEN` with a `23:59:59` end;
  name the time zone the day is cut in.
- **Window frames.** With `ORDER BY` and no frame, the frame ends at the current row's last peer, so
  `LAST_VALUE` returns the current row. Write the frame (`ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED
  FOLLOWING`) whenever the function reads beyond the current row.
- **Deterministic picks.** Dedupe with `ROW_NUMBER()` / `QUALIFY` / `DISTINCT ON` over an `ORDER BY` that is
  unique within the partition; a tie makes the kept row change between runs. The same holds for `LIMIT` /
  `TOP` / `FETCH`: no unique `ORDER BY`, no defined result.
- **Order only at the end.** Row order exists only in the outermost `ORDER BY`; one inside a view, subquery,
  or `INSERT … SELECT` orders nothing.
- **Types meet on purpose.** Compare like with like; an implicit cast between a string and a number or a
  date changes results and blocks indexes. Read the dialect's coercion rules.

## Verification

Run the original and the fix on the same input (a fixed snapshot, time travel, or a clone when the dialect
has them) and compare:

1. Row count of both.
2. Control totals: `SUM` of every measure, `COUNT(DISTINCT key)`, `MIN`/`MAX` of dates.
3. Row-level difference both ways on the rows with their multiplicity, which works the same in every
   dialect and catches a changed duplicate:

   ```sql
   WITH old_rows AS (SELECT <columns>, COUNT(*) AS copies FROM (<original>) AS original GROUP BY <columns>),
        new_rows AS (SELECT <columns>, COUNT(*) AS copies FROM (<fixed>) AS fixed GROUP BY <columns>)
   SELECT 'only in fixed' AS side, * FROM (SELECT * FROM new_rows EXCEPT SELECT * FROM old_rows) AS added
   UNION ALL
   SELECT 'only in original', * FROM (SELECT * FROM old_rows EXCEPT SELECT * FROM new_rows) AS removed;
   ```

   The grouped rows are already distinct, so whichever spelling the dialect requires (`EXCEPT`,
   `EXCEPT DISTINCT`, Snowflake's `MINUS`) gives the same answer.
4. For performance, the same metric before and after from the plan or the query history (time, rows read,
   bytes scanned, spill).

A difference that the fix intended is listed as such; any other difference sends the work back to step 3.

## Output

One block per finding: severity (`critical` wrong result · `high` slow or costly at real volume · `medium` ·
`low`), the line, what happens, why, and the fix. Then the fixed query in full, then the verification
evidence.
