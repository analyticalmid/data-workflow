---
name: etl-idempotency
description: A daily Airflow load that duplicates on rerun, loses updates, and reads "now".
tags: [etl, smoke]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
Review this daily Airflow task that loads orders from our Postgres into the warehouse table `silver.orders`. It runs at 02:00 for the previous day.

```python
from datetime import datetime, timedelta

def load_orders(**context):
    yesterday = (datetime.now() - timedelta(days=1)).date()
    rows = postgres.fetch(
        "SELECT * FROM orders WHERE created_at::date = %s", (yesterday,)
    )
    warehouse.execute_many(
        "INSERT INTO silver.orders VALUES (%s, %s, %s, %s, %s)", rows
    )
```

Orders can change status and amount for up to 30 days after they are created. Last week the task failed halfway and we cleared it in the Airflow UI to rerun it.
