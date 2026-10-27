---
name: dax-measures
description: Three DAX measures with IFERROR division, FILTER(ALL()) as a CALCULATE argument, and time intelligence on a fact date column.
tags: [power-bi, smoke]
runs: 1
max_turns: 8
allowed_tools: [Skill, Read]
---
Our Power BI report is slow, and users say "Red Revenue" ignores the product category slicer. The model has a `Sales` fact (with `Sales[OrderDate]`), a `Product` dimension, and a `Date` table marked as a date table and related to `Sales[OrderDate]`. Review these measures and give me fixed versions.

```dax
Margin % = IFERROR(SUM(Sales[Margin]) / SUM(Sales[Revenue]), 0)

Red Revenue = CALCULATE([Revenue], FILTER(ALL(Product), Product[Color] = "Red"))

Revenue PY = CALCULATE([Revenue], SAMEPERIODLASTYEAR(Sales[OrderDate]))
```
