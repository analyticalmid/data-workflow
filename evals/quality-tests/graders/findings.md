---
type: llm
weight: 2
---
PASS if the response covers at least five of these six and states a failure action (error, warn, or quarantine) for the ones it covers:
1. `unique` and `not_null` on `order_line_id` (the grain).
2. `relationships` from `customer_key` and `product_key` to their dimensions, noting that Snowflake does not enforce foreign keys.
3. `accepted_values` on `currency`.
4. Range checks: `quantity` and `net_amount` within business bounds, `order_date` not in the future.
5. Freshness on `loaded_at` (or on the source) matching the hourly rebuild.
6. Reconciliation of row count or `net_amount` total against `silver.order_lines`.

It must also give dbt YAML for the tests. FAIL if it covers fewer than five, or gives no failure actions.
