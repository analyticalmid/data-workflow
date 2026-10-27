---
name: pandas-merge
description: A pandas 3 script with a fan-out merge, a chained assignment that does nothing, and a groupby that drops missing keys.
tags: [python, smoke]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
We are on pandas 3.1. The revenue per segment from this script does not match the finance system: some segments are too high and the cancelled orders still count. Review it and give me the corrected code.

```python
import pandas as pd

orders = pd.read_csv("orders.csv")
customers = pd.read_csv("customers.csv")

df = orders.merge(customers, on="customer_id", how="left")
df[df["status"] == "cancelled"]["amount"] = 0
totals = df.groupby("segment")["amount"].sum()
print(totals)
```

`orders.csv` has one row per order. `customers.csv` keeps one row per customer per address change, so a customer who moved appears more than once. Some orders belong to customers missing from `customers.csv`.
