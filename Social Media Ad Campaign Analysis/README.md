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

This project analyzes the performance of a social media advertising campaign with a focus on structured KPI evaluation and cost-efficiency analysis.

The objective was to translate marketing metrics into measurable analytical indicators and design a dashboard capable of evaluating campaign performance across demographic segments and engagement behavior.

The dataset originates from a Kaggle marketing dataset describing an anonymous organization’s social media advertising campaign.

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
- Metric validation and consistency checks  
- Creation of calculated attributes for campaign categorization  

Auto date/time was disabled to ensure controlled modeling behavior.

---

## Analytical Approach

The model implements structured KPI logic for:

- CTR (Click Through Rate)  
- CVR (Conversion Rate)  
- CPA (Cost Per Acquisition)  
- CPC (Cost Per Click)  
- CPM (Cost Per Mille)  

Safe ratio calculations were implemented using DIVIDE to prevent division errors and ensure reliable aggregation.

The dashboard evaluates:

- Engagement vs conversion efficiency  
- Cost structure and acquisition dynamics  
- Category ranking  
- Demographic performance breakdown  

---

## Key Technical Components

### DAX
- CALCULATE and filter context logic  
- DIVIDE for safe ratio calculations  
- Conditional KPI classification logic  
- Aggregated cost and performance measures  

### Modeling
- Structured single-table dataset  
- Controlled metric derivation  
- Clear KPI layer separation  

### UX & Interaction
- KPI cards for executive monitoring  
- Dynamic filtering across demographic attributes  
- Custom tooltips  
- Bookmark-driven navigation  

---

## Business Value

- Enables structured performance comparison across demographic groups  
- Highlights cost-efficiency drivers  
- Identifies high-performing campaign segments  
- Converts marketing metrics into actionable performance insight  

---

Demonstrates structured marketing KPI modeling, analytical interpretation of campaign metrics and disciplined DAX implementation.
