---
type: llm
weight: 2
---
PASS if the response fixes at least five of these seven, each with the reason:
1. Replaces the `country_name` Python UDF with a join to `countries` (broadcast, since it is tiny).
2. Replaces the `normalize_email` Python UDF with built-in functions (`lower`, `trim`).
3. Removes the `collect()` of distinct `user_id`s and the `isin` list, which pull every ID to the driver (the cause of the driver OOM); the filter is either dropped as a no-op or rewritten as a join or semi join.
4. Removes `coalesce(1)`, which forces the whole write through one task.
5. Drops `user_id` from `partitionBy` (high cardinality, millions of tiny partitions), and questions partitioning at all, or recommends liquid clustering instead.
6. Removes or justifies the `.cache()` of a 400 GB table read once.
7. Mentions verifying the output (row counts or totals) or reading the Spark UI / plan to confirm the speed-up.

FAIL if it keeps the driver `collect()` of user IDs or keeps `coalesce(1)`.
