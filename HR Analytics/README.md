# HR Analytics Dashboard

Workforce analytics model built in Power BI focused on attrition analysis and employee distribution.

(Real-world HR dataset | Multi-page analytical report)

---

## Report Overview

<p align="center">
  <img src="images/HR Analytics by Zeljko Blagojevic 1.png" width="48%">
  <img src="images/HR Analytics by Zeljko Blagojevic 2.png" width="48%">
</p>

<p align="center">
  <img src="images/HR Analytics by Zeljko Blagojevic 3.png" width="48%">
  <img src="images/HR Analytics by Zeljko Blagojevic 4.png" width="48%">
</p>

<p align="center">
  <img src="images/HR Analytics by Zeljko Blagojevic 5off.png" width="48%">
  <img src="images/HR Analytics by Zeljko Blagojevic_Page_5.png" width="48%">
</p>

---

## Background

This project analyzes workforce structure and employee attrition patterns using structured HR data.

The objective was to build a KPI-driven dashboard that helps identify:

- Attrition rate trends  
- Salary-related attrition patterns  
- Job role distribution  
- Education field segmentation  
- Gender-based differences  
- Workforce composition insights  

---

## Dataset

- Real-world HR dataset  
- Employee-level records  
- Demographic attributes  
- Salary information  
- Job role and department classification  

Selected attributes were adjusted where necessary to preserve privacy.

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization  
- Type transformation  
- Creation of calculated attributes (salary slabs, categorical groupings)  
- Structured one-to-many relationships  

The final structure follows a star-schema approach.

---

## Analytical Approach

The model includes:

- Attrition rate calculation  
- KPI aggregation logic  
- Salary slab classification  
- Category mapping using SWITCH(TRUE())  
- Gender-based dynamic filtering  
- Bookmark-driven toggle slicer implementation  
- Conditional formatting and dynamic visual behavior  

A custom toggle slicer was implemented using bookmarks and selection logic to dynamically switch gender views while altering visual styling.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- SWITCH(TRUE()) classification logic  
- Ratio calculations for attrition metrics  
- Calculated columns and measures  

### Modeling
- One-to-many relationships  
- Structured dimensional separation  
- Controlled filter propagation  

### UX & Interaction
- Dynamic filters (Gender, Education Field, Job Role, Salary Slab)  
- Bookmark-driven toggle slicer  
- Conditional visual behavior  
- Multi-page report navigation  

---

## Business Value

- Identifies key drivers of employee attrition  
- Highlights salary-related turnover patterns  
- Supports HR retention strategy  
- Provides structured workforce KPI monitoring  

---

Demonstrates KPI-driven HR analytics modeling, structured DAX implementation and advanced Power BI interaction design.
