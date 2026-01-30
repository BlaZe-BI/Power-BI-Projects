# NBA Multi-Fact Analytical Model

Multi-table relational performance model built in Power BI combining team, player, draft and physical metrics.

(65k+ games | 11 related tables | Multi-fact modeling structure)

---

## Report Overview

<p align="center">
  <img src="images/NBA stats by Zeljko Blagojevic 1.png" width="48%">
  <img src="images/NBA stats by Zeljko Blagojevic 2.png" width="48%">
</p>

<p align="center">
  <img src="images/NBA stats by Zeljko Blagojevic 3.png" width="48%">
  <img src="images/NBA stats by Zeljko Blagojevic 4.png" width="48%">
</p>

---

## Background

This project analyzes historical NBA data to explore performance patterns across teams and players, incorporating draft position, physical attributes and statistical performance metrics.

The objective was to design a structured analytical model capable of handling multiple related fact tables while preserving clean semantic modeling and controlled filter behavior.

Beyond standard performance comparison, the model enables exploration of analytical relationships between player physical characteristics and statistical outcomes.

---

## Dataset

- Historical NBA statistics dataset  
- 65k+ recorded games  
- Player-level and team-level performance metrics  
- Draft position data  
- Physical attributes dataset  
- 11-table relational structure  

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization  
- Type transformation  
- Harmonization of player and team identifiers  
- Alignment of fact and dimension tables  
- Structured relationship definition across the model  

The final structure follows a multi-fact modeling approach with shared dimensions.

---

## Analytical Approach

The model includes:

- Multi-fact structure (games, player statistics, draft data)  
- Shared dimension tables  
- Controlled single and bi-directional relationships  
- Active and inactive relationship handling  
- Context-aware filtering across related tables  
- Comparative performance logic  

Special attention was given to relationship direction management to prevent ambiguity and circular filter propagation.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- Aggregation measures across multiple fact tables  
- Comparative performance calculations  
- Conditional evaluation logic  

### Modeling
- 11-table relational structure  
- Multi-fact modeling with shared dimensions  
- Active/inactive relationship management  
- Controlled filter propagation  

### UX & Interaction
- Multi-page analytical report  
- Context-driven comparison views  
- Custom tooltips and bookmarks  
- Interactive filtering across multiple dimensions  

---

## Business Value

- Demonstrates advanced data modeling discipline  
- Showcases multi-fact relational design capability  
- Highlights structured filter propagation management  
- Enables structured comparative analytics  

---

Demonstrates multi-fact modeling capability, relationship management discipline and advanced analytical design in Power BI.
