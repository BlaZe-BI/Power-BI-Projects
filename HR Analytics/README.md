# HR Analytics Dashboard

Workforce analytics and attrition modeling solution built in Power BI.

(Real-world HR dataset | Multi-page analytical report | KPI-driven workforce analysis)

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

The objective was to design a KPI-oriented dashboard that enables structured evaluation of:

- Attrition rate dynamics  
- Salary-related turnover behavior  
- Job role distribution  
- Education field segmentation  
- Gender-based workforce comparison  
- Overall workforce composition  

The solution translates employee-level data into measurable retention indicators.

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
- Structured dimensional separation  

Auto date/time was disabled to ensure controlled modeling behavior.

The final structure follows a star-schema approach.

---

## Analytical Approach

The model includes:

- Attrition rate calculation  
- KPI aggregation logic  
- Salary slab segmentation  
- Category mapping using SWITCH(TRUE())  
- Gender-based dynamic filtering  
- Bookmark-driven toggle slicer implementation  
- Conditional formatting and dynamic visual behavior  

A custom toggle slicer was implemented using bookmark logic and selection layering to dynamically switch gender perspectives while modifying visual appearance and metric emphasis.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- SWITCH(TRUE()) classification framework  
- Ratio calculations for attrition metrics  
- Calculated columns and measures  

### Modeling
- One-to-many relationships  
- Structured fact/dimension separation  
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
- Supports retention-oriented HR strategy  
- Provides structured workforce KPI monitoring  

---

Demonstrates KPI-driven HR analytics modeling, structured DAX implementation and advanced Power BI interaction design.
