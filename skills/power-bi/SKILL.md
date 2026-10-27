---
name: power-bi
description: Power BI semantic models, DAX, and Power Query M — star schema, relationships and filter direction, date tables, storage modes (Import, DirectQuery, Dual, Direct Lake), RLS, VertiPaq memory, measures and filter context, CALCULATE, time intelligence, query folding, and incremental refresh. Use when building or reviewing a Power BI or Fabric model, writing or fixing a DAX measure, writing Power Query, or when a report is slow, a total is wrong, or a refresh is long.
---

# Power BI

A Power BI number is right when the model's shape makes the measure simple. Fix the model before the DAX, and the
Power Query before the model.

## Steps

1. **Read the model**: tables, relationships with cardinality and filter direction, storage mode per table, the date
   table, and RLS roles. Done when every table is classed as a fact or a dimension, with its grain.
2. **Fix the shape** per [Model](#model) before touching a measure that fights it.
3. **Write or review the DAX** with [`references/dax.md`](references/dax.md).
4. **Write or review Power Query** with [`references/power-query.md`](references/power-query.md).
5. **Measure** with Performance Analyzer: each visual's time split into DAX query, Direct query, and visual display;
   copy the query and run it in DAX query view. A change is kept when that time drops and the numbers stay the same.
   https://learn.microsoft.com/en-us/power-bi/create-reports/desktop-performance-analyzer

## Model

- **Star schema.** Every table is a dimension (filters and groups) or a fact (summarized) at one consistent grain; the
  one side of a relationship is a dimension, the many side a fact. Flatten snowflaked dimensions into one table.
  https://learn.microsoft.com/en-us/power-bi/guidance/star-schema
- **Single filter direction.** Bidirectional relationships slow queries and produce surprising totals. For slicers that
  show only items with data, use a visual-level filter "measure is not blank"; for one measure that needs both
  directions, `CROSSFILTER(…, BOTH)` inside it. One-to-one relationships are bidirectional, so merge those tables.
  https://learn.microsoft.com/en-us/power-bi/guidance/relationships-bidirectional-filtering
- **Many-to-many**: between dimensions, a bridge table with one-to-many relationships on both sides (one set to both
  directions), hidden; between facts, never a direct relationship, only shared dimensions; a fact at a coarser grain
  than a dimension relates on the period's first date and returns BLANK below its grain.
  https://learn.microsoft.com/en-us/power-bi/guidance/relationships-many-to-many
- **Role-playing dates** (order date, ship date): one active relationship plus `USERELATIONSHIP` in the measure, or one
  date table per role with renamed columns.
- **A real date table**, marked as such: unique, contiguous dates, no blanks, covering every fact date. Turn off the
  global Auto date/time option: it adds a hidden table per date column, grows the model, and filters one fact at a
  time. https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-date-tables ·
  https://learn.microsoft.com/en-us/power-bi/guidance/auto-date-time
- **Memory**: VertiPaq compresses about 10x and pays per distinct value. Remove unused columns and rows, convert
  high-cardinality text codes to integers, split a datetime into a date and a time rounded to the second, and do all
  of it in Power Query or the source rather than in calculated columns.
  https://learn.microsoft.com/en-us/power-bi/guidance/import-modeling-data-reduction ·
  https://www.sqlbi.com/articles/optimizing-high-cardinality-columns-in-vertipaq/
- **Storage modes**:
  - **Import**: an in-memory snapshot, fastest, needs refresh.
  - **DirectQuery**: queries the source per visual; keep its measures and Power Query simple, set Assume Referential
    Integrity, and limit visuals per page.
  - **Dual**: dimensions serve both modes.
  - **Direct Lake** (Fabric): reads Delta tables in OneLake with Import-like speed, within per-capacity guardrails.
  - https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-storage-mode ·
    https://learn.microsoft.com/en-us/power-bi/guidance/directquery-model-guidance ·
    https://learn.microsoft.com/en-us/fabric/fundamentals/direct-lake-overview
- **RLS** filters rows of the tables a rule touches and propagates through active relationships only:
  - Put rules on dimensions.
  - Roles add up (a union), so one broad role opens everything.
  - Dynamic rules return no rows for an unexpected user.
  - Test with View As.
  - RLS binds only users with Read permission on the semantic model.
  - https://learn.microsoft.com/en-us/power-bi/guidance/rls-guidance
