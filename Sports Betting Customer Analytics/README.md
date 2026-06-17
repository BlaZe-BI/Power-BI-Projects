# Sports Betting Customer Value Analytics — Project Documentation

**Author:** Željko Blagojević
**Tool:** Microsoft Power BI Desktop
**Source Files:** `client_deposits_.csv`, `client_withdrawals_.csv`, `client_activity_.csv` + live FX rates (Frankfurter API)
**Data Period (current load):** May 2025 – June 2025 (2 monthly snapshots)
**Documentation Version:** 1.0 *(generated directly from the live Power BI semantic model via MCP connection)*

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Source Data](#2-source-data)
3. [ETL Process — Power Query](#3-etl-process--power-query)
4. [Data Model](#4-data-model)
5. [Date Table](#5-date-table)
6. [DAX Measures](#6-dax-measures)
7. [Report Pages](#7-report-pages)
8. [Known Issues & Data Considerations](#8-known-issues--data-considerations)

---

## 1. Project Overview

This model implements a **Customer Value Analytics** layer for a sports betting operator, built around a monthly client-level fact table. The pipeline ingests three raw transactional CSV sources (deposits, withdrawals, daily activity), aggregates them to a **client × month** grain, enriches deposit amounts with a live EUR→USD exchange rate, and classifies every client-month into a **deposit group**, an **activity group**, and a derived **value segment** (e.g. VIP Premium, Gold, Silver, Bronze, No deposit player, Unsegmented).

The semantic model is designed for executive KPI reporting (deposits, withdrawals, profit, profit margin) and behavioral segmentation analysis, with built-in **currency switching (EUR/USD)** via a disconnected `Currency` table consumed inside the core measures.

---

## 2. Source Data

### 2.1 Source Files

- **`client_deposits_.csv`** — raw deposit transactions (CSV, comma-delimited, code page 1252)
- **`client_withdrawals_.csv`** — raw withdrawal transactions (CSV, comma-delimited, code page 1252)
- **`client_activity_.csv`** — raw daily activity log (CSV, comma-delimited, code page 1252)
- **FX rates** — fetched live from `https://api.frankfurter.dev/v2/rates` (EUR base, USD quote)

### 2.2 `client_deposits_.csv` (via `stg_client_deposits`)

| Column | Data Type | Description |
|---|---|---|
| `client_id` | Integer | Foreign key to client |
| `deposit_date` | Date | Date of the deposit |
| `deposit_amount_eur` | Currency | Deposit amount in EUR |

Enriched in staging with a left join to `fx_rates` on `deposit_date` = `rate_date`, producing a derived `deposit_amount_usd` column (`deposit_amount_eur × EUR_USD_rate`).

### 2.3 `client_withdrawals_.csv` (via `stg_client_withdrawals`)

| Column | Data Type | Description |
|---|---|---|
| `client_id` | Integer | Foreign key to client |
| `withdrawal_date` | Date | Date of the withdrawal |
| `withdrawal_amount_eur` | Currency | Withdrawal amount in EUR |

Same FX enrichment pattern as deposits: left join to `fx_rates` on `withdrawal_date`, plus a calculated `Start of Month` column and a derived `withdrawal_amount_usd`.

### 2.4 `client_activity_.csv` (via `stg_client_activity`)

| Column | Data Type | Description |
|---|---|---|
| `client_id` | Integer | Foreign key to client |
| `activity_date` | Date | Date of recorded activity |

No FX enrichment needed — used purely to count active days per client per month.

### 2.5 FX Rates (`fx_rates`)

Pulled live via `Web.Contents` from the Frankfurter API (`base=EUR`, `quotes=USD`, date range 2025‑05‑01 to 2025‑06‑30). Columns renamed to `rate_date` and `EUR_USD_rate`; `base`/`quote` columns dropped after expansion.

> **Note:** the FX API call is currently hardcoded to the 2025‑05‑01 → 2025‑06‑30 window, which matches the current data period but is **not dynamic** relative to the source data's actual date range.

---

## 3. ETL Process — Power Query

The ETL layer is organised into **3 layers**: staging (raw ingestion), monthly aggregation (transform), and final model tables.

### 3.1 Query Overview

```
Queries
├── fx_rates                    ← Source: live FX rate fetch (Frankfurter API)
├── stg_client_deposits         ← Stage: raw deposits + FX enrichment
├── stg_client_withdrawals      ← Stage: raw withdrawals + FX enrichment + Start of Month
├── stg_client_activity         ← Stage: raw daily activity
├── monthly_client_deposits     ← Transform: deposits grouped to client × month (EUR + USD)
├── monthly_client_withdrawals  ← Transform: withdrawals grouped to client × month (EUR + USD)
├── monthly_client_activity     ← Transform: distinct active days grouped to client × month
├── fact_client_month           ← Final: combined fact table (client × month grain)
├── dim_client                  ← Final: client dimension (distinct client_id)
├── dim_deposit_group           ← Final: static deposit-group lookup (D1–D5)
├── dim_activity_group          ← Final: static activity-group lookup (A1–A6)
├── dim_client_segment          ← Final: static segment lookup (7 segments, with sort order)
├── Currency                    ← Final: disconnected EUR/USD selector table
└── __Measures                  ← DAX measure container (empty table)
```

### 3.2 `stg_client_deposits` — Stage Query

```m
let
    Source = Csv.Document(File.Contents("client_deposits_.csv"),
        [Delimiter=",", Columns=3, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",
        {{"client_id", Int64.Type}, {"deposit_date", type date}, {"deposit_amount_eur", type number}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type",
        {{"deposit_amount_eur", Currency.Type}}),
    #"Merged Queries" = Table.NestedJoin(#"Changed Type1", {"deposit_date"}, fx_rates, {"rate_date"}, "fx_rates", JoinKind.LeftOuter),
    #"Expanded fx_rates" = Table.ExpandTableColumn(#"Merged Queries", "fx_rates", {"EUR_USD_rate"}, {"EUR_USD_rate"}),
    #"Added Custom" = Table.AddColumn(#"Expanded fx_rates", "deposit_amount_usd", each [deposit_amount_eur]*[EUR_USD_rate]),
    #"Changed Type2" = Table.TransformColumnTypes(#"Added Custom", {{"deposit_amount_usd", Currency.Type}})
in
    #"Changed Type2"
```

### 3.3 `stg_client_withdrawals` — Stage Query

Identical FX-join pattern to deposits, with an additional `Start of Month` column inserted ahead of the downstream aggregation step:

```m
let
    Source = Csv.Document(File.Contents("client_withdrawals_.csv"),
        [Delimiter=",", Columns=3, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",
        {{"client_id", Int64.Type}, {"withdrawal_date", type date}, {"withdrawal_amount_eur", type number}}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type", {{"withdrawal_amount_eur", Currency.Type}}),
    #"Merged Queries" = Table.NestedJoin(#"Changed Type1", {"withdrawal_date"}, fx_rates, {"rate_date"}, "fx_rates", JoinKind.LeftOuter),
    #"Expanded fx_rates" = Table.ExpandTableColumn(#"Merged Queries", "fx_rates", {"EUR_USD_rate"}, {"EUR_USD_rate"}),
    #"Inserted Start of Month" = Table.AddColumn(#"Expanded fx_rates", "Start of Month", each Date.StartOfMonth([withdrawal_date]), type date),
    #"Added Custom" = Table.AddColumn(#"Inserted Start of Month", "withdrawal_amount_usd", each [withdrawal_amount_eur]*[EUR_USD_rate]),
    #"Changed Type2" = Table.TransformColumnTypes(#"Added Custom", {{"withdrawal_amount_usd", Currency.Type}})
in
    #"Changed Type2"
```

### 3.4 `stg_client_activity` — Stage Query

```m
let
    Source = Csv.Document(File.Contents("client_activity_.csv"),
        [Delimiter=",", Columns=2, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",
        {{"client_id", Int64.Type}, {"activity_date", type date}})
in
    #"Changed Type"
```

### 3.5 `monthly_client_deposits` — Aggregation to Client × Month

```m
let
    Source = stg_client_deposits,
    #"Inserted Start of Month" = Table.AddColumn(Source, "Start of Month", each Date.StartOfMonth([deposit_date]), type date),
    #"Renamed Columns" = Table.RenameColumns(#"Inserted Start of Month", {{"Start of Month", "month_start"}}),
    #"Grouped Rows" = Table.Group(#"Renamed Columns", {"client_id", "month_start"}, {
        {"monthly_deposit_amount_eur", each List.Sum([deposit_amount_eur]), type nullable number},
        {"monthly_deposit_amount_usd", each List.Sum([deposit_amount_usd]), type nullable number}
    }),
    #"Changed Type" = Table.TransformColumnTypes(#"Grouped Rows",
        {{"monthly_deposit_amount_eur", Currency.Type}, {"monthly_deposit_amount_usd", Currency.Type}})
in
    #"Changed Type"
```

### 3.6 `monthly_client_withdrawals` — Aggregation to Client × Month

```m
let
    Source = stg_client_withdrawals,
    #"Renamed Columns" = Table.RenameColumns(Source, {{"Start of Month", "month_start"}}),
    #"Grouped Rows" = Table.Group(#"Renamed Columns", {"client_id", "month_start"}, {
        {"monthly_withdrawal_amount_eur", each List.Sum([withdrawal_amount_eur]), type nullable number},
        {"monthly_withdrawal_amount_usd", each List.Sum([withdrawal_amount_usd]), type nullable number}
    }),
    #"Changed Type" = Table.TransformColumnTypes(#"Grouped Rows",
        {{"monthly_withdrawal_amount_usd", Currency.Type}, {"monthly_withdrawal_amount_eur", Currency.Type}})
in
    #"Changed Type"
```

### 3.7 `monthly_client_activity` — Active Days per Client × Month

```m
let
    Source = stg_client_activity,
    #"Inserted Start of Month" = Table.AddColumn(Source, "Start of Month", each Date.StartOfMonth([activity_date]), type date),
    #"Renamed Columns" = Table.RenameColumns(#"Inserted Start of Month", {{"Start of Month", "month_start"}}),
    #"Removed Duplicates" = Table.Distinct(#"Renamed Columns", {"client_id", "activity_date"}),
    #"Grouped Rows" = Table.Group(#"Removed Duplicates", {"client_id", "month_start"}, {
        {"active_days", each Table.RowCount(_), Int64.Type}
    }),
    #"Renamed Columns1" = Table.RenameColumns(#"Grouped Rows", {{"active_days", "monthly_active_days"}})
in
    #"Renamed Columns1"
```

### 3.8 `fact_client_month` — Final Fact Table

Combines all three monthly aggregates on the `client_id` + `month_start` key (left-outer merges), fills missing activity with 0, then derives **`deposit_group`**, **`activity_group`**, and **`client_segment`** via nested conditional logic:

```m
let
    Source = Table.Combine({monthly_client_deposits, monthly_client_withdrawals, monthly_client_activity}),
    #"Removed Other Columns" = Table.SelectColumns(Source,{"client_id", "month_start"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Other Columns"),
    #"Merged Queries" = Table.NestedJoin(#"Removed Duplicates", {"client_id", "month_start"}, monthly_client_deposits, {"client_id", "month_start"}, "monthly_client_deposits", JoinKind.LeftOuter),
    #"Expanded monthly_client_deposits" = Table.ExpandTableColumn(#"Merged Queries", "monthly_client_deposits", {"monthly_deposit_amount_eur", "monthly_deposit_amount_usd"}, {"monthly_client_deposits.monthly_deposit_amount_eur", "monthly_client_deposits.monthly_deposit_amount_usd"}),
    #"Renamed Columns2" = Table.RenameColumns(#"Expanded monthly_client_deposits",{{"monthly_client_deposits.monthly_deposit_amount_eur", "monthly_deposit_amount_eur"}, {"monthly_client_deposits.monthly_deposit_amount_usd", "monthly_deposit_amount_usd"}}),
    #"Merged Queries1" = Table.NestedJoin(#"Renamed Columns2", {"client_id", "month_start"}, monthly_client_withdrawals, {"client_id", "month_start"}, "monthly_client_withdrawals", JoinKind.LeftOuter),
    #"Expanded monthly_client_withdrawals" = Table.ExpandTableColumn(#"Merged Queries1", "monthly_client_withdrawals", {"monthly_withdrawal_amount_eur", "monthly_withdrawal_amount_usd"}, {"monthly_withdrawal_amount_eur", "monthly_withdrawal_amount_usd"}),
    #"Merged Queries2" = Table.NestedJoin(#"Expanded monthly_client_withdrawals", {"client_id", "month_start"}, monthly_client_activity, {"client_id", "month_start"}, "monthly_client_activity", JoinKind.LeftOuter),
    #"Expanded monthly_client_activity" = Table.ExpandTableColumn(#"Merged Queries2", "monthly_client_activity", {"monthly_active_days"}, {"monthly_active_days"}),
    #"Replaced Value" = Table.ReplaceValue(#"Expanded monthly_client_activity",null,0,Replacer.ReplaceValue,{"monthly_active_days"}),
    #"Changed Type6" = Table.TransformColumnTypes(#"Replaced Value",{{"monthly_active_days", Int64.Type}}),
    #"Sorted Rows" = Table.Sort(#"Changed Type6",{{"client_id", Order.Ascending}, {"month_start", Order.Ascending}}),
    #"Added Conditional Column" = Table.AddColumn(#"Sorted Rows", "deposit_group", each
        if [monthly_deposit_amount_eur] = 0 then "D1"
        else if [monthly_deposit_amount_eur] <= 100 then "D2"
        else if [monthly_deposit_amount_eur] <= 500 then "D3"
        else if [monthly_deposit_amount_eur] <= 1000 then "D4"
        else "D5"),
    #"Changed Type3" = Table.TransformColumnTypes(#"Added Conditional Column",{{"deposit_group", type text}}),
    #"Added Conditional Column1" = Table.AddColumn(#"Changed Type3", "activity_group", each
        if [monthly_active_days] = 0 then "A1"
        else if [monthly_active_days] <= 2 then "A2"
        else if [monthly_active_days] <= 5 then "A3"
        else if [monthly_active_days] <= 10 then "A4"
        else if [monthly_active_days] <= 20 then "A5"
        else "A6"),
    #"Changed Type4" = Table.TransformColumnTypes(#"Added Conditional Column1",{{"activity_group", type text}}),
    #"Added Custom" = Table.AddColumn(#"Changed Type4", "client_segment", each
        if [deposit_group] = "D1" then "No deposit player"
        else if [deposit_group] = "D2" and List.Contains({"A1","A2"}, [activity_group]) then "Bronze"
        else if (
                (List.Contains({"D2","D3"}, [deposit_group]) and List.Contains({"A2","A3","A4","A5"}, [activity_group]))
                or ([deposit_group] = "D4" and [activity_group] = "A3")
            ) then "Silver"
        else if (
                ([deposit_group] = "D4" and List.Contains({"A4","A5"}, [activity_group]))
                or ([deposit_group] = "D5" and List.Contains({"A2","A3","A4"}, [activity_group]))
            ) then "Gold"
        else if [deposit_group] = "D5" and [activity_group] = "A5" then "VIP"
        else if [deposit_group] = "D5" and [activity_group] = "A6" then "VIP Premium"
        else "Unsegmented"),
    #"Changed Type5" = Table.TransformColumnTypes(#"Added Custom",{{"client_segment", type text}})
in
    #"Changed Type5"
```

**Segmentation logic summary:**

| Deposit Group | Range (EUR, monthly) |
|---|---|
| D1 | 0 (no deposit) |
| D2 | ≤ 100 |
| D3 | ≤ 500 |
| D4 | ≤ 1,000 |
| D5 | > 1,000 |

| Activity Group | Active Days (monthly) |
|---|---|
| A1 | 0 |
| A2 | ≤ 2 |
| A3 | ≤ 5 |
| A4 | ≤ 10 |
| A5 | ≤ 20 |
| A6 | > 20 |

Combining deposit group × activity group yields one of 7 segments: **No deposit player, Bronze, Silver, Gold, VIP, VIP Premium, Unsegmented** (the last being a catch-all for combinations not explicitly mapped above).

### 3.9 `dim_client`, `dim_deposit_group`, `dim_activity_group`, `dim_client_segment`, `Currency`

- **`dim_client`** — distinct `client_id` extracted from `fact_client_month`.
- **`dim_deposit_group`** — static lookup table: `D1`–`D5`.
- **`dim_activity_group`** — static lookup table: `A1`–`A6`.
- **`dim_client_segment`** — static lookup table with a `segment_number` sort column (`VIP Premium`=1 → `Unsegmented`=7), used to control display order independent of alphabetical sorting.
- **`Currency`** — disconnected single-column table (`EUR`, `USD`) used as a slicer to drive the EUR/USD `SWITCH` logic inside the core measures.

---

## 4. Data Model

**Tables:** `fact_client_month` (fact), `dim_client`, `dim_date` (calculated), `dim_deposit_group`, `dim_activity_group`, `dim_client_segment`, `Currency` (disconnected), `__Measures` (measure container, no columns).

**Relationships (5, all single-direction, Many-to-One, active):**

| From Table | From Column | To Table | To Column |
|---|---|---|---|
| `fact_client_month` | `client_id` | `dim_client` | `client_id` |
| `fact_client_month` | `month_start` | `dim_date` | `Date` |
| `fact_client_month` | `deposit_group` | `dim_deposit_group` | `deposit_group` |
| `fact_client_month` | `activity_group` | `dim_activity_group` | `activity_group` |
| `fact_client_month` | `client_segment` | `dim_client_segment` | `client_segment` |

The model follows a star schema centered on `fact_client_month`. The `Currency` table has **no relationship** to any other table by design — it is a disconnected parameter table read exclusively via `SELECTEDVALUE()` inside measures.

---

## 5. Date Table

`dim_date` is a **calculated table** (DAX, not Power Query), dynamically bounded by the min/max of `fact_client_month[month_start]`:

```dax
VAR MinDate = MIN(fact_client_month[month_start])
VAR MaxDate = MAX(fact_client_month[month_start])
RETURN
ADDCOLUMNS(
    CALENDAR(MinDate, MaxDate),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "Year-Month", FORMAT([Date], "yyyy-MM")
)
```

| Column | Data Type | Description |
|---|---|---|
| `Date` | DateTime | Calendar date (one row per day) |
| `Year` | Integer | Calendar year |
| `Month Number` | Integer | Month number (1–12) |
| `Month` | Text | Full month name |
| `Year-Month` | Text | `yyyy-MM` formatted period label |

> **Note:** the relationship to `fact_client_month` is on `month_start` (always the 1st of the month), so although `dim_date` contains a full daily calendar, only the first day of each month carries fact data — day-level granularity in the date table currently has no matching detail in the fact table.

---

## 6. DAX Measures

All measures live in the `__Measures` display table, organised into **3 display folders**: Executive, Segmentation, Behavior.

### 6.1 Executive

| Measure | Expression | Format |
|---|---|---|
| `Total Deposits` | `SWITCH(SELECTEDVALUE('Currency'[Currency]), "EUR", SUM(fact_client_month[monthly_deposit_amount_eur]), "USD", SUM(fact_client_month[monthly_deposit_amount_usd]))` | `#,0.00` |
| `Total Withdrawals` | `SWITCH(SELECTEDVALUE('Currency'[Currency]), "EUR", SUM(fact_client_month[monthly_withdrawal_amount_eur]), "USD", SUM(fact_client_month[monthly_withdrawal_amount_usd]))` | `#,0.00` |
| `Profit` | `[Total Deposits] - [Total Withdrawals]` | `#,0.00` |
| `Client count` | `DISTINCTCOUNT(fact_client_month[client_id])` | `0` |
| `Profit margin` | `DIVIDE([Profit], [Total Deposits])` | `0.0%;-0.0%;0.0%` |

The currency-aware pattern (`Total Deposits` / `Total Withdrawals`) is the foundation of the whole measure layer: every downstream measure that touches money inherits EUR/USD switching automatically by referencing these two base measures rather than the raw columns.

### 6.2 Segmentation

| Measure | Expression | Format |
|---|---|---|
| `Avg Active Days` | `AVERAGE(fact_client_month[monthly_active_days])` | General number |
| `Unsegmented Clients` | `CALCULATE([Client count], dim_client_segment[client_segment] = "Unsegmented")` | `0` |
| `Unsegmented %` | `DIVIDE([Unsegmented Clients], [Client count])` | `0.0%;-0.0%;0.0%` |
| `Client Count Heatmap` | `COALESCE([Client count], 0)` | `0` |

`Client Count Heatmap` is a null-safe wrapper around `[Client count]`, intended for matrix/heatmap visuals where empty deposit‑group × activity‑group combinations would otherwise render as blank instead of `0`.

### 6.3 Behavior

| Measure | Expression | Format |
|---|---|---|
| `Avg Deposit per Client` | `DIVIDE([Total Deposits], [Client count])` | `#,0.00` |
| `Avg Withdrawal per Client` | `DIVIDE([Total Withdrawals], [Client count])` | `#,0.00` |
| `Avg Profit per Client` | `DIVIDE([Profit], [Client count])` | `#,0.00` |

All three behavior measures are simple per-client averages built directly on top of the Executive base measures, preserving the EUR/USD switch automatically.

---

## 7. Report Pages

**Report pages:**

| Page | Purpose |
|---|---|
| Navigation Page | Landing/home screen with logo and section navigation buttons |
| Executive Overview | Top-line KPIs and deposit/profit trend by segment |
| Segmentation Analysis | Deposit × Activity group distribution and segment composition |
| Client Behavior / Advanced Analysis | Per-client deposit behavior and segment profit ranking over time |
| Task Info | Assignment brief reference page |

A consistent slicer panel appears on the left of every analytical page: **Currency** (EUR/USD), **Reporting Period** (2025‑05 / 2025‑06), **Client Segment** (multi-select checkboxes for all 7 segments), **Deposit Tier** (D1–D5), and **Player Activity** (A1–A6). A circular back-arrow button (top-right) returns to the Navigation Page from every other page.

### 7.1 Navigation Page

Dark charcoal background with a light-blue line-art sports logo (football, tennis racket, basketball) and the title "SPORTS BETTING". Four gold-bordered navigation buttons link to: Executive Overview, Segmentation Analysis, Client Behavior / Advanced Analysis, and Task Info. Subtitle credits "Assignment by Željko Blagojević".

### 7.2 Executive Overview

**KPI cards:** Total Deposits, Total Withdrawals, Profit, Client count, Profit margin — values shown for EUR, all periods/segments unfiltered.

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Total Deposits and Profit by Year-Month | Combo chart (column + line) | X: Year-Month (2025-05, 2025-06); Columns: `Total Deposits`; Line: `Profit` |
| Client count by Client Segment | Column chart | X: `client_segment` (VIP, Gold, Silver, Bronze, Unsegmented); Y: `Client count`. Color-coded per segment. |
| Profit by Client Segment | Horizontal bar chart | Segments ranked by `Profit` (Gold highest at 66K, then VIP 40K, Silver 7K, Bronze 2K, Unsegmented 1K) |
| Deposits and Profit margin by Segments | Matrix table | Rows: `client_segment`; Columns grouped by Year-Month (2025-05, 2025-06), each showing `Total Deposits` and `Profit margin`. Alternating row shading. |

### 7.3 Segmentation Analysis

**KPI cards:** Client count, Avg Active Days, Unsegmented Clients, Unsegmented % — filtered to Reporting Period 2025-05 in the screenshot.

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Client Distribution by Deposit and Activity Groups | Matrix/heatmap table | Rows: `deposit_group` (D1–D5); Columns: `activity_group` (A1–A6); values: client counts with color-intensity heatmap shading; row/column totals |
| Monthly Client Distribution by Segment | 100% stacked bar | Single bar per Year-Month showing % share of each `client_segment` (e.g. 2025-05: Gold 38%, Silver 27%, Bronze 25%, plus VIP/Unsegmented slivers) |
| Total Deposits and Total Withdrawals by Client Segment | Clustered column chart | X: `client_segment`; Y: `Total Deposits` and `Total Withdrawals` side by side per segment |
| Client count and Profit by Segments | Table | Columns: `client_segment`, `Client count`, `Profit`, filtered to the selected Year-Month |

### 7.4 Client Behavior / Advanced Analysis

**KPI cards:** Avg Deposit per Client, Avg Withdrawal per Client, Avg Profit per Client, Profit margin.

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Client Deposit Behavior by Activity Intensity | Bubble/scatter chart | X: `monthly_active_days`; Y: deposit amount; bubble size: deposit volume; color: `client_segment` (VIP, Gold, Silver, Bronze, Unsegmented) |
| Segment Profit Ranking by Month | Stacked area/ribbon chart | X: Year-Month (2025-05, 2025-06); stacked `Profit` by `client_segment`, showing Gold and VIP as dominant contributors growing month-over-month |

### 7.5 Task Info (Reference Page)

Static text panel reproducing the original assignment brief: build a Power BI report on monthly performance metrics per segmentation group from deposit/withdrawal/activity CSVs, covering monthly totals (deposits, withdrawals, profit), monthly client counts/percentages, deposit groups (D1–D5), activity groups (A1–A6), and combined client segmentation — with open guidance on visualization approach and an emphasis on usability and a clean data model.

---

## 8. Known Issues & Data Considerations

- **Limited data scope:** only **2 monthly snapshots** (May 2025 – June 2025) and **100 clients** / **200 fact rows** are currently loaded — this is a small working/sample dataset, not a full production load.
- **FX rate window is hardcoded** (`2025-05-01` to `2025-06-30`) inside the `fx_rates` query and will not automatically extend if the underlying CSV sources are refreshed with a wider date range.

---

*Documentation v1.0 — all tables, relationships, Power Query (M) and DAX expressions extracted directly from the live Power BI semantic model via MCP connection.*
*Prepared by Željko Blagojević, 2026.*
