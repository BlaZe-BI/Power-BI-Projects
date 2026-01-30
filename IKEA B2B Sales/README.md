# Retail / B2B Sales Analysis

B2B retail sales performance model built in Power BI.

(220k+ transactions | Croatia | ~2 fiscal years | Anonymized real-world dataset)

---

## Report Overview

<p align="center">
  <img src="images/IKEA B2B Sales by Zeljko Blagojevic 1.png" width="48%">
  <img src="images/IKEA B2B Sales by Zeljko Blagojevic 2.png" width="48%">
</p>

---

## Background

This project analyzes B2B sales performance across two fiscal years with a focus on structured KPI tracking, loyalty contribution and geographic performance comparison.

The report was designed to answer concrete management questions related to:

- Revenue dynamics across fiscal periods  
- Loyalty vs total sales contribution  
- Customer activity and concentration  
- Geographic performance differences  

The solution emphasizes fiscal-aware analytics rather than standard calendar-based reporting.

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
- Fiscal calendar logic implementation  
- Auto date/time disabled for controlled time intelligence modeling  

The final structure follows a structured star-schema approach.

---

## Analytical Approach

The model includes:

- Custom fiscal year comparison logic  
- Year-over-Year difference calculation (absolute and percentage)  
- Loyalty sales share computation  
- Distinct customer KPI tracking  
- Geographic drill-down hierarchy (Region → Country → Place)  
- Contribution analysis using decomposition logic  

Fiscal comparisons are based on business-relevant period alignment rather than default calendar aggregation.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- Custom fiscal time intelligence logic  
- DIVIDE-based safe ratio calculations  
- YoY variance computation  
- KPI aggregation logic  

### Modeling
- One-to-many relationships  
- Structured fact/dimension separation  
- Custom date dimension (fiscal-aware)  
- Controlled filter propagation  

### UX & Interaction
- Drill-down geographic hierarchy  
- KPI cards and variance indicators  
- Decomposition tree for contribution analysis  
- Dynamic filtering across fiscal periods  

---

## Business Value

- Enables structured fiscal performance comparison  
- Highlights loyalty program revenue contribution  
- Identifies regional growth and decline patterns  
- Supports management-level KPI monitoring  

---

Demonstrates fiscal-aware KPI modeling, structured DAX implementation and business-oriented analytical reporting.
