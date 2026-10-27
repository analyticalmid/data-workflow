---
type: llm
weight: 2
---
Count how many of these safeguards the code carries:
1. Reads only the needed columns (`columns=` on `read_parquet`).
2. Guards the join cardinality: `merge(..., validate="m:1")` or an explicit uniqueness check on `customers.customer_id`.
3. Keeps orders whose customer or segment is missing visible (`how="left"` plus `dropna=False` on the groupby, an explicit fill, or a reported count), instead of letting them vanish.
4. Cuts September in São Paulo time with a half-open range (start inclusive, next month's start exclusive) after converting from UTC.
5. Reconciles: compares a row count or an amount total before and after the join or filter, or asserts it.

PASS if the code carries at least four of the five. FAIL otherwise. State the count in your verdict.
