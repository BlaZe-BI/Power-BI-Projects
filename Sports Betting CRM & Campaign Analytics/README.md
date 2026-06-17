#  Sports Betting CRM & Campaign Analytics Case Study — Project Documentation

**Author:** Željko Blagojević  
**Tool:** Microsoft Power BI Desktop (Version 2.153.910.0, April 2026)  
**Source File:** `PBi_task_2026_online.xlsx`  
**Report Date Range:** 2025-12-01 – 2026-02-28  
**Documentation Version:** 3.0 *(initially generated from the live Power BI model via MCP connection and subsequently reviewed, refined, and expanded manually)*

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Source Data](#2-source-data)
3. [ETL Process — Power Query](#3-etl-process--power-query)
4. [Data Model](#4-data-model)
5. [Date Table](#5-date-table)
6. [DAX Measures & Calculated Columns](#6-dax-measures--calculated-columns)
7. [Report Pages](#7-report-pages)
8. [Parameter Tables (Field Parameters)](#8-parameter-tables-field-parameters)
9. [Design & Theming](#9-design--theming)

---

## 1. Project Overview

This report was built as a structured interview assignment, covering seven analytical tasks. The goal was to demonstrate end-to-end Power BI development — from raw data ingestion through ETL, data modeling, DAX measure development, and report visualisation — all using a custom visual identity.

The report is structured as a navigable multi-page dashboard with a central navigation page and one dedicated page per task. Interactive slicers (Date, City, Gender, VIP) are consistent across most pages and allow cross-filtering of all visuals.

**Report pages:**

| Page | Purpose |
|---|---|
| Navigation Page | Landing/home screen with logo and task navigation buttons |
| Task 1 \| Task 2 | KPI overview: acquisition, deposits, betting activity, GGR |
| Task 3 | Active players, AVG Bet per Player, AVG GGR per Player with product breakdown |
| Task 4 | Player segmentation by deposit quartile |
| Task 5 | Daily Bet vs. 7-Day Moving Average |
| Task 6 | Dynamic metric selector with configurable time granularity |
| Task 7 | TV Draw promotion analysis with abusive behavior detection |
| TV Draw Promotion — Key Findings | Written analytical insights page (bookmark target) |
| Tasks Info | Task descriptions reference page (hidden from end users) |

---

## 2. Source Data

### 2.1 Source File

- **File:** `PBi_task_2026_online.xlsx`
- **Relevant sheets:** `client`, `transaction`, `tv_draw_promo`
- **Ignored sheets:** `Intro`, `Task` (task descriptions)

### 2.2 Sheet: `client`

Contains one row per registered player. **100 rows** (excluding header).

| Column | Data Type | Description |
|---|---|---|
| `client_code` | Integer | Unique player identifier (PK) |
| `client_registration_date` | DateTime | Date and time of player registration |
| `client_first_deposit_date` | DateTime / NULL | Date of first deposit; `[NULL]` string for players who never deposited |
| `VIP` | Text | VIP segment: `NE`, `PIP`, `VIP` |
| `city` | Text | Player's city: `Beograd`, `Niš`, `Novi Sad` |
| `gender` | Text | Player gender: `M`, `Ž` (raw source; corrected in ETL) |

**Data quality issues identified and resolved:**
- `client_first_deposit_date` contains the literal string `[NULL]` instead of a true null for 46 of 100 players — handled in Power Query.
- `gender` contains `Ž` (incorrect value for female) — corrected to `F` in Power Query.
- Text fields (`VIP`, `city`, `gender`) contain potential leading/trailing whitespace — trimmed and cleaned in Power Query.

### 2.3 Sheet: `transaction`

Contains one row per daily transaction summary per player. **4,546 rows** (excluding header).

| Column | Data Type | Description |
|---|---|---|
| `client_code` | Integer | Foreign key to `client` |
| `transaction_date` | Date | Date of the transaction |
| `Type` | Text | Transaction type: `Betting`, `Casino`, `Depozit` (raw; standardised in ETL) |
| `in` | Number | Amount going in (bet placed or deposit made) |
| `out` | Number | Amount going out (win paid or withdrawal made) |

**Date range:** 2025-12-01 to 2026-02-28

**Data quality issues identified and resolved:**
- `Type` contains `Depozit` (Serbian spelling) — replaced with `Deposit` in ETL.
- `Type` field may contain leading/trailing whitespace — trimmed and cleaned in Power Query.

### 2.4 Sheet: `tv_draw_promo`

Contains one row per player who opted in to the TV Draw promotion. **42 rows** (excluding header). Not all clients appear here — used as a left-join enrichment source.

| Column | Data Type | Description |
|---|---|---|
| `client_code` | Integer | Foreign key to `client` |
| `opt_in_date` | DateTime | Date and time player opted in |
| `tvdraw_tickets` | Integer | Number of TV Draw tickets awarded |
| `informed_about_promo` | Text | Communication channel: `e-mail`, `sms`, or NULL |

**Opt-in date range:** 2026-01-01 to 2026-01-30  
**Data quality issues resolved:** NULL in `informed_about_promo` replaced with `No Promo Info` in ETL.

---

## 3. ETL Process — Power Query

The ETL layer is built in Power Query (M language) and organised into **8 queries** grouped by function: stage, temp/intermediate, and final model tables.

### 3.1 Query Overview

```
Queries [8]
├── stg_client          ← Stage: raw client data
├── stg_transaction     ← Stage: raw transaction data
├── stg_tv_draw_promo   ← Stage: raw promo data
├── tmp_client          ← Temp: pass-through alias
├── tmp_promo           ← Temp: pass-through alias
├── fact_transaction    ← Final: cleaned transaction fact table
├── dim_client          ← Final: enriched client dimension (denormalized)
└── __Measures          ← DAX measure container (empty table)
```

### 3.2 `stg_client` — Stage Query

**Purpose:** Load and clean the raw `client` sheet.

```m
let
    Source = Excel.Workbook(File.Contents("...PBi_task_2026_online.xlsx"), null, true),
    client_Sheet = Source{[Item="client", Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(client_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"client_code", Int64.Type},
        {"client_registration_date", type datetime},
        {"client_first_deposit_date", type any},
        {"VIP", type text}, {"city", type text}, {"gender", type text}
    }),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type", {
        {"client_registration_date", type date}
    }),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type1",
        "[NULL]", null, Replacer.ReplaceValue, {"client_first_deposit_date"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Replaced Value", {
        {"client_first_deposit_date", type date}
    }),
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type2", {
        {"VIP", Text.Trim, type text},
        {"city", Text.Trim, type text},
        {"gender", Text.Trim, type text}
    }),
    #"Cleaned Text" = Table.TransformColumns(#"Trimmed Text", {
        {"VIP", Text.Clean, type text},
        {"city", Text.Clean, type text},
        {"gender", Text.Clean, type text}
    })
in
    #"Cleaned Text"
```

**Transformation steps:**

| Step | Action |
|---|---|
| Promote Headers | First row becomes column headers |
| Changed Type | Set `client_code` to Int64; `client_registration_date` as datetime; `client_first_deposit_date` as `any` (deferred — contains mixed types incl. `[NULL]` string) |
| Changed Type1 | Cast `client_registration_date` to `date` (drop time component) |
| Replaced Value | Replace literal string `"[NULL]"` with proper `null` in `client_first_deposit_date` |
| Changed Type2 | Cast `client_first_deposit_date` to `date` |
| Trimmed Text | Remove leading/trailing whitespace from `VIP`, `city`, `gender` |
| Cleaned Text | Remove non-printable control characters from `VIP`, `city`, `gender` |

---

### 3.3 `stg_transaction` — Stage Query

**Purpose:** Load and clean the raw `transaction` sheet.

```m
let
    Source = Excel.Workbook(File.Contents("...PBi_task_2026_online.xlsx"), null, true),
    transaction_Sheet = Source{[Item="transaction", Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(transaction_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"client_code", Int64.Type},
        {"transaction_date", type date},
        {"Type", type text},
        {"in", type number},
        {"out", type number}
    }),
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type", {{"Type", Text.Trim, type text}}),
    #"Cleaned Text" = Table.TransformColumns(#"Trimmed Text", {{"Type", Text.Clean, type text}})
in
    #"Cleaned Text"
```

**Transformation steps:**

| Step | Action |
|---|---|
| Promote Headers | First row becomes column headers |
| Changed Type | Set correct types: `client_code` Int64, `transaction_date` date, `Type` text, `in`/`out` number |
| Trimmed Text | Trim whitespace from `Type` |
| Cleaned Text | Remove non-printable characters from `Type` |

---

### 3.4 `stg_tv_draw_promo` — Stage Query

**Purpose:** Load and clean the raw `tv_draw_promo` sheet.

```m
let
    Source = Excel.Workbook(File.Contents("...PBi_task_2026_online.xlsx"), null, true),
    tv_draw_promo_Sheet = Source{[Item="tv_draw_promo", Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(tv_draw_promo_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {
        {"client_code", Int64.Type},
        {"opt_in_date", type datetime},
        {"tvdraw_tickets", Int64.Type},
        {"informed_about_promo", type text}
    }),
    #"Changed Type1" = Table.TransformColumnTypes(#"Changed Type", {
        {"opt_in_date", type date}
    }),
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type1", {
        {"informed_about_promo", Text.Trim, type text}
    }),
    #"Cleaned Text" = Table.TransformColumns(#"Trimmed Text", {
        {"informed_about_promo", Text.Clean, type text}
    })
in
    #"Cleaned Text"
```

**Transformation steps:**

| Step | Action |
|---|---|
| Promote Headers | First row becomes column headers |
| Changed Type | Cast `opt_in_date` as datetime, `tvdraw_tickets` as Int64 |
| Changed Type1 | Strip time component from `opt_in_date`, cast to date |
| Trimmed Text | Trim whitespace from `informed_about_promo` |
| Cleaned Text | Remove non-printable characters from `informed_about_promo` |

---

### 3.5 `tmp_client` and `tmp_promo` — Intermediate Queries

Pass-through aliases referencing the stage queries. They serve as a clean separation layer — if the stage logic changes, only the stage query needs updating without touching downstream transformation logic.

```m
-- tmp_client
let Source = stg_client in Source

-- tmp_promo
let Source = stg_tv_draw_promo in Source
```

---

### 3.6 `fact_transaction` — Final Fact Table

**Purpose:** Produce the final clean transaction table with standardised column names and transaction type values.

```m
let
    Source = stg_transaction,
    #"Renamed Columns" = Table.RenameColumns(Source, {
        {"Type", "transaction_type"},
        {"in", "amount_in"},
        {"out", "amount_out"}
    }),
    #"Filtered Rows" = Table.SelectRows(#"Renamed Columns", each true),
    #"Replaced Value" = Table.ReplaceValue(#"Filtered Rows",
        "Depozit", "Deposit", Replacer.ReplaceText, {"transaction_type"}),
    #"Filtered Rows1" = Table.SelectRows(#"Replaced Value", each true)
in
    #"Filtered Rows1"
```

**Transformation steps:**

| Step | Action |
|---|---|
| Renamed Columns | `Type` → `transaction_type`; `in` → `amount_in`; `out` → `amount_out` |
| Filtered Rows | Placeholder filter (no rows removed; reserved for future use) |
| Replaced Value | Standardise: `"Depozit"` → `"Deposit"` |
| Filtered Rows1 | Second placeholder filter |

**Final `fact_transaction` columns:**

| Column | Type | Description |
|---|---|---|
| `client_code` | Int64 | FK to `dim_client` |
| `transaction_date` | Date | Date of transaction |
| `transaction_type` | Text | `Betting`, `Casino`, `Deposit` |
| `amount_in` | Decimal | Inflow amount (bet or deposit) |
| `amount_out` | Decimal | Outflow amount (win or withdrawal) |

---

### 3.7 `dim_client` — Final Dimension Table (Denormalized)

**Purpose:** Single enriched client dimension produced by left-joining client data with promo data. Because the relationship between clients and promo is 1:1, denormalization is the correct modelling choice — it eliminates an unnecessary join and keeps the schema simple.

```m
let
    Source = Table.NestedJoin(tmp_client, {"client_code"},
                              tmp_promo,  {"client_code"},
                              "_promo", JoinKind.LeftOuter),
    #"Expanded _promo" = Table.ExpandTableColumn(Source, "_promo",
        {"opt_in_date", "tvdraw_tickets", "informed_about_promo"},
        {"opt_in_date", "tvdraw_tickets", "informed_about_promo"}),
    #"Replaced Value" = Table.ReplaceValue(#"Expanded _promo",
        "Ž", "F", Replacer.ReplaceText, {"gender"}),
    #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value",
        null, "No Promo Info", Replacer.ReplaceValue, {"informed_about_promo"}),
    #"Renamed Columns" = Table.RenameColumns(#"Replaced Value1", {
        {"informed_about_promo", "informed_about_promo"}
    })
in
    #"Renamed Columns"
```

**Transformation steps:**

| Step | Action |
|---|---|
| NestedJoin | Left outer join of `tmp_client` ← `tmp_promo` on `client_code` |
| Expand `_promo` | Flatten nested table: extract `opt_in_date`, `tvdraw_tickets`, `informed_about_promo` |
| Replaced Value | Fix data quality: `"Ž"` → `"F"` in `gender` |
| Replaced Value1 | Replace `null` with `"No Promo Info"` in `informed_about_promo` |
| Renamed Columns | No-op rename (step kept for traceability) |

**Final `dim_client` source columns (from Power Query):**

| Column | Type | Description |
|---|---|---|
| `client_code` | Int64 | PK — unique player identifier |
| `client_registration_date` | Date | Registration date |
| `client_first_deposit_date` | Date / null | First deposit date; null if no deposit made |
| `VIP` | Text | `NE`, `PIP`, `VIP` |
| `city` | Text | `Beograd`, `Niš`, `Novi Sad` |
| `gender` | Text | `M`, `F` (corrected from source `Ž`) |
| `opt_in_date` | Date / null | TV Draw opt-in date; null if not opted in |
| `tvdraw_tickets` | Int64 / null | Number of TV Draw tickets; null if not opted in |
| `informed_about_promo` | Text | `e-mail`, `sms`, `No Promo Info` |

---

### 3.8 `__Measures` — Measure Container

An empty table used as a centralised container for all DAX measures — a standard Power BI best practice that keeps measures organised and separate from the tables they reference.

---

## 4. Data Model

### 4.1 Schema Overview

The data model follows a **star schema** pattern:

```
dim_client (1) ──────────────── (*) fact_transaction        [active]
dim_client (*) ──────────────── (1) Date [registration_date] [inactive]
dim_client (*) ──────────────── (1) Date [first_deposit_date][inactive]
dim_client (*) ──────────────── (1) Date [opt_in_date]       [inactive]
fact_transaction (*) ─────────── (1) Date [transaction_date] [active]
```

### 4.2 Tables in the Model

| Table | Type | Description |
|---|---|---|
| `dim_client` | Dimension | 100 rows — enriched player dimension with promo data and calculated columns |
| `fact_transaction` | Fact | 4,546 rows — all transactions (betting, casino, deposits) |
| `Date` | Date/Calendar | Generated date dimension, marked as Date Table |
| `Selected Metric` | Field Parameter | Dynamic metric selector for Task 6 |
| `Time Granularity` | Field Parameter | Time granularity selector for Task 6 |
| `__Measures` | Measure container | 0 rows — holds all 31 DAX measures |

### 4.3 Relationships

*Extracted directly from live model via MCP:*

| From Table | From Column | Cardinality | To Table | To Column | Active | Cross-filter |
|---|---|---|---|---|---|---|
| `fact_transaction` | `client_code` | Many → One | `dim_client` | `client_code` | ✅ Yes | OneDirection |
| `fact_transaction` | `transaction_date` | Many → One | `Date` | `Date` | ✅ Yes | OneDirection |
| `dim_client` | `client_registration_date` | Many → One | `Date` | `Date` | ❌ No | OneDirection |
| `dim_client` | `client_first_deposit_date` | Many → One | `Date` | `Date` | ❌ No | OneDirection |
| `dim_client` | `opt_in_date` | Many → One | `Date` | `Date` | ❌ No | OneDirection |

> The three inactive relationships on `dim_client` are activated selectively via `USERELATIONSHIP()` inside specific DAX measures (NRC uses `client_registration_date`, FTD uses `client_first_deposit_date`). The `opt_in_date` relationship exists in the model but is not currently used via `USERELATIONSHIP` — promo filtering is handled directly through `dim_client[opt_in_date]` column filters.

### 4.4 Calculated Columns in `dim_client`

*All DAX expressions extracted directly from the live model:*

**`Total Deposit CC`** — Total deposit per player across all time. `COALESCE` returns 0 instead of BLANK for players without deposits. Used as the base for quartile segmentation.
```dax
Total Deposit CC =
COALESCE(
    CALCULATE(
        SUM(fact_transaction[amount_in]),
        fact_transaction[transaction_type] = "Deposit"
    ),
    0
)
```

**`Deposit Rank`** — Ranks players by total deposit (DESC). Uses a composite sort key (`Total Deposit CC * 1000000000 - client_code`) as a tiebreaker to ensure deterministic Dense ranking when deposits are equal.
```dax
Deposit Rank =
IF(
    dim_client[Total Deposit CC] > 0,
    RANKX(
        ALL(dim_client),
        'dim_client'[Total Deposit CC] * 1000000000
            - dim_client[client_code],
        ,
        DESC,
        Dense
    )
)
```

**`Deposit Segment`** — Assigns each depositing player to one of four equally-sized quartile groups using `CEILING` on the normalized rank ratio. Non-depositing players return BLANK.
```dax
Deposit Segment =
VAR DepositorCount =
    COUNTROWS(
        FILTER(
            ALL(dim_client),
            dim_client[Total Deposit CC] > 0
        )
    )
RETURN
IF(
    dim_client[Total Deposit CC] > 0,
    "G-" &
    CEILING(
        DIVIDE(dim_client[Deposit Rank] * 4, DepositorCount),
        1
    )
)
```

**`Top Segment`** — Simplified binary grouping used in the GGR contribution bar chart on Task 4.
```dax
Top Segment =
IF(
    dim_client[Deposit Segment] = "G-1",
    "Top (G-1)",
    "Others (G-2, G-3, G-4)"
)
```

**`Promo Participation`** — Text label for promo opt-in status. Used as a legend/axis dimension on Task 7 bar charts.
```dax
Promo Participation =
IF(
    NOT ISBLANK(dim_client[opt_in_date]),
    "Opted-in",
    "Non Opted-in"
)
```

**`Player Type`** — Classifies each player as New or Existing based on whether their registration date falls within the promotion period (Jan 1–30, 2026).
```dax
Player Type =
IF(
    dim_client[client_registration_date] >= DATE(2026, 1, 1)
        && dim_client[client_registration_date] <= DATE(2026, 1, 30),
    "New",
    "Existing"
)
```

**`Total Bet CC`** — Total bet amount per player (Betting + Casino). `COALESCE` returns 0 for players with no betting activity. Used in the Potential Abusive Behavior table on Task 7.
```dax
Total Bet CC =
COALESCE(
    CALCULATE(
        SUM(fact_transaction[amount_in]),
        KEEPFILTERS(fact_transaction[transaction_type] IN {"Casino", "Betting"})
    ),
    0
)
```

**`Total GGR CC`** — Total GGR per player (amount_in minus amount_out for Betting + Casino). Used alongside `Total Bet CC` for abusive behavior detection. Negative values indicate players who cost the operator money.
```dax
Total GGR CC =
COALESCE(
    CALCULATE(
        SUM(fact_transaction[amount_in]) - SUM(fact_transaction[amount_out]),
        KEEPFILTERS(fact_transaction[transaction_type] IN {"Casino", "Betting"})
    ),
    0
)
```

---

## 5. Date Table

The `Date` table is a custom-generated calendar table marked as a **Date Table** in the model, spanning the full range of dates in the transaction data (2025-12-01 to 2026-02-28).

### 5.1 Date Table Columns

*Extracted directly from live model:*

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | Primary key — marked as date key |
| `Year` | Integer | Calendar year |
| `Month Number` | Integer | Month number (1–12) |
| `Month` | Text | Month name — sorted by `Month Number` |
| `Quarter` | Text | Quarter label (Q1–Q4) |
| `Day` | Integer | Day of month (1–31) |
| `Day Name` | Text | Day name — sorted by `Day Number` |
| `ISO Week` | Integer | ISO week number |
| `Day Type` | Text | `Weekday` or `Weekend` |
| `Day Number` | Integer | ISO day number (1=Monday, 7=Sunday) |
| `Year-Month` | Text | e.g. `2026-01` |
| `Year-Week` | Text | e.g. `2026-W04` |

### 5.2 Hierarchy: `Year-Month Hierarchy`

A drill-down hierarchy defined on the Date table, used in the Task 3 matrix visual:

```
Year-Month Hierarchy
├── Year-Month   (e.g. 2026-01)
├── Year-Week    (e.g. 2026-W04)
└── Date         (e.g. 2026-01-15)
```

---

## 6. DAX Measures & Calculated Columns

All 31 measures reside in the `__Measures` table, organised into display folders by task. All DAX expressions are extracted directly from the live model.

### 6.1 Task 2\Acquisition

**`NRC`** — New Registered Clients. Uses `USERELATIONSHIP` to activate the inactive `client_registration_date → Date` relationship.
```dax
NRC =
CALCULATE(
    DISTINCTCOUNT(dim_client[client_code]),
    USERELATIONSHIP('Date'[Date], dim_client[client_registration_date])
)
```
*Format: 0*

**`FTD`** — First Time Depositors. Activates the `client_first_deposit_date → Date` relationship and filters to clients with a non-blank first deposit date.
```dax
FTD =
CALCULATE(
    DISTINCTCOUNT(dim_client[client_code]),
    NOT ISBLANK(dim_client[client_first_deposit_date]),
    USERELATIONSHIP('Date'[Date], dim_client[client_first_deposit_date])
)
```
*Format: 0*

**`Conversion Rate`**
```dax
Conversion Rate =
DIVIDE(
    [FTD],
    [NRC]
)
```
*Format: 0.00%;-0.00%;0.00%*

**`Cohort FTD`** — Alternative cohort logic: counts players registered in the selected period who eventually made a first deposit. Uses registration date relationship (not FTD date), with a comment preserved from the original DAX.
```dax
Cohort FTD =
CALCULATE(
    DISTINCTCOUNT(dim_client[client_code]),
    USERELATIONSHIP('Date'[Date], dim_client[client_registration_date]),
    /*Alternative cohort logic:
    Use client_registration_date instead of client_first_deposit_date
    if the goal is to count registered players who eventually made a first deposit.
    */
    NOT ISBLANK(dim_client[client_first_deposit_date])
)
```
*Format: 0*

---

### 6.2 Task 2\Performance

**`Bet`** — Total bet inflow for Casino and Betting. `KEEPFILTERS` preserves any active filter on `transaction_type`, making the measure sensitive to product-level filtering.
```dax
Bet =
CALCULATE(
    SUM(fact_transaction[amount_in]),
    KEEPFILTERS(
        fact_transaction[transaction_type] IN {"Casino", "Betting"}
    )
)
```

**`Win`** — Total win/payout for Casino and Betting.
```dax
Win =
CALCULATE(
    SUM(fact_transaction[amount_out]),
    KEEPFILTERS(
        fact_transaction[transaction_type] IN {"Casino", "Betting"}
    )
)
```

**`GGR`** — Gross Gaming Revenue.
```dax
GGR = [Bet] - [Win]
```

---

### 6.3 Task 2\Payment

**`Deposit`**
```dax
Deposit =
CALCULATE(
    SUM(fact_transaction[amount_in]),
    fact_transaction[transaction_type] = "Deposit"
)
```

**`Withdrawal`** — Outflow on Deposit-type transactions.
```dax
Withdrawal =
CALCULATE(
    SUM(fact_transaction[amount_out]),
    fact_transaction[transaction_type] = "Deposit"
)
```

**`Net Deposit`**
```dax
Net Deposit = [Deposit] - [Withdrawal]
```

---

### 6.4 Task 3

**`Active Players`** — Count of players with Bet > 0 in the current filter context. Sensitive to product filtering because `[Bet]` is evaluated per player.
```dax
Active Players =
COUNTROWS(
    FILTER(
        VALUES(dim_client[client_code]),
        [Bet] > 0
    )
)
```
*Format: 0*

**`AVG Bet per Player`**
```dax
AVG Bet per Player =
DIVIDE(
    [Bet],
    [Active Players]
)
```
*Format: #,0.00*

**`AVG GGR per Player`**
```dax
AVG GGR per Player =
DIVIDE(
    [GGR],
    [Active Players]
)
```
*Format: #,0.00*

**`Max Bet`** — Highest total bet among individual players in the current context.
```dax
Max Bet =
MAXX(
    VALUES(dim_client[client_code]),
    [Bet]
)
```

**`Top Client (Max Bet)`** — Returns `client_code` of the top-betting player.
```dax
Top Client (Max Bet) =
VAR TopClient =
    TOPN(
        1,
        VALUES(dim_client[client_code]),
        [Bet],
        DESC
    )
RETURN
    MAXX(
        TopClient,
        dim_client[client_code]
    )
```
*Format: 0*

**`Dynamic Subtitle`** — Builds a dynamic filter context description for visual subtitles. Detects active slicer selections via `ISFILTERED` and formats them into a readable label.
```dax
Dynamic Subtitle =
VAR MinDate = MIN('Date'[Date])
VAR MaxDate = MAX('Date'[Date])
VAR CityText =
    IF(
        ISFILTERED(dim_client[city]),
        CONCATENATEX(VALUES(dim_client[city]), dim_client[city], ", "),
        "All Cities"
    )
VAR GenderText =
    IF(
        ISFILTERED(dim_client[gender]),
        CONCATENATEX(VALUES(dim_client[gender]), dim_client[gender], ", "),
        "All Genders"
    )
VAR VIPText =
    IF(
        ISFILTERED(dim_client[VIP]),
        CONCATENATEX(VALUES(dim_client[VIP]), dim_client[VIP], ", "),
        "All Segments"
    )
RETURN
    "Filtered by:   Date: "
        & FORMAT(MinDate, "dd/MM/yyyy")
        & " - "
        & FORMAT(MaxDate, "dd/MM/yyyy")
        & "  |  City: " & CityText
        & "  |  Gender: " & GenderText
        & "  |  VIP: " & VIPText
```

---

### 6.5 Task 4

**`Total Deposit (Segment)`**
```dax
Total Deposit (Segment) =
CALCULATE(
    SUM(fact_transaction[amount_in]),
    fact_transaction[transaction_type] = "Deposit"
)
```

**`Min Deposit (Segment)`** / **`Max Deposit (Segment)`**
```dax
Min Deposit (Segment) =
CALCULATE(
    MIN(dim_client[Total Deposit CC]),
    dim_client[Total Deposit CC] > 0
)

Max Deposit (Segment) =
CALCULATE(
    MAX(dim_client[Total Deposit CC]),
    dim_client[Total Deposit CC] > 0
)
```

**`Number of Depositors (Segment)`**
```dax
Number of Depositors (Segment) =
CALCULATE(
    COUNTROWS(dim_client),
    dim_client[Total Deposit CC] > 0
)
```
*Format: 0*

**`AVG Deposit Amount (Segment)`**
```dax
AVG Deposit Amount (Segment) =
DIVIDE(
    SUM(dim_client[Total Deposit CC]),
    [Number of Depositors (Segment)]
)
```

**`Deposit Range`** — Formatted deposit range label using the `#,0.0,K` custom format string.
```dax
Deposit Range =
VAR MinDep = [Min Deposit (Segment)] / 1000
VAR MaxDep = [Max Deposit (Segment)] / 1000
RETURN
FORMAT(MinDep, "#,0.0,K") & " - " & FORMAT(MaxDep, "#,0.0,K")
```

**`% of Total GGR`**
```dax
% of Total GGR =
DIVIDE(
    [GGR],
    CALCULATE([GGR], ALL(dim_client[Deposit Segment]))
)
```
*Format: 0.00%;-0.00%;0.00%*

---

### 6.6 Task 5

**`Bet MA 7D`** — 7-day rolling average of daily Bet, looking back 7 days from the current date in context.
```dax
Bet MA 7D =
AVERAGEX(
    DATESINPERIOD(
        'Date'[Date],
        MAX('Date'[Date]),
        -7,
        DAY
    ),
    [Bet]
)
```

**`Bet vs MA 7D`** — Percentage deviation of current Bet from the moving average.
```dax
Bet vs MA 7D =
DIVIDE(
    [Bet] - [Bet MA 7D],
    [Bet MA 7D]
)
```

**`Bet vs MA 7D Tooltip`** — Formatted tooltip for the line chart.
```dax
Bet vs MA 7D Tooltip =
VAR Diff = [Bet] - [Bet MA 7D]
VAR Pct = [Bet vs MA 7D]
RETURN
FORMAT(Diff, "#,0") & " (" & FORMAT(Pct, "0.0%") & ")"
```

---

### 6.7 Task 7

**`Opted-in Players`**
```dax
Opted-in Players =
CALCULATE(
    COUNTROWS(dim_client),
    NOT ISBLANK(dim_client[opt_in_date])
)
```

**`Opt-in Rate`**
```dax
Opt-in Rate =
DIVIDE(
    [Opted-in Players],
    COUNTROWS(dim_client)
)
```

**`Promo GGR`**
```dax
Promo GGR =
CALCULATE(
    [GGR],
    NOT ISBLANK(dim_client[opt_in_date])
)
```

**`Avg Tickets per Opted-in Player`**
```dax
Avg Tickets per Opted-in Player =
DIVIDE(
    SUM(dim_client[tvdraw_tickets]),
    [Opted-in Players]
)
```

**`Post-Promo Active Opted-in Players`** — Opted-in players with at least one transaction after 2026-01-30.
```dax
Post-Promo Active Opted-in Players =
CALCULATE(
    DISTINCTCOUNT(fact_transaction[client_code]),
    fact_transaction[transaction_date] > DATE(2026, 1, 30),
    NOT ISBLANK(dim_client[opt_in_date])
)
```

---

## 7. Report Pages

The report uses a consistent dark theme across all pages. Each task page includes a back-navigation button (↩) top-right to return to the Navigation Page.

### 7.1 Navigation Page

Landing page — purely navigational, no analytics content.

**Visuals:** Logo, arrow-shaped navigation buttons (Task 1|Task 2 through Task 7), "Tasks Descriptions" button linking to the Tasks Info page.

---

### 7.2 Task 1 | Task 2 — Acquisition & Financial Overview

**Task objectives:** Custom visual identity (Task 1) + Date table, relationships, core KPIs and visualisations with interactive filtering (Task 2).

**Slicers (left panel):** Date range (picker + slider), City, Gender, VIP

**KPI Cards:**

| Card | Measure | Value (full period, no filters) |
|---|---|---|
| NRC | New Registered Clients | 100 |
| FTD | First Time Depositors | 54 |
| Conversion Rate | FTD / NRC | 54.00% |
| Net Deposit | Deposit – Withdrawal | 52.20M |

**Charts:**

| Visual | Type | X-Axis | Y-Axis | Notes |
|---|---|---|---|---|
| NRC and FTD by Month | Line chart | Year-Week | NRC, FTD | Weekly acquisition trend, two lines |
| Withdrawal, Deposit and Net Deposit by Month | Stacked bar + line combo | Month | Withdrawal (red), Deposit (green), Net Deposit (line) | Dual Y-axis |
| Win and Bet by Month | Grouped bar chart | Month | Win, Bet | Side-by-side bars |
| GGR by Month | Horizontal bar chart | Month | GGR | |

---

### 7.3 Task 3 — Player Activity & Performance

**Task objective:** Calculate Active Players, AVG Bet per Player, and AVG GGR per Player. Active players = Bet > 0. All calculations sensitive to product/transaction_type filtering.

**Slicers:** Date, City, Gender, VIP

**KPI Cards:** Active Players, AVG Bet per Player, AVG GGR per Player

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Player Activity & Performance by Time and Product | Matrix | Rows: Year-Month hierarchy (expandable to Year-Week and Date); Columns: `transaction_type` × Active Players, AVG Bet per Player, AVG GGR per Player. Subtitle: `Dynamic Subtitle`. |
| Active Players by Year-Month | Line chart | X: Year-Month; Y: Active Players |
| AVG GGR per Player by Year-Month | Line chart | X: Year-Month; Y: AVG GGR per Player |

---

### 7.4 Task 4 — Player Segmentation by Deposit Behavior

**Task objective:** Four equally-sized quartile groups by total deposit. Compare by deposit range, depositor count, AVG deposit, total Bet, total GGR.

**Slicers:** Date (single date picker), City, Gender, VIP

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Player Segmentation by Deposit Behavior | Table | Columns: `Deposit Segment`, `Deposit Range`, `Number of Depositors (Segment)`, `AVG Deposit Amount (Segment)`, `Bet`, `GGR`, `% of Total GGR`. Conditional formatting: negative GGR red. Subtitle: `Dynamic Subtitle`. |
| GGR Contribution: Top vs Remaining Segments | Clustered bar | `Top Segment` on X-axis; `GGR` on Y-axis — visualises G-1 dominance |

**Segment summary (full dataset, all filters off):**

| Segment | Deposit Range | Depositors | AVG Deposit | GGR | % of Total GGR |
|---|---|---|---|---|---|
| G-1 | 3,843.1K – 36,961.9K | 14 | 13,343.11K | 394.66M | 81.91% |
| G-2 | 437.0K – 2,637.2K | 14 | 1,125.39K | 100.34M | 20.82% |
| G-3 | 116.9K – 390.8K | 14 | 250.92K | -23.62M | -4.90% |
| G-4 | 1.5K – 107.7K | 14 | 53.02K | 3.03M | 0.63% |

---

### 7.5 Task 5 — Daily Bet vs. 7-Day Moving Average

**Task objective:** 7-day moving average of Bet Amount displayed alongside daily Bet.

**Slicers:** Date, City, Gender, VIP

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Daily Bet vs 7-Day Moving Average | Line chart | X: Date (daily); Y: `Bet` (dark green, volatile) and `Bet MA 7D` (lighter, smoothed). Tooltip: `Bet vs MA 7D Tooltip`. Subtitle: `Dynamic Subtitle`. |

---

### 7.6 Task 6 — Dynamic Metric Selector

**Task objective:** One unified page where users select any metric, time period, and time granularity in a single line chart.

**Slicers:** Date (full-width top panel), `Selected Metric` (radio buttons), `Time Granularity` (radio buttons)

**Available metrics:** NRC, FTD, Conversion Rate, Bet, Win, GGR, Deposit, Withdrawal, Net Deposit, Active Players, AVG Bet per Player, AVG GGR per Player, Bet MA 7D

**Available granularities:** Date (daily), Year-Week, Year-Month

**Implementation:** Power BI **Field Parameters** — fully dynamic axis and measure binding without SWITCH logic. Both `Selected Metric` and `Time Granularity` are field parameter tables whose visible columns drive the chart axes.

---

### 7.7 Task 7 — TV Draw Promotion Analysis

**Task objective:** Freestyle analysis — opted-in vs non-opted-in comparison, new vs existing players, communication channel effectiveness, abusive behavior detection, post-promo impact.

**Slicers:** Date, City, VIP, Channel (e-mail, No Promo Info, sms)

**KPI Cards:**

| Card | Measure | Value (full period) |
|---|---|---|
| Opted-in Players | `Opted-in Players` | 42 |
| Opt-in Rate | `Opt-in Rate` | 42.00% |
| Promo GGR | `Promo GGR` | 289.90M |
| Avg Tickets per Opted-in Player | `Avg Tickets per Opted-in Player` | 132.05 |
| Post-Promo Active Opted-in Players | `Post-Promo Active Opted-in Players` | 29 |

**Visuals:**

| Visual | Type | Description |
|---|---|---|
| Player Performance by Promo Participation | Clustered bar | X: `Promo Participation` (Opted-in / Non Opted-in); Y: `Bet` and `GGR` |
| Promo Impact: New vs Existing Players | Clustered bar | X: `Player Type` (New / Existing); Y: `Bet` and `GGR` split by `Promo Participation` legend |
| Player Performance by Communication Channel | Clustered bar | X: `informed_about_promo`; Y: `GGR` and `Bet` per channel |
| Potential Abusive Behavior | Table | Columns: `client_code`, `tvdraw_tickets`, `informed_about_promo`, `Total Bet CC`, `Total GGR CC`. Sorted by `tvdraw_tickets` DESC. Visual-level filter: opted-in only. Negative `Total GGR CC` highlighted red. |
| Promo Trend — GGR Analysis | Line chart | X: Date (daily); Y: `GGR` — Opted-in (green) vs Non Opted-in (white). Reference lines at 01/01/2026 and 30/01/2026. |

**Navigation:** "Show Insights" button (top-right) → TV Draw Promotion — Key Findings page via bookmark.

**Key outliers from Potential Abusive Behavior table:**

| client_code | tvdraw_tickets | Channel | Total Bet CC | Total GGR CC |
|---|---|---|---|---|
| 85 | 1,361 | sms | 134,499.36K | 44,068.61K |
| 43 | 630 | No Promo Info | 10,034.31K | 4,202.61K |
| 49 | 515 | e-mail | 141,778.37K | 100,904.96K |
| 10 | 489 | No Promo Info | 5,947.20K | 5,767.27K |
| **11** | **409** | **e-mail** | **6,858.64K** | **-43,798.51K** |

> Player #85 (1,361 tickets, disproportionately low GGR relative to ticket volume) and Player #11 (409 tickets, **negative GGR of -43,798K**) are the primary outliers flagged for potential abusive behavior.

---

### 7.8 TV Draw Promotion — Key Findings

A dedicated written insights page navigable from Task 7 via the "Show Insights" button (bookmark navigation). Styled with the dark theme as a text box visual.

**Overview:** From 100 players, 42 voluntarily opted in (42% opt-in rate), generating ~290M in Promo GGR at an average of 132 tickets per opted-in player. Post-promotion, 29 of 42 opted-in players (69%) remained active — a strong retention signal.

**Opted-in vs Non-opted-in:** Opted-in players generated significantly higher Bet (0.64bn vs 0.32bn) and GGR (0.29bn vs 0.19bn), confirming that the promotion attracted — or was self-selected by — the most valuable player segment.

**New vs Existing Players:** The promotion was far more effective for existing players (GGR ~0.25bn) than for new ones (~0.04bn). The ticket mechanic alone is insufficient to convert new players — a dedicated onboarding incentive is recommended for future campaigns.

**Communication Channel:** The "No Promo Info" group paradoxically recorded the highest total Bet (~0.44bn), suggesting strong organic opt-in motivation. SMS and e-mail channels performed comparably (~0.26–0.27bn Bet) with no clear advantage.

**Potential Abusive Behavior:** Player #85 accumulated 1,361 tickets with a disproportionately low GGR contribution (~44M). More critically, Player #11 collected 409 tickets while generating a negative GGR of -43,798K — actively costing the operator during the promotion period. A minimum GGR threshold for ticket eligibility is recommended for future promotions.

**Post-Promo Trend:** The line chart shows a clear GGR spike among opted-in players during January, gradually normalizing after January 30th. Reference lines mark the promotion boundaries (01/01 – 30/01/2026).

**Conclusion:** The promotion delivered a net positive outcome across all key KPIs. The only structural improvement needed is a GGR-based eligibility threshold to eliminate opportunistic behavior without impacting genuinely high-value players.

---

### 7.9 Tasks Info (Hidden Page)

Reference page containing all task descriptions. Hidden from end users — accessible only via the "Tasks Descriptions" button on the Navigation Page.

---

## 8. Parameter Tables (Field Parameters)

Two Field Parameter tables used in Task 6 for dynamic axis and measure binding.

### 8.1 `Selected Metric`

*Columns extracted from live model:*

| Column | Visible | Description |
|---|---|---|
| `Selected Metric` | Yes | Display name shown in slicer |
| `Parameter Fields` | No | Internal DAX field reference |
| `Parameter Order` | No | Sort order |

**Metrics:** NRC, FTD, Conversion Rate, Bet, Win, GGR, Deposit, Withdrawal, Net Deposit, Active Players, AVG Bet per Player, AVG GGR per Player, Bet MA 7D

### 8.2 `Time Granularity`

*Columns extracted from live model:*

| Column | Visible | Description |
|---|---|---|
| `Time Granularity` | Yes | Display name: Date, Year-Week, Year-Month |
| `Time Granularity Fields` | No | Reference to Date table column |
| `Time Granularity Order` | No | Sort order |

---

## 9. Design & Theming

### 9.1 Visual Identity

| Element | Description |
|---|---|
| Background | Dark charcoal (near-black) |
| Primary accent | Gold/khaki — card borders, titles, navigation arrows |
| Secondary accent | logo, negative value highlights |
| Data color (primary) | Green — all main chart series |
| Data color (secondary) | Light green / white — secondary series |
| Font | Light sans-serif, white on dark |
| Logo | Logo image visual, top-left on all task pages |

### 9.2 Navigation Pattern

- All task pages: circular back button (↩) top-right → Navigation Page.
- Navigation Page: arrow-shaped buttons for each task.
- Task 7: "Show Insights" button → TV Draw Promotion — Key Findings (bookmark).
- Navigation Page: "Tasks Descriptions" button → Tasks Info hidden page.

### 9.3 Consistency Standards

- All task pages except Task 6 share the same slicer layout: Date picker + slider, City, Gender, VIP.
- Task 6 uses Metric and Time Granularity field parameter selectors instead.
- Task 7 adds a Channel slicer and the "Show Insights" navigation button.
- Conditional formatting (red) applied to negative GGR in Task 4 segmentation table and Task 7 abusive behavior table.
- `Dynamic Subtitle` measure used on all matrix and key chart visuals.
- Measures organised in display folders within `__Measures` by task number.

---

*Documentation v3.0 — all DAX, relationships, and column expressions extracted directly from the live Power BI model via MCP connection.*  
*Prepared by Željko Blagojević, 2026.*
