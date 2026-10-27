# Dimensional modeling

Kimball Group techniques; each link below sits under
https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/.

## Order of decisions

1. **Business process**: one fact table per process (orders, payments, shipments), never one per report.
2. **Grain**: declared before any dimension or fact, at the most atomic level the source supports. Different grains never
   share a fact table. (`grain/`)
3. **Dimensions**: the who, what, where, when, and why that describe each fact row.
4. **Facts**: the numeric measurements true at that grain.

## Fact tables

| Type | One row per | Example |
| --- | --- | --- |
| Transaction | measurement event (`transaction-fact-table/`) | an order line |
| Periodic snapshot | entity per period (`periodic-snapshot-fact-table/`) | account balance per day |
| Accumulating snapshot | pipeline item, updated as it moves, one date key per milestone (`accumulating-snapshot-fact-table/`) | an order from placed to delivered |
| Factless | event or coverage with no measure (`factless-fact-table/`) | a student attending a class; products on promotion that did not sell |

- **Additivity** (`additive-semi-additive-non-additive-fact/`): additive facts sum across every dimension (amount,
  quantity); semi-additive ones not across time (balances, inventory: take the period's last value or the average);
  non-additive ones never (ratios, percentages, unit prices): store the additive parts and divide at query time.

## Dimensions

- **Conformed dimensions** share names and contents across fact tables, which is what lets one report combine processes.
  (`conformed-dimension/`)
- **Surrogate keys**: integer keys owned by the warehouse; natural keys repeat under SCD 2 and collide across sources.
  (`dimension-surrogate-key/`)
- **Degenerate dimension**: an identifier with no attributes (invoice number) kept in the fact table.
  (`degenerate-dimension/`)
- **Junk dimension**: low-cardinality flags and codes bundled into one dimension holding only the combinations that occur.
  (`junk-dimension/`)
- **Bridge table**: a many-to-many between a fact and a dimension (several diagnoses per visit) through a group key; with
  SCD 2 it needs effective and expiry dates. (`multivalued-dimension-bridge-table/`)

## Slowly changing dimensions

| Type | Change handling | Use when |
| --- | --- | --- |
| 0 | never changes (`type-0/`) | original values such as the signup date |
| 1 | overwrite; history is lost and aggregates over the attribute must be recomputed (`type-1/`) | corrections, attributes nobody analyses historically |
| 2 | new row with effective date, expiry date, and current flag (`type-2/`) | history matters: facts keep the attribute value true when they happened |
| 3 | an extra column for the previous value (`type-3/`) | rare: one alternate reality, such as an old and new region map |
| 4 | rapidly changing attributes in a mini-dimension, with both keys on the fact (`type-4-mini-dimension/`) | attributes that change too often for type 2 |
| 6 | type 2 rows plus type 1 current-value columns (`type-6/`) | reports need both "as it was" and "as it is" |

## Late arrivals

- **Late dimension**: insert a placeholder row with the natural key and "unknown" attributes so facts can load, then
  overwrite it when the real row arrives; a retroactive type 2 change inserts the row and restates the affected facts.
  (`late-arriving-dimension/`)
- **Late fact**: look up the dimension row that was effective at the event time, not the current one.
  (`late-arriving-fact/`)
