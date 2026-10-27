---
name: sql-fanout
description: A revenue query with a fan-out join, an outer join turned inner, and a closed timestamp range.
tags: [sql, smoke]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
Finance says the September revenue by region from this PostgreSQL query is too high. Review it and give me the corrected query.

```sql
SELECT c.region,
       SUM(o.total_amount) AS revenue,
       COUNT(oi.product_id) AS items
FROM orders o
JOIN order_items oi ON oi.order_id = o.id
LEFT JOIN customers c ON c.id = o.customer_id
WHERE c.status = 'active'
  AND o.created_at BETWEEN '2026-09-01' AND '2026-09-30 23:59:59'
GROUP BY c.region;
```

`orders` has one row per order, `order_items` one row per product in an order, `customers` one row per customer. `created_at` is `timestamptz`.
