# NBA Multi-Fact Analytical Model

Multi-table relational performance model built in Power BI combining team, player, draft and physical metrics.

(65k+ games | Multi-fact structure | 11 related tables)

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

This project analyzes historical NBA data to explore performance patterns across teams and players, including draft position, physical attributes and statistical performance.

The objective was to design a structured analytical model capable of handling multiple related fact tables while maintaining clean semantic modeling and controlled filter behavior.

In addition to performance comparison, the model allows exploration of less conventional analytical angles such as correlations between player physical characteristics and performance metrics.

---

## Dataset

- Historical NBA statistics dataset  
- 65k+ recorded games  
- Player-level and team-level performance metrics  
- Draft and physical attribute information  
- Multi-table relational structure (11 tables)

---

## Data Preparation

All data transformation was performed in **Power Query**, including:

- Data cleaning and normalization  
- Type transformation  
- Harmonization of player and team identifiers  
- Structured relationship definition across fact and dimension tables  

The final structure follows a multi-fact modeling approach with shared dimensions.

---

## Analytical Approach

The model includes:

- Multi-fact structure (games, player stats, draft data)  
- Shared dimension tables  
- Controlled single and bi-directional relationships  
- Active and inactive relationship handling  
- Performance comparison logic  
- Context-aware filtering across tables  

Special attention was given to relationship direction management to prevent ambiguous filter propagation.

---

## Key Technical Components

### DAX
- CALCULATE and context transition  
- Performance aggregation measures  
- Comparative metric calculations  
- Conditional logic for player evaluation  

### Modeling
- 11-table relational structure  
- Multi-fact modeling with shared dimensions  
- Active/inactive relationship management  
- Controlled filter propagation  

### UX & Interaction
- Multi-page analytical report  
- Context-driven comparison views  
- Custom tooltips and bookmarks  
- Interactive filtering across dimensions  

---

## Business Value

- Demonstrates advanced modeling discipline  
- Shows ability to manage complex relational structures  
- Highlights understanding of filter propagation logic  
- Enables structured comparative analytics  

---

Demonstrates multi-fact modeling capability, structured relationship management and advanced analytical design in Power BI.
