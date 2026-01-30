# Social Media Campaign Analysis

Marketing performance dashboard built in Power BI focused on conversion efficiency and cost optimization.

(CTR | CVR | CPA | CPC | CPM | Conversion funnel analysis)

---

## Report Overview

<p align="center">
  <img src="images/Social Media Campaign by Zeljko Blagojevic_Page_1.png" width="48%">
  <img src="images/Social Media Campaign by Zeljko Blagojevic_Page_2.png" width="48%">
</p>

---

## Background

This project analyzes the performance of a social media advertising campaign with a focus on conversion success and cost efficiency.

The objective was to translate marketing metrics into structured analytical KPIs and build a dashboard capable of evaluating campaign performance across demographic segments and engagement behavior.

The dataset originates from a Kaggle marketing dataset describing an anonymous organization’s social media ad campaign.

---

## Dataset

- Source: Kaggle – Sales Conversion Optimization dataset  
- Format: XLSX  
- Campaign-level performance data  
- Demographic attributes  
- Engagement and conversion metrics  

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization  
- Type transformation  
- Metric validation  
- Creation of calculated columns for campaign categorization  

Auto date/time was disabled to ensure controlled analytical modeling.

---

## Analytical Approach

The model includes structured KPI logic for:

- CTR (Click Through Rate)  
- CVR (Conversion Rate)  
- CPA (Cost Per Acquisition)  
- CPC (Cost Per Click)  
- CPM (Cost Per Mille)  

Safe ratio calculations were implemented using DIVIDE to prevent calculation errors.

The dashboard evaluates:

- Engagement vs conversion efficiency  
- Cost structure  
- Category ranking  
- Demographic performance breakdown  

---

## Key Technical Components

### DAX
- CALCULATE and filter context logic  
- DIVIDE for safe ratio calculations  
- Conditional KPI classification  
- Aggregated cost and performance metrics  

### Modeling
- Single-table structured dataset  
- Controlled metric derivation  
- KPI layer separation  

### UX & Interaction
- KPI cards for high-level monitoring  
- Dynamic filtering  
- Custom tooltips  
- Bookmark-driven navigation  

---

## Business Value

- Enables performance comparison across demographic groups  
- Highlights cost-efficiency drivers  
- Identifies high-performing campaign segments  
- Converts marketing metrics into actionable performance insight  

---

Demonstrates structured KPI modeling, marketing analytics interpretation and clean DAX implementation.
