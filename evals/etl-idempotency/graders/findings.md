---
type: llm
weight: 2
---
PASS if the response does all of the following:
- Explains that the plain `INSERT` duplicates rows when the task is rerun (as happened after the halfway failure) and replaces it with an idempotent load: a `MERGE`/upsert on the order key, or a delete-and-replace of the period inside one transaction.
- Replaces `datetime.now()` with the run's data interval (`data_interval_start` / `data_interval_end` or the logical date) so reruns and backfills load the intended day.
- Explains that filtering on `created_at` misses later changes to status and amount, and extracts by an update timestamp (or CDC) with a lookback window covering the 30 days of changes.
- Mentions at least one of: the `created_at::date` cast in the session time zone, the non-sargable filter on a cast column, or `SELECT *` breaking on schema change.

FAIL if it keeps the plain `INSERT` or keeps `datetime.now()`.
