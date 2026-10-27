---
name: pandas-method
description: Writing a join-and-aggregate from scratch, scored on the safeguards the code carries.
tags: [python]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
Write the pandas code (pandas 3.1) that computes September 2026 revenue per customer segment.

- `orders.parquet`: `order_id`, `customer_id`, `created_at` (UTC timestamp), `amount`, `status`, and 30 other columns.
- `customers.parquet`: `customer_id`, `segment`, `name`, `email`, and 20 other columns.

Revenue excludes cancelled orders. The business reports in São Paulo time.
