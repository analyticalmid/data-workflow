---
name: pandas-3-upgrade
description: A cleaning step that silently changes behaviour after upgrading from pandas 2.2 to 3.1.
tags: [python]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
We upgraded from pandas 2.2 to pandas 3.1. This cleaning step runs without errors, but the downstream system says the data is wrong now. Review it and give me a version that is correct on pandas 3.1.

```python
import pandas as pd

df = pd.read_csv("events.csv")
df["created_at"] = pd.to_datetime(df["created_at"], format="%Y-%m-%d %H:%M:%S")
df["epoch_ns"] = df["created_at"].astype("int64")

text_columns = [column for column in df.columns if df[column].dtype == object]
for column in text_columns:
    df[column] = df[column].str.strip()

df["status"].replace({"canceled": "cancelled"}, inplace=True)

monthly = df.set_index("created_at").resample("M")["amount"].sum()
df.to_parquet("events_clean.parquet")
```

The downstream system reads `epoch_ns` as nanoseconds since the epoch.
