---
type: llm
weight: 2
---
PASS if the response does all of the following:
- Explains that joining `order_items` repeats each order once per item, so `SUM(o.total_amount)` counts an order's amount several times, and fixes it by aggregating `order_items` per order before the join (or computing revenue without the item join).
- Notes that `WHERE c.status = 'active'` on the `LEFT JOIN`ed table turns it into an inner join, and makes the intent explicit (move the condition into `ON`, or use an inner join on purpose).
- Replaces the `BETWEEN … '23:59:59'` range with a half-open range (`>= '2026-09-01' AND < '2026-10-01'`) and mentions the time zone the month is cut in.
- Ends with a corrected query in full.

FAIL if it misses the fan-out, or if it "fixes" the revenue with `DISTINCT` inside the `SUM`.
