# E-Commerce Sales Dashboard

E-commerce sales and profitability performance model built in Power BI.

(Revenue | Profit | Orders | Customer insights | Geographic analysis)

---

## Report Overview

<p align="center">
  <img src="images/E-commerce Sales by Zeljko Blagojevic_Page_1.png" width="48%">
  <img src="images/E-commerce Sales by Zeljko Blagojevic_Page_2.png" width="48%">
</p>

<p align="center">
  <img src="images/E-commerce Sales by Zeljko Blagojevic_Page_3.png" width="70%">
</p>

---

## Background

This project focuses on analyzing e-commerce sales performance through structured KPI tracking and interactive reporting.

The objective was to design a dashboard that monitors profitability, order trends, product performance and geographic distribution while maintaining a clean dimensional model structure.

The solution provides management-level visibility into sales dynamics and performance drivers.

---

## Dataset

- Real-world sales dataset  
- Transaction-level order data  
- Product hierarchy  
- Customer attributes  
- Geographic information  

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization  
- Type transformation  
- Removal of inconsistencies  
- Creation of calculated columns  
- Structured dimension separation  

Auto date/time was disabled to ensure controlled time intelligence modeling.

The final structure follows a star-schema approach.

---

## Analytical Approach

The model includes:

- Revenue and profitability calculation  
- Order volume tracking  
- Customer segmentation metrics  
- Quarter-level dynamic filtering  
- Category hierarchy drill-down  
- Geographic visualization using Azure Map  

Structured KPI logic ensures consistent aggregation and business-ready metrics.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- SUMX-based aggregations  
- DIVIDE for safe ratio calculations  
- Quarter filtering logic  
- Profit and margin computation  

### Modeling
- One-to-many relationships  
- Proper fact/dimension separation  
- Clean semantic model structure  

### UX & Interaction
- KPI cards for performance monitoring  
- Drill-down category hierarchy  
- Azure Map visualization  
- Dynamic slicers  
- Custom tooltip behavior  

---

## Business Value

- Enables monitoring of revenue and profit trends  
- Identifies high-performing product categories  
- Supports geographic performance comparison  
- Translates transaction-level data into structured decision support  

---

Demonstrates foundational BI modeling, structured KPI implementation and practical dashboard design discipline.
