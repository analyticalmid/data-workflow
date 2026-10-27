---
type: llm
weight: 2
---
PASS if the response does all of the following:
- Explains that duplicate `customer_id` rows in `customers` repeat orders in the merge, and fixes it by reducing `customers` to one row per customer before merging, guarded with `validate="m:1"` (or an equivalent explicit uniqueness check).
- Explains that `df[mask]["amount"] = 0` is chained assignment that never modifies `df` under pandas 3 Copy-on-Write, and rewrites it as `df.loc[mask, "amount"] = 0` (or filters the cancelled orders out).
- Notes that `groupby` drops rows whose `segment` is missing (orders of customers absent from `customers.csv`) and keeps them visible with `dropna=False` or an explicit fill.
- Ends with corrected code in full.

FAIL if it misses the duplicate-customer fan-out or leaves the chained assignment in place.
