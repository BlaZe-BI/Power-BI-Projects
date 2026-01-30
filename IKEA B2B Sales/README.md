# Retail / B2B Sales Analysis

B2B retail sales performance model built in Power BI.

(220k+ transactions | Croatia | ~2 fiscal years of sales data)

---

## Report Overview

<p align="center">
  <img src="images/IKEA B2B Sales by Zeljko Blagojevic 1.png" width="48%">
  <img src="images/IKEA B2B Sales by Zeljko Blagojevic 2.png" width="48%">
</p>

---

## Background

This project analyzes B2B sales performance across two fiscal years with a focus on KPI tracking, loyalty sales contribution, and geographic performance comparison.

The report was designed to support management-level decision-making by answering structured business questions related to customer activity, revenue contribution, and fiscal performance differences.

---

## Dataset

- Real-world B2B sales dataset (anonymized)
- Croatia market
- Approximately 2 fiscal years of transaction history
- Fact table: 220k+ transactions

Selected attributes were modified to preserve confidentiality.

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization
- Type transformation
- Removal of inconsistencies
- Creation of a custom `dimDate` table
- Auto date/time disabled for controlled time intelligence modeling

The final structure follows a star-schema approach.

---

## Analytical Approach

The model includes:

- Custom fiscal year comparison logic (Sep–Feb focus)
- Year-over-Year difference calculation (absolute and percentage)
- Loyalty sales share calculation
- Distinct customer KPI tracking
- Geographic drill-down hierarchy (Region → Country → Place)

---

## Key Technical Components

### DAX
- CALCULATE and context transition
- Time intelligence logic using custom fiscal calendar
- DIVIDE-based safe ratio calculations
- YoY variance computation

### Modeling
- One-to-many relationships
- Structured star-schema design
- Controlled filter propagation

### UX & Interaction
- Drill-down geographic hierarchy
- KPI cards and variance indicators
- Decomposition tree for contribution analysis

---

## Business Value

- Enables fiscal performance comparison
- Highlights loyalty program contribution
- Identifies regional growth and decline patterns
- Supports management-level KPI monitoring

---

Demonstrates structured KPI modeling, fiscal time intelligence implementation and business-oriented analytical reporting.
