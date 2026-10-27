---
name: data-profile
description: Profile a dataset — a file, a DataFrame, or a warehouse table — before using it.
disable-model-invocation: true
argument-hint: "[file path, table name, or DataFrame variable]"
---

Profile: $ARGUMENTS

The profile answers one question: can this data be trusted for the work ahead, and where does it break.

1. **Find the data and the engine.** A file (`.csv`, `.parquet`, `.xlsx`, `.json`) is profiled with pandas; invoke the
   `data-workflow:python-data` skill and load it lean. A warehouse or database table is profiled with SQL aggregates
   in place, never by pulling the rows into memory; invoke the `data-workflow:sql` skill for the dialect. Use only
   access the repo or the shell already has.
2. **Size and cost first.** Row count, column count, and, on a warehouse billed by bytes, the estimated bytes
   scanned. Above a few million rows, profile a stated sample and say which numbers come from it.
3. **Per column**, in one table: type as stored, the type it should be, null share, distinct count, min and max,
   the five most frequent values with their share, and for numbers the mean, median, and the share outside the
   IQR fences.
4. **Suspicious values**: empty or whitespace-only strings, sentinels (`-1`, `0`, `9999`, `1900-01-01`,
   `'N/A'`, `'null'` as text), mixed types in one column, numbers or dates stored as text, leading zeros lost,
   inconsistent casing or spacing in categories, future dates, negative amounts or quantities.
5. **Keys and grain**: every column (or small set) that is unique and non-null is a candidate key; state the grain
   the data supports and the duplicates that break it, with a count and three examples.
6. **Time**: for each date column, the range, gaps in the daily or monthly series, the time zone if any, and
   whether the latest date matches the expected freshness.
7. **Personal data**: flag columns that hold names, national IDs, emails, phones, addresses, health, or financial
   identifiers. Their sample values in the report are masked, per the contract.
8. **Report** in the chat: the per-column table, then the findings ranked by how badly they would break the work,
   each with the fix (a cast, a dedupe rule, a filter, a question for the data owner). Write it to a file only when
   asked. Done when every column has its row and every finding its fix or its open question.
