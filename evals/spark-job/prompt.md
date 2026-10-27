---
name: spark-job
description: A slow Databricks PySpark job with a driver collect, a Python UDF, coalesce(1), and partitioning on a small table.
tags: [spark, smoke]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
This PySpark job on Databricks (DBR 18 LTS) takes 3 hours and sometimes dies with driver out-of-memory. `events` is a 400 GB Delta table; `countries` has 250 rows. Review it and give me a faster version.

```python
from pyspark.sql import functions as F
from pyspark.sql.types import StringType

events = spark.table("main.raw.events").cache()
countries = {row["code"]: row["name"] for row in spark.table("main.ref.countries").collect()}

@F.udf(StringType())
def country_name(code):
    return countries.get(code, "unknown")

@F.udf(StringType())
def normalize_email(email):
    return email.strip().lower() if email else None

enriched = (
    events
    .withColumn("country", country_name("country_code"))
    .withColumn("email", normalize_email("email"))
)

ids = [row["user_id"] for row in enriched.select("user_id").distinct().collect()]
active = enriched.filter(F.col("user_id").isin(ids))

(active.coalesce(1)
    .write.mode("overwrite")
    .partitionBy("event_date", "user_id")
    .saveAsTable("main.silver.events_enriched"))
```
