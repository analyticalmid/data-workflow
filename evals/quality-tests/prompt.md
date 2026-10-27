---
name: quality-tests
description: Choosing the tests and failure actions for a gold fact table that feeds a revenue dashboard.
tags: [data-quality]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
We use dbt on Snowflake. `gold.fct_sales` has one row per order line and feeds the CFO's revenue dashboard. Columns: `order_line_id`, `order_id`, `customer_key` (references `gold.dim_customer`), `product_key` (references `gold.dim_product`), `order_date`, `quantity`, `net_amount`, `currency`, `loaded_at`. It is rebuilt every hour from `silver.order_lines`.

Which tests should this model have, and what should happen when each one fails? Give me the YAML.
